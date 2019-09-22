---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-09-21 18:20:00'
layout: post
original_url: https://vsoch.github.io//2019/singularity-web/
title: Inspect Singularity SIF with Web Assembly
---

<p>Today I woke up and decided that I wanted to load a container in the browser. I’ve
thrown around this idea for years - back in 2017 I created <a href="https://vsoch.github.io/2017/singularity-nginx/" target="_blank">Singularity Nginx</a> that would load and interact with a container via an API,
but that was technically a separate server issuing commands to the container on the host,
and not really so great. But today I woke up and was hungry for challenge. So that’s
what I did.</p>

<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/singularity-web/sifweb.png" />
</div>

<p>If you want to jump to the code, <a href="https://github.com/vsoch/sifweb" target="_blank">it’s all here!</a>.
If you want to jump to the live demo, you can find it at <a href="https://vsoch.github.io/sifweb/" target="_blank">vsoch.github.io/sifweb/</a>. Here I’ll talk about some of the challenges and details of the implementation.</p>

<h2 id="web-assembly--golang">Web Assembly + GoLang</h2>

<p>My spidey senses were telling me that I should try using <a href="https://emscripten.org/docs/getting_started/FAQ.html" target="_blank">emscripten</a> to compile GoLang into a wasm (Web Assembly). Since this
entire process was rather foreign to me, I started with a <a href="https://www.sitepen.com/blog/compiling-go-to-webassembly/" target="_blank">hello world</a> example. You can best understand the dependencies by looking at the <a href="https://github.com/vsoch/sifweb/blob/master/Dockerfile" target="_blank">Dockerfile</a> that I put together. The base container uses GoLang 1.13, and we need to do this for a newly added function
<a href="https://tip.golang.org/pkg/syscall/js/#CopyBytesToGo">CopyBytesToGo</a>. On top
of the base we install nginx, git, python, and build essential (providing make, etc.).</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
FROM golang:1.13
# docker build -t vanessa/sifweb .
RUN apt-get update &amp;&amp; apt-get install -y nginx git python build-essential
</code></pre></div></div>

<p>We then install emscripten and add it to the path:</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
WORKDIR /opt
RUN git clone https://github.com/emscripten-core/emsdk.git &amp;&amp; \
    cd emsdk &amp;&amp; \
    git pull &amp;&amp; \
    ./emsdk install latest &amp;&amp; \
    ./emsdk activate latest

ENV PATH /opt/emsdk:/opt/emsdk/fastcomp/emscripten:/opt/emsdk/node/12.9.1_64bit/bin:$PATH
</code></pre></div></div>

<p>And then we add the content of our repository (the GoLang to be compiled plus the static html,
javascript, and css files)</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
WORKDIR /var/www/html
COPY . /var/www/html
</code></pre></div></div>

<p>And finally, we use make on the Makefile, which will build the output wasm. Importantly,
we also need to add “application/wasm” to our container host’s mime.types, otherwise the browser will spit
out an error that it doesn’t recognize the content type.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>RUN make <span class="o">&amp;&amp;</span> <span class="se">\</span>
    mv docs/<span class="k">*</span> /var/www/html <span class="o">&amp;&amp;</span> <span class="se">\</span>
    <span class="nb">echo</span> <span class="s2">"application/wasm                                                wasm"</span> <span class="o">&gt;&gt;</span> /etc/mime.types
</code></pre></div></div>

<p>Running the container means exposing port 80, and starting the web server nginx. Since
nginx by default will load static files in /var/www/html, since we added them there, we are good to go!</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
</code></pre></div></div>

<p>Here is what is in the Makefile - we literally just install one dependency and then
make the wasm.</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
all:
	go get github.com/google/uuid
	GOOS=js GOARCH=wasm go build -o docs/main.wasm
</code></pre></div></div>

<p>The sylabs sif library uses <a href="https://github.com/sylabs/sif/blob/e230d96e4fcc3e4b4013b251a5d4fa8eb3a732bd/pkg/sif/sif.go#L90" target="_blank">github.com/satori/go.uuid</a> for uuid functions, but I found this
had issues with GoLang 1.13 and opted to use google’s.</p>

<p>The output is to the “docs” folder only because this is an option to render on GitHub pages.</p>

<h3 id="build-local">Build Local</h3>

<p>If you are running this locally, you’d also need to add the mime.type to your local
machine, and ensure that you have GoLang 1.13+. Then run make, and cd into the docs folder
and start a web server however you please! I usually just use python:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>python <span class="nt">-m</span> http.server 9999
</code></pre></div></div>

<h2 id="reading-the-file">Reading the File</h2>

<p>The web portion has a drag and drop (or click and select) box that you can use to upload a file.
When the form is submit (meaning the drop or upload is done) this was the hardest bit to figure
out in the html - actually it took me most of the day to get the data format correct, so I’ll share it with you.</p>

<p>If we look at the <a href="https://github.com/sylabs/sif/blob/e230d96e4fcc3e4b4013b251a5d4fa8eb3a732bd/pkg/sif/load.go#L124">loadContainer</a> function for sylabs/sif, it is totally dependent on a file object. We can’t actually
pass a file (at least that I’m aware of, easily) from the browser to GoLang, so instead I was trying to work
with <a href="https://developer.mozilla.org/en-US/docs/Web/API/FileReader#Methods" target="_blank">different
FileReader methods</a>. I tried many variations and many times got something that appeared to work,
but checking against the <a href="https://github.com/sylabs/sif">siftool</a> I hadn’t read the right thing.
The trick was actually to use readAsArrayBuffer and convert the result to a UInt8Array in the browser,
and pass the final raw_data, the file name, and the number of bytes to the function “loadContainer”
in our GoLang:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nx">$</span><span class="p">(</span><span class="s1">'form'</span><span class="p">).</span><span class="nx">submit</span><span class="p">(</span><span class="kd">function</span><span class="p">(</span><span class="nx">event</span><span class="p">){</span>
  <span class="kd">var</span> <span class="nx">file</span> <span class="o">=</span> <span class="nx">$</span><span class="p">(</span><span class="s1">'#file'</span><span class="p">).</span><span class="nx">prop</span><span class="p">(</span><span class="s1">'files'</span><span class="p">)[</span><span class="mi">0</span><span class="p">];</span>
  <span class="kd">var</span> <span class="nx">reader</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">FileReader</span><span class="p">();</span>

  <span class="nx">reader</span><span class="p">.</span><span class="nx">onload</span> <span class="o">=</span> <span class="p">(</span><span class="kd">function</span><span class="p">(</span><span class="nx">theFile</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="kd">function</span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nx">e</span><span class="p">);</span>

      <span class="c1">// This is key! We need to read the Array Buffer as Uint8Array</span>
      <span class="kd">var</span> <span class="nx">raw_data</span> <span class="o">=</span> <span class="k">new</span> <span class="nb">Uint8Array</span><span class="p">(</span><span class="nx">e</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nx">result</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nx">e</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nx">result</span><span class="p">.</span><span class="nx">byteLength</span><span class="p">);</span>

      <span class="c1">// name, bytes, total bytes</span>
      <span class="nx">loadContainer</span><span class="p">(</span><span class="nx">file</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span> <span class="nx">raw_data</span><span class="p">,</span> <span class="nx">reader</span><span class="p">.</span><span class="nx">result</span><span class="p">.</span><span class="nx">byteLength</span><span class="p">);</span>
    <span class="p">};</span>
  <span class="p">})(</span><span class="nx">file</span><span class="p">);</span>

  <span class="c1">// Read in the image file as a data URL.</span>
  <span class="nx">reader</span><span class="p">.</span><span class="nx">readAsArrayBuffer</span><span class="p">(</span><span class="nx">file</span><span class="p">);</span>
  <span class="nx">event</span><span class="p">.</span><span class="nx">preventDefault</span><span class="p">();</span>
<span class="p">})</span>
</code></pre></div></div>

<p>loadContainer (now in GoLang) will take in the name, bytes, and total length</p>

<div class="language-go highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="x">
</span><span class="k">func</span><span class="x"> </span><span class="n">loadContainer</span><span class="p">(</span><span class="n">this</span><span class="x"> </span><span class="n">js</span><span class="o">.</span><span class="n">Value</span><span class="p">,</span><span class="x"> </span><span class="n">val</span><span class="x"> </span><span class="p">[]</span><span class="n">js</span><span class="o">.</span><span class="n">Value</span><span class="p">)</span><span class="x"> </span><span class="k">interface</span><span class="p">{}</span><span class="x"> </span><span class="p">{</span><span class="x">

	</span><span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"The container binary is:"</span><span class="p">,</span><span class="x"> </span><span class="n">val</span><span class="p">[</span><span class="m">0</span><span class="p">])</span><span class="x">
        </span><span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Size:"</span><span class="p">,</span><span class="x"> </span><span class="n">val</span><span class="p">[</span><span class="m">2</span><span class="p">]</span><span class="o">.</span><span class="n">Int</span><span class="p">())</span><span class="x">
	</span><span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"ArrayBuffer:"</span><span class="p">,</span><span class="x"> </span><span class="n">val</span><span class="p">[</span><span class="m">1</span><span class="p">])</span><span class="x">
	
	</span><span class="n">fimg</span><span class="x"> </span><span class="o">:=</span><span class="x"> </span><span class="n">FileImage</span><span class="p">{}</span><span class="x">

	</span><span class="c">// read the string of given size to bytes from the SIF file</span><span class="x">
	</span><span class="k">if</span><span class="x"> </span><span class="n">err</span><span class="x"> </span><span class="o">:=</span><span class="x"> </span><span class="n">fimg</span><span class="o">.</span><span class="n">loadBytes</span><span class="p">(</span><span class="n">val</span><span class="p">[</span><span class="m">1</span><span class="p">],</span><span class="x"> </span><span class="n">val</span><span class="p">[</span><span class="m">2</span><span class="p">]</span><span class="o">.</span><span class="n">Int</span><span class="p">());</span><span class="x"> </span><span class="n">err</span><span class="x"> </span><span class="o">!=</span><span class="x"> </span><span class="no">nil</span><span class="x"> </span><span class="p">{</span><span class="x">
		</span><span class="n">returnResult</span><span class="p">(</span><span class="s">"Error loading bytes."</span><span class="p">)</span><span class="x">
		</span><span class="k">return</span><span class="x"> </span><span class="no">nil</span><span class="x">
	</span><span class="p">}</span><span class="x">
</span><span class="o">...</span><span class="x">

</span></code></pre></div></div>

<p>and by the way it’s bound to the global DOM in our main.go:</p>

<div class="language-go highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="x">
</span><span class="k">package</span><span class="x"> </span><span class="n">main</span><span class="x">
 
</span><span class="k">import</span><span class="x"> </span><span class="s">"syscall/js"</span><span class="x">

</span><span class="k">func</span><span class="x"> </span><span class="n">main</span><span class="p">()</span><span class="x"> </span><span class="p">{</span><span class="x">

	</span><span class="n">c</span><span class="x"> </span><span class="o">:=</span><span class="x"> </span><span class="nb">make</span><span class="p">(</span><span class="k">chan</span><span class="x"> </span><span class="k">struct</span><span class="p">{},</span><span class="x"> </span><span class="m">0</span><span class="p">)</span><span class="x">
	</span><span class="n">js</span><span class="o">.</span><span class="n">Global</span><span class="p">()</span><span class="o">.</span><span class="n">Set</span><span class="p">(</span><span class="s">"loadContainer"</span><span class="p">,</span><span class="x"> </span><span class="n">js</span><span class="o">.</span><span class="n">FuncOf</span><span class="p">(</span><span class="n">loadContainer</span><span class="p">))</span><span class="x">
	</span><span class="o">&lt;-</span><span class="n">c</span><span class="x">
</span><span class="p">}</span><span class="x">
</span></code></pre></div></div>

<p>and then in a helper function called in loadContainer, we use CopyBytesToGo to convert the js.Value to the byte array, and
pass this into a bytes.NewReader to populate fimg.Reader. What I was trying to do here
is emulate exactly what the final fimg.Reader gets, without having read from a file.</p>

<div class="language-go highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="x">
</span><span class="c">// loadBytes loads an imageString from the browser and populates FileImage with data.</span><span class="x">
</span><span class="k">func</span><span class="x"> </span><span class="p">(</span><span class="n">fimg</span><span class="x"> </span><span class="o">*</span><span class="n">FileImage</span><span class="p">)</span><span class="x"> </span><span class="n">loadBytes</span><span class="p">(</span><span class="n">value</span><span class="x"> </span><span class="n">js</span><span class="o">.</span><span class="n">Value</span><span class="p">,</span><span class="x"> </span><span class="n">size</span><span class="x"> </span><span class="kt">int</span><span class="p">)</span><span class="x"> </span><span class="kt">error</span><span class="x"> </span><span class="p">{</span><span class="x">

	</span><span class="c">// We can use CopyBytesToGo, need golang 1.13+</span><span class="x">
	</span><span class="n">sif</span><span class="x"> </span><span class="o">:=</span><span class="x"> </span><span class="nb">make</span><span class="p">([]</span><span class="kt">byte</span><span class="p">,</span><span class="x"> </span><span class="n">size</span><span class="p">)</span><span class="x">
	</span><span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="n">value</span><span class="p">)</span><span class="x">
	</span><span class="n">howmany</span><span class="x"> </span><span class="o">:=</span><span class="x"> </span><span class="n">js</span><span class="o">.</span><span class="n">CopyBytesToGo</span><span class="p">(</span><span class="n">sif</span><span class="p">,</span><span class="x"> </span><span class="n">value</span><span class="p">)</span><span class="x">
	</span><span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="n">howmany</span><span class="p">)</span><span class="x">

	</span><span class="c">// Read in the string to bytes, n should equal size</span><span class="x">
	</span><span class="n">reader</span><span class="x"> </span><span class="o">:=</span><span class="x"> </span><span class="n">bytes</span><span class="o">.</span><span class="n">NewReader</span><span class="p">(</span><span class="n">sif</span><span class="p">)</span><span class="x">
        </span><span class="n">n</span><span class="p">,</span><span class="x"> </span><span class="n">_</span><span class="x"> </span><span class="o">:=</span><span class="x"> </span><span class="n">reader</span><span class="o">.</span><span class="n">Read</span><span class="p">(</span><span class="n">sif</span><span class="p">)</span><span class="x">
	</span><span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="n">sif</span><span class="p">)</span><span class="x">

	</span><span class="c">// Save the data and size to the FileImage</span><span class="x">
	</span><span class="n">fimg</span><span class="o">.</span><span class="n">Filesize</span><span class="x"> </span><span class="o">=</span><span class="x"> </span><span class="kt">int64</span><span class="p">(</span><span class="n">n</span><span class="p">)</span><span class="x">
	</span><span class="n">fimg</span><span class="o">.</span><span class="n">Filedata</span><span class="x"> </span><span class="o">=</span><span class="x"> </span><span class="n">sif</span><span class="x">
	</span><span class="n">fimg</span><span class="o">.</span><span class="n">Reader</span><span class="x"> </span><span class="o">=</span><span class="x"> </span><span class="n">bytes</span><span class="o">.</span><span class="n">NewReader</span><span class="p">(</span><span class="n">fimg</span><span class="o">.</span><span class="n">Filedata</span><span class="p">)</span><span class="x">

	</span><span class="k">return</span><span class="x"> </span><span class="no">nil</span><span class="x">
</span><span class="p">}</span><span class="x">
</span></code></pre></div></div>

<p>The rest of loadContainer basically:</p>

<ol class="custom-counter">
  <li>determines if it's a valid SIF based on SIFMAGIC!</li>
  <li>reads descriptors</li>
  <li>formats header metadata into a string to return</li>
</ol>

<p>Actually, we don’t really return - we just interact with the DOM directly from
GoLang. That looks like this (I wrote a function to populate the #result element)</p>

<div class="language-go highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="x">
</span><span class="c">// returnResult back to the browser, in the innerHTML of the result element</span><span class="x">
</span><span class="k">func</span><span class="x"> </span><span class="n">returnResult</span><span class="p">(</span><span class="n">output</span><span class="x"> </span><span class="kt">string</span><span class="p">)</span><span class="x"> </span><span class="p">{</span><span class="x">
	</span><span class="n">js</span><span class="o">.</span><span class="n">Global</span><span class="p">()</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="s">"document"</span><span class="p">)</span><span class="o">.</span><span class="x">
		</span><span class="n">Call</span><span class="p">(</span><span class="s">"getElementById"</span><span class="p">,</span><span class="x"> </span><span class="s">"result"</span><span class="p">)</span><span class="o">.</span><span class="x">
		</span><span class="n">Set</span><span class="p">(</span><span class="s">"innerHTML"</span><span class="p">,</span><span class="x"> </span><span class="n">output</span><span class="p">)</span><span class="x">
</span><span class="p">}</span><span class="x">
</span></code></pre></div></div>

<p>And that’s it! It may seem simple, but not having any experience with Web Assembly,
not much with GoLang, and definitely consistent struggles with JavaScript,
I’m feeling like a badass right now! :D</p>

<blockquote>
  <p>Yo dawg we just read a container binary into the browser!!</p>
</blockquote>

<h2 id="next-steps">Next Steps</h2>

<p>Let’s talk about next steps, because now that the basics are done, there is so much
more we can do!</p>

<h3 id="filereader-limits">FileReader Limits</h3>

<p>Firstly, the example will crash if you load a fat container, and
this is obviously because the browser has <a href="https://stackoverflow.com/questions/53237829/how-to-load-large-local-files-using-javascript" target="_blank">upload limits</a>. What we need to do is
either:</p>

<ol class="custom-counter">
  <li>smartly read chunks that we need</li>
  <li>read up to a maximum size where we can find the header</li>
  <li>some other partial reading strategy</li>
</ol>

<p>to support larger images. I did a quick and dirty “read the entire thing” because
I knew it would work for busybox, and knew this would need to be refined.
We definitely don’t need to read all that image data.</p>

<h3 id="signatures-and-partitions">Signatures and Partitions</h3>

<p>Guess what - the SIF header is loaded with goodness! I wrote a <a href="https://pypi.org/project/sif/">library in Python</a>
to extract the signature blocks, and that would be (I think) possible to do here too.</p>

<h3 id="your-feedback">Your Feedback!</h3>

<p>Speaking of, I’d love to know your feedback about what kind of tools you’d like to see,
given that you can load and inspect some of these fields in the browser.  Let me know,
and <a href="https://github.com/vsoch/sifweb/issues" target="_blank">open an issue!</a>
I had amazing fun today, and am looking forward to turning this into something that
could be useful for you.</p>