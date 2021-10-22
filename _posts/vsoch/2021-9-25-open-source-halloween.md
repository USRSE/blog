---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2021-09-25 08:30:00'
layout: post
original_url: https://vsoch.github.io//2021/open-source-halloween/
title: Open Source Halloween
---

<p>Now that we’ve officially entered Fall, I think it’s fair to be thinking about pumpkins,
scary stories, and my favorite holiday… Halloween! Sure, I don’t go trick or treating anymore,
and I definitely don’t eat any candy, but there is something hugely magical ✨️ about this time
of year. I am drawn to the oranges and purples, the coolness of the air, and the sudden
inundation of candy and festive holiday things at stores (well, at least back when I went to stores before February 2020!).</p>

<h2 id="open-source-halloween">Open Source Halloween?</h2>

<p>If you are an open source maintainer, you are likely familiar with <a href="https://hacktoberfest.digitalocean.com/" target="_blank">Hacktoberfest</a>, which I (think) has been around since maybe 2014, and the spirit
of the effort is to get people excited and participating in open source. But in recent years,
Hacktoberfest has been more of a <a href="https://blog.domenic.me/hacktoberfest/" target="_blank">stress</a>
on open source maintainers than anything else. I have high hopes that the issues will be resolved, but in
the meantime I was left to ask myself</p>

<blockquote>
  <p>How else can we have fun for Halloween?</p>
</blockquote>

<p>This is where I came up with the idea of 🍬️ Open Source Trick or Treating 🍬️.
The idea would be simple - as you visit repositories, if they offer a piece of candy, you
can grab it and go trick or treating, and add to some basket. Now, since I don’t control or work at
GitHub and can’t add a magical “trick or treat” button, I had to be clever about how to go about this.
This led to my idea for the following:</p>

<ol class="custom-counter">
  <li> Use a candy generator to create a piece of candy for your repository.</li>
  <li> Put it somewhere in your repository named in a way it can be discovered.</li>
  <li> Create a community "treat bag" that discovers and displays the files!</li>
</ol>

<p>The steps above were fairly simple, but it took me a few weekends to get everything in place, because, well,
making web interfaces can be a bit slower than other kinds of programming, and further, creating custom images of candy on the fly is no small task! For the rest of this post, I’ll talk about the different steps.</p>

<h2 id="the-candy-generator">The Candy Generator</h2>

<p>The <a href="https://vsoch.github.io/candy-generator" target="_blank">candy generator</a> is what took me a few evenings to make. It generates pieces of candy that look like this:</p>

<div class="padding:20px">
<img src="https://vsoch.github.io/assets/images/posts/open-source-halloween/open-source-halloween-2021.png" />
</div>
<p><br /></p>

<p>Don’t worry, not all of the candy is pink! The generator allows you to customize the colors, candy texture, candy shape, and then load a specific repository to load the candy “nutrition facts” and logo. Here is another one:</p>

<p><br /></p>

<div class="padding:20px">
<img src="https://raw.githubusercontent.com/pydicom/deid/54c8f68fe1fa0ea65371b78b30e265646ccae49d/docs/assets/img/open-source-halloween-2021.png" />
</div>
<p><br /></p>

<h3 id="candy-shape">Candy Shape</h3>

<p>The candy shape (and editing it on the fly) was something I wasn’t sure I could do, but I thought I might figure it out. I first started with a basic process to generate an svg shape. I would start with an actual piece of candy, trace the bitmap in Inkscape, and then tweak the surrounding points to both change and smooth out the shape to my liking. I could then save to this file as an svg with a single color. Then I needed to add it to a web page. I was first experimenting with adding it via an <a href="https://d3js.org/" target="_blank">D3</a> shape, also just using basic javascript to load it and add to the page. That worked okay, but it was fairly hard to interact with further. I then had the insight that all I really needed were the paths, and in fact, d3 was very good at manipulating paths. This is the solution that ultimately worked, because I could create a bunch of candy svgs, and copy paste the paths (and other metadata) into a data structure to load into d3. You can see the data structure (json) <a href="https://github.com/vsoch/candy-generator/blob/main/assets/js/candy.json" target="_blank">here</a>, and an example with annotations is below.</p>

<div class="language-json highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">{</span><span class="w">

  </span><span class="err">#</span><span class="w"> </span><span class="err">The</span><span class="w"> </span><span class="err">name</span><span class="w"> </span><span class="err">of</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">candy</span><span class="w"> </span><span class="err">path</span><span class="w"> </span><span class="err">is</span><span class="w"> </span><span class="s2">"crunch"</span><span class="w"> </span><span class="err">and</span><span class="w"> </span><span class="err">this</span><span class="w"> </span><span class="err">is</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">path</span><span class="w"> </span><span class="err">from</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">svg</span><span class="w">
  </span><span class="nl">"crunch"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="nl">"paths"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"M 903.641 370.032 C 902.541 369.749 892.344 ... 370.032 Z"</span><span class="p">],</span><span class="w">

    </span><span class="err">#</span><span class="w"> </span><span class="err">Each</span><span class="w"> </span><span class="err">candy</span><span class="w"> </span><span class="err">needed</span><span class="w"> </span><span class="err">a</span><span class="w"> </span><span class="err">custom</span><span class="w"> </span><span class="err">transform</span><span class="w"> </span><span class="err">to</span><span class="w"> </span><span class="err">position</span><span class="w"> </span><span class="err">in</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">interface</span><span class="w">
    </span><span class="nl">"transform"</span><span class="p">:</span><span class="w"> </span><span class="s2">"translate(38.130333,-41.091284)"</span><span class="p">,</span><span class="w">

    </span><span class="err">#</span><span class="w"> </span><span class="err">These</span><span class="w"> </span><span class="err">are</span><span class="w"> </span><span class="err">extra</span><span class="w"> </span><span class="err">attributes</span><span class="w"> </span><span class="err">to</span><span class="w"> </span><span class="err">add!</span><span class="w">
    </span><span class="nl">"attrs"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
      </span><span class="p">[</span><span class="w">
        </span><span class="s2">"fill"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"#482f1e"</span><span class="w">
      </span><span class="p">],</span><span class="w">
      </span><span class="p">[</span><span class="w">
        </span><span class="s2">"stroke-width"</span><span class="p">,</span><span class="w">
        </span><span class="mf">1.33333337</span><span class="w">
      </span><span class="p">],</span><span class="w">
      </span><span class="p">[</span><span class="w">
        </span><span class="s2">"fill-opacity"</span><span class="p">,</span><span class="w">
        </span><span class="mi">1</span><span class="w">
      </span><span class="p">]</span><span class="w">
    </span><span class="p">],</span><span class="w">

    </span><span class="err">#</span><span class="w"> </span><span class="err">These</span><span class="w"> </span><span class="err">help</span><span class="w"> </span><span class="err">with</span><span class="w"> </span><span class="err">positioning</span><span class="w">
    </span><span class="nl">"yoffset"</span><span class="p">:</span><span class="w"> </span><span class="mi">80</span><span class="p">,</span><span class="w">
    </span><span class="nl">"xoffset"</span><span class="p">:</span><span class="w"> </span><span class="mi">100</span><span class="p">,</span><span class="w">
    </span><span class="nl">"transform"</span><span class="p">:</span><span class="w"> </span><span class="s2">"translate(6.7638037,-29.309819)"</span><span class="p">,</span><span class="w">

    </span><span class="err">#</span><span class="w"> </span><span class="err">And</span><span class="w"> </span><span class="err">making</span><span class="w"> </span><span class="err">sure</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">size</span><span class="w"> </span><span class="err">looks</span><span class="w"> </span><span class="err">okay</span><span class="w">
    </span><span class="nl">"scale"</span><span class="p">:</span><span class="w"> </span><span class="s2">"0.8"</span><span class="w">
  </span><span class="p">},</span><span class="w">
</span><span class="err">...</span><span class="w">
</span></code></pre></div></div>

<p>With this data structure, I could have a basic formula for the user to select
some preferences in a panel, and then update the svg with the choice. And in fact that is how most of the interface works - we keep an object that has the state of things (colors, shapes, textures) and then change those attributes when you make some new selection. Here is is a snippet that shows mapping a datum like the above to the page as an svg:</p>

<div class="language-js highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">this</span><span class="p">.</span><span class="nx">svg</span> <span class="o">=</span> <span class="nx">d3</span><span class="p">.</span><span class="nx">select</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">divid</span><span class="p">).</span><span class="nx">append</span><span class="p">(</span><span class="dl">"</span><span class="s2">svg</span><span class="dl">"</span><span class="p">)</span>
    <span class="p">.</span><span class="nx">attr</span><span class="p">(</span><span class="dl">"</span><span class="s2">id</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">svg</span><span class="dl">"</span><span class="p">)</span>
    <span class="p">.</span><span class="nx">attr</span><span class="p">(</span><span class="dl">"</span><span class="s2">width</span><span class="dl">"</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">width</span><span class="p">)</span>
    <span class="p">.</span><span class="nx">attr</span><span class="p">(</span><span class="dl">"</span><span class="s2">height</span><span class="dl">"</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">height</span><span class="p">);</span>

<span class="k">this</span><span class="p">.</span><span class="nx">group</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">svg</span><span class="p">.</span><span class="nx">selectAll</span><span class="p">(</span><span class="dl">'</span><span class="s1">paths</span><span class="dl">'</span><span class="p">)</span>
    <span class="p">.</span><span class="nx">data</span><span class="p">(</span><span class="nb">Array</span><span class="p">(</span><span class="nx">choice</span><span class="p">))</span>
    <span class="p">.</span><span class="nx">enter</span><span class="p">()</span>
    <span class="p">.</span><span class="nx">append</span><span class="p">(</span><span class="dl">"</span><span class="s2">g</span><span class="dl">"</span><span class="p">)</span>
    <span class="p">.</span><span class="nx">attr</span><span class="p">(</span><span class="dl">"</span><span class="s2">transform</span><span class="dl">"</span><span class="p">,</span> <span class="kd">function</span><span class="p">(</span><span class="nx">d</span><span class="p">)</span> <span class="p">{</span><span class="k">return</span> <span class="nx">d</span><span class="p">.</span><span class="nx">transform</span><span class="p">})</span>

<span class="k">this</span><span class="p">.</span><span class="nx">paths</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">group</span><span class="p">.</span><span class="nx">append</span><span class="p">(</span><span class="dl">'</span><span class="s1">svg:path</span><span class="dl">'</span><span class="p">)</span>
    <span class="p">.</span><span class="nx">attr</span><span class="p">(</span><span class="dl">'</span><span class="s1">d</span><span class="dl">'</span><span class="p">,</span> <span class="kd">function</span><span class="p">(</span><span class="nx">d</span><span class="p">)</span> <span class="p">{</span><span class="k">return</span> <span class="nx">d</span><span class="p">.</span><span class="nx">paths</span><span class="p">})</span>
    <span class="p">.</span><span class="nx">attr</span><span class="p">(</span><span class="dl">'</span><span class="s1">id</span><span class="dl">'</span><span class="p">,</span> <span class="dl">"</span><span class="s2">candy-path</span><span class="dl">"</span><span class="p">)</span>
    <span class="p">.</span><span class="nx">attr</span><span class="p">(</span><span class="dl">'</span><span class="s1">transform</span><span class="dl">'</span><span class="p">,</span> <span class="kd">function</span><span class="p">(</span><span class="nx">d</span><span class="p">)</span> <span class="p">{</span><span class="k">return</span> <span class="dl">"</span><span class="s2">scale(</span><span class="dl">"</span><span class="o">+</span> <span class="nx">d</span><span class="p">.</span><span class="nx">scale</span> <span class="o">+</span> <span class="dl">"</span><span class="s2">)</span><span class="dl">"</span><span class="p">})</span>
<span class="p">...</span>
</code></pre></div></div>

<p>You can see the entire javascript file <a href="https://github.com/vsoch/candy-generator/blob/main/assets/js/candygen.js" target="_blank">here</a>. It’s implemented as a class (the reference to “this”) that you can instantiate
with the loaded data:</p>

<div class="language-js highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nx">fetch</span><span class="p">(</span><span class="dl">'</span><span class="s1">assets/js/candy.json</span><span class="dl">'</span><span class="p">)</span>
  <span class="p">.</span><span class="nx">then</span><span class="p">(</span><span class="nx">response</span> <span class="o">=&gt;</span> <span class="nx">response</span><span class="p">.</span><span class="nx">json</span><span class="p">())</span>
  <span class="p">.</span><span class="nx">then</span><span class="p">(</span><span class="nx">json</span> <span class="o">=&gt;</span> <span class="p">{</span>
     <span class="kd">var</span> <span class="nx">candygen</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">CandyGen</span><span class="p">(</span><span class="nx">json</span><span class="p">);</span>
<span class="p">})</span>
</code></pre></div></div>

<p>I had higher aspirations to use Vue.js and include even the html with a component to plug
in anywhere, but about half-way in I had trouble getting d3 and Vue to live side by side.
I think there is <a href="https://alligator.io/vuejs/visualization-vue-d3/" target="_blank">a way to do it</a>
but it involves npm and that is something I just won’t touch.</p>

<h3 id="candy-texture">Candy Texture</h3>

<p>I found a lovely <a href="https://riccardoscalco.it/textures/" target="_blank">texture library</a> that
I suspect data scientists will love that allowed me to generate textures on the fly, map them to names for the user to choose, and then to update the interface with a new choice. This is how you can easily click “carrots” and then
see that texture (^) appear. This logic is a pretty long and redundant function you can see <a href="https://github.com/vsoch/candy-generator/blob/main/assets/js/candygen.js#L141-L248" target="_blank">here</a>. It was really very fun coming up with the textures, and if you’d like to generate one and add to the interface please open a pull request!</p>

<h3 id="nutrition-facts-and-branding">Nutrition Facts and Branding</h3>

<p>The coolest part is loading a repository by name, and then seeing “nutrition facts” pop up that show the number of stars, subscribers, the repository description, and the repository name and logo. This was <a href="https://github.com/vsoch/candy-generator/blob/main/assets/js/candygen.js#L105-L112" target="_blank">fairly easy to do</a>, and comes down to making a request to the GitHub RestFUL API.</p>

<div class="language-js highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nx">fetch</span><span class="p">(</span><span class="dl">'</span><span class="s1">https://api.github.com/repos/</span><span class="dl">'</span> <span class="o">+</span> <span class="nx">uri</span><span class="p">)</span>
    <span class="p">.</span><span class="nx">then</span><span class="p">(</span><span class="nx">response</span> <span class="o">=&gt;</span> <span class="nx">response</span><span class="p">.</span><span class="nx">json</span><span class="p">())</span>
    <span class="p">.</span><span class="nx">then</span><span class="p">(</span><span class="nx">data</span> <span class="o">=&gt;</span> <span class="p">{</span>
         <span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nx">data</span><span class="p">);</span>
         <span class="nb">window</span><span class="p">.</span><span class="nx">github_data</span> <span class="o">=</span> <span class="nx">data</span><span class="p">;</span>		
    <span class="p">})</span>
    <span class="p">.</span><span class="k">catch</span><span class="p">(</span><span class="nx">error</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nx">error</span><span class="p">(</span><span class="nx">error</span><span class="p">));</span>
</code></pre></div></div>

<p>This is another likely source of error - I’m not sure how rate limiting works for un-authenticated requests on GitHub pages, although I suspect that it just uses the un-authenticated endpoint associated with the ip address of the request. If anyone knows, please let me know!</p>

<p>Finally, I decided to write the date (year) on the candy, so if this continues forward into the future, we can distinguish candies from different years. Maybe they will be collectible items, har har. 😄️</p>

<h3 id="missing-functionality">Missing Functionality</h3>

<p>I’ve had experience with saving svg or canvas to file before, but this time around since either fonts were lost
or the exact look of the candy wasn’t quite right, I didn’t get it working as I wanted, so I opted for the instruction to the user to
“take a screen shot.” This of course isn’t perfect (and if I had infinite time I might come back to this) but it’s definitely missing functionality.</p>

<h2 id="the-final-page">The Final Page</h2>

<div class="padding:20px">
<img src="https://vsoch.github.io/assets/images/posts/open-source-halloween/candy-generator.png" />
</div>
<p><br /></p>

<p>Tada! That’s it! If you haven’t yet, check out the <a href="https://vsoch.github.io/candy-generator" target="_blank">candy generator</a>! It was so fun to make. See, you don’t need to be an official “frontend” designer to make front end things, even with basic javascript, html, and css.</p>

<h2 id="trick-or-treat">Trick or Treat!</h2>

<p>After I had the candy generator, I wanted a way to trick or trick. Since I couldn’t control GitHub and add a button, I considered making a browser extension, but decided against that because I personally hate installing extra extensions. I also thought about making a GitHub action where a user could add a list of repos with candy to “discover” but ultimately decided that was too much work. I changed my mindset from “How can a user trick or treat” to “How can the entire community Trick or Treat” (and celebrate open source projects) and realized that I could make a single interface that would discover these files! I’ve had experience doing this for other find-able files with the <a href="https://singularityhub.github.io/singularity-catalog" target="_blank">Singularity Catalog</a> and <a href="https://spack.github.io/spack-stack-catalog/" target="_blank">Spack Stacks</a>, so this was a matter of customizing the script to do a different search, and capture and save a slightly different set of metadata and files to render into a GitHub pages site.  The steps are simple:</p>

<ol class="custom-counter">
  <li> Search GitHub for files named "open-source-halloween-2021.png"</li>
  <li> Clone the repository to find the file to save, and save if we haven't seen the digest yet.</li>
  <li> Save to jekyll data (yaml) and javascript (js) files to load into interface.</li>
  <li> Run a nightly job to update it!</li>
</ol>

<p>The searching and parsing over results was fairly simple and easy, and there was a bit of novelty with checking digests and then generating the interface, discussed next!</p>

<h3 id="image-digests">Image digests</h3>

<p>I was worried about someone forking a repository with a candy image, and then the image being added twice. The best way to prevent this, I thought, would be to calculate the digests of the images, and skip over those that we’ve already seen. Yes, this could mean that if for some reason a fork of your repository with the same image is indexed before your repository, you’d see their image and not yours, but ultimatly if it’s a fork it will lead to your repository, so I’m not hugely worried. In Python it’s pretty simple to calculate a digest for an image (or any file really):</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">def</span> <span class="nf">get_digest</span><span class="p">(</span><span class="n">filename</span><span class="p">):</span>
    <span class="s">"""
    Don't add repeated images of candy (from forks, etc.)
    """</span>
    <span class="n">hasher</span> <span class="o">=</span> <span class="n">hashlib</span><span class="p">.</span><span class="n">md5</span><span class="p">()</span>
    <span class="k">with</span> <span class="nb">open</span><span class="p">(</span><span class="n">filename</span><span class="p">,</span> <span class="s">"rb"</span><span class="p">)</span> <span class="k">as</span> <span class="n">fd</span><span class="p">:</span>
        <span class="n">content</span> <span class="o">=</span> <span class="n">fd</span><span class="p">.</span><span class="n">read</span><span class="p">()</span>
    <span class="n">hasher</span><span class="p">.</span><span class="n">update</span><span class="p">(</span><span class="n">content</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">hasher</span><span class="p">.</span><span class="n">hexdigest</span><span class="p">()</span>
</code></pre></div></div>

<p>So I first load the images that are already present in the repository and data, and then 
calculate digests for all of them, and upon discovery of a new repository, I only add images that
we have not seen anywhere else yet.</p>

<h3 id="the-interface">The Interface</h3>

<p>The interface is a lot of javascript and styling, and I don’t need to go into details, but <a href="https://github.com/rseng/open-source-halloween" target="_blank">check out the source code</a> if you are interested. It will allow you to browse open source candies</p>

<p><br /></p>

<div class="padding:20px">
<img src="https://raw.githubusercontent.com/rseng/open-source-halloween/main/assets/img/open-source-halloween.png" />
</div>
<p><br /></p>

<p>And for any candy, mouse-over to see it larger, and click to see more metadata and links to explore!</p>

<p><br /></p>

<div class="padding:20px">
<img src="https://raw.githubusercontent.com/rseng/open-source-halloween/main/assets/img/trick-or-treat.png" />
</div>
<p><br /></p>

<p>I really love all the candies, here is another one!</p>

<p><br /></p>

<div class="padding:20px">
<img src="https://raw.githubusercontent.com/rseng/open-source-halloween/main/_candy/singularityhub/sregistry/docs/assets/img/open-source-halloween-2021.png" />
</div>

<h3 id="automate">Automate!</h3>

<p>It’s not hard to automate - we can basically run a nightly GitHub workflow to do exactly that. It will run the script and push updated results. For larger projects I’ve found that I’ve hit “secondary rate limits” of the GitHub API, and this worries me because I suspect these are for things that are malicious, but I hope that GitHub realizes I am definitely not that!</p>

<h2 id="how-to-contribute">How to contribute?</h2>

<p>There are so many ways to contribute or have fun with this project!</p>

<h3 id="make-candy">Make Candy</h3>

<p>Obviously the most straight forward thing to do is create open source candy with the <a href="https://vsoch.github.io/candy-generator" target="_blank">candy generator</a>. If you don’t want to do it, open a “good first issue” for Hacktoberfest (or not if you aren’t participating) asking someone to create and add one to the repository. You don’t need to display it at the front of the README, but if you do, it would be a lot of fun, and feel free to link to these tools so others can have fun too.</p>

<h3 id="edit-the-generator">Edit the Generator</h3>

<p>There could be much improvement to the generator! It would be nice to have a button to save directly to a png (with the correct name) and further ability to customize the candy. Any contributions to <a href="https://github.com/vsoch/candy-generator" target="_blank">the candy generator</a> would be greatly appreciated.</p>

<h3 id="design-a-trick-or-treat-tool">Design a Trick or Treat Tool!</h3>

<p>As I alluded to earlier, there could be other ways to emulate trick or treating - perhaps a browser extension, a GitHub action, or something else I haven’t thought of. I would love to see others get excited about Halloween and make a new project to supplement these two.</p>

<h2 id="why-should-i-care">Why should I care?</h2>

<p>I think it’s easy in our current culture to be a consumer. We consume experiences that others provide for us, and we consume things that we buy. If you look at a typical social media account, it’s common for people to re-post, re-tweet, or just mindlessly consume what others have provided for them. I have a slightly different mindset, because I consider myself more of a producer than a consumer. I like building things, or making things, and sharing them. The lesson h ere, if there is any lesson, is that we don’t need companies or those in power to provide us with experiences. We can create out own, share it with others to enjoy, and also possibly inspire those others to make them too.</p>

<p>That’s all friends! Happy Fall, and happy Halloween season! 🍂️</p>