---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-10-16 12:00:00'
layout: post
original_url: https://vsoch.github.io//2019/nushell-plugin-golang/
title: Nushell Plugins in GoLang
---

<p><a href="https://github.com/nushell/nushell" target="_blank">Nushell</a> is a modern shell written in
rust. I’m excited about the project because the landscape of shells hasn’t had much attention recently.
You know, most of us are generally happy with bash, and it wouldn’t typically cross our
minds that a shell might offer so much more.</p>

<blockquote>
  <p>It’s like we need a new shell! nushell?</p>
</blockquote>

<p>I’ll encourage you to take a look at the <a href="https://book.nushell.sh/" target="_blank">The Nu Book</a>
and try out some of <a href="https://quay.io/organization/nushell" target="_blank">the Nushell containers</a>
for an easy way to try it out (hint, use the devel tag for the latest builds).</p>

<h2 id="too-long-didnt-read">Too Long, Didn’t Read</h2>

<p>While GoLang isn’t ideal for developing a binary that has lots of nested and varying
inputs, in that a developer might want to develop a plugin that uses some Go library,
it’s important to have a getting started example. <a href="https://github.com/vsoch/nushell-plugin-len" target="_blank">nushell-plugin-len</a> is that starting example, and I encourage contributions
to help make it better.</p>

<div style="margin-top: 20px; margin-bottom: 20px;">
   <img src="https://raw.githubusercontent.com/vsoch/nushell-plugin-len/master/img/nushell-plugin-len.png" />
</div>

<p><br /></p>

<h2 id="empower-others-to-contribute">Empower Others to Contribute</h2>

<p>In talking about contribution of plugins, we are going
to inadvertently talk about open source communities. Here’s the thing - developing software (or being a maintainer)
is about so much more than putting code on GitHub. If a project
makes it easy (or even fun!) to contribute, the project is more likely to be successful. On the other
hand, if you choose to develop your software in a niche language, and find that only
the core maintainers are the contributors, then you will have a much harder time.
Why might this be?</p>

<ol class="custom-counter">
    <li>It's hard to contribute</li>
    <li>Communication isn't great between core developers and contributors</li>
    <li>It isn't very fun</li>
</ol>

<p>or let’s flip these so we can describe <strong>good</strong> practices:</p>

<ol class="custom-counter">
    <li>It's easy to contribute</li>
    <li>Goals and next steps are openly discussed, maintainers are friendly to new contributors and questions.</li>
    <li>Working on the project, and with the prople, is fun!</li>
</ol>

<p>You can probably think of the open source projects that you work on, and ask these three
simple questions. A project doesn’t need to score highly on all points to be good
to contribute to! For example, a project that is really fun for you might outweigh 
some negative factor that it takes a little prodding to figure out how to contribute.
A project that is really easy to contribute to might offer a lot for you to learn, so you
don’t care that much that it isn’t fun. You run into trouble when none of the positive
practices are present.</p>

<h2 id="an-elegant-plugin-system">An Elegant Plugin System</h2>

<p>This is exactly what makes the <a href="https://github.com/nushell/contributor-book/blob/master/en/plugins.md" target="_blank">nushell plugin framework</a> so elegant. While the shell itself is implemented in rust (and rust
is a <a href="https://jmmv.dev/2018/06/rust-review-learning-curve.html" target="_blank">steep learning curve</a>,
the plugin framework allows for <strong>any binary on the path</strong> to be discovered and registered
as a plugin.</p>

<p>Let that sink in! Any binary that is named appropriately and can return an understood set
of json responses can act as a plugin! This means that you don’t need to know rust to 
develop for nushell. You can create plugins in Python, ruby, GoLang (this example <a href="https://github.com/vsoch/nushell-plugin-len" target="_blank">here</a>), or pretty much whatever your little heart desires.</p>

<p>It’s also elegant because the discovery happens when the user starts the shell. I’ve
seen a lot of plugin developers require the plugin to be compiled alongside the software,
and while this is sometimes necessary, it makes it really hard to ask to use your special plugin
on a shared resource, or even convince another person to re-compile their software.
Given this discovery, I have questions about best practices for security when using nushell,
but this is outside the scope of this post.</p>

<h2 id="how-does-it-work">How does it work?</h2>
<p>You can imagine it like a conversation between Nushell and the plugin. This
is what happens after Nushell finds a binary on the path that starts with nu_plugin_*:</p>

<h3 id="discovery">Discovery</h3>

<ol class="custom-counter">
    <li><span style="color: purple;">(Nushell)</span> "Hello nu_plugin, can you tell me how to configure you?"</li>
    <li><span style="color: darkgreen;">(Plugin)</span> "Sure nushell, my name is "len" and here is my usage and other metadata."</li>
    <li><span style="color: purple;">(Nushell)</span> "Your metadata looks great! I'll register you so the user can run 'len' to interact"</li>
</ol>

<h3 id="interaction">Interaction</h3>

<p>We now add the user, who wants to calculate the length of some input, to the conversation.</p>

<ol class="custom-counter">
    <li><span style="color: orangered;">(Edward)</span> "Oh! I know that 'len' is installed, let's ask for help len"</li>
    <li><span style="color: purple;">(Nushell)</span> "Oh yeah! I know about nu_plugin_len, here is his usage."</li>
    <li><span style="color: orangered;">(Edward)</span> "Okay, I want to know the length of this string."</li>
    <li><span style="color: purple;">(Nushell)</span> "Hey nu_plugin_len, I know you're a filter, I'm going to start the filter."</li>
    <li><span style="color: darkgreen;">(Plugin)</span> "Ready!"</li>
    <li><span style="color: purple;">(Nushell)</span> "Great! Here is the content to calculate the length for!"</li>
    <li><span style="color: darkgreen;">(Plugin)</span> "The length is 9!"</li>
    <li><span style="color: purple;">(Nushell)</span> "Thanks! We're all done now, I'm going to end the filter."</li>
    <li><span style="color: darkgreen;">(Plugin)</span> "Goodbye!"</li>
</ol>

<h3 id="json-specification">Json Specification</h3>

<p>While in my head I like to imagine binaries talking with one another, in
reality this is actually working by way of the <a href="https://www.jsonrpc.org/specification">JsonRPC</a>
specification, so the interactions are json messages. I’ll show you some examples in this post.</p>

<h2 id="building-the-plugin-locally">Building the Plugin Locally</h2>

<p>But first I’ll show you how to build the binary in Go, which we are going to
use next to show the json responses. Clone the repository:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>git clone https://github.com/vsoch/nushell-plugin-len
<span class="nv">$ </span><span class="nb">cd </span>nushell-plugin-len
</code></pre></div></div>

<p>and if you have Go installed locally, you can use the Makefile to build the plugin</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>make
go build <span class="nt">-o</span> nu_plugin_len
</code></pre></div></div>

<p>And given that we have a sense of the inputs that the plugin expects, we can
first test outside of nushell. We’ll start with discovery.</p>

<h3 id="discovery-1">Discovery</h3>

<p>Discovery happens by way of passing a json object with method == config.</p>

<blockquote>
  <p>Hello nu_plugin, can you tell me how to configure you?</p>
</blockquote>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>./nu_plugin_len

<span class="c"># This would be passed from Nushell when the plugin is found on the path</span>
<span class="o">{</span><span class="s2">"method"</span>:<span class="s2">"config"</span><span class="o">}</span>
</code></pre></div></div>

<p>The plugin responds with a jsonRPC response, with status “Ok”. I’ve parsed this
out from a single line for your readability.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">{</span>
  <span class="s2">"jsonrpc"</span>: <span class="s2">"2.0"</span>,
  <span class="s2">"method"</span>: <span class="s2">"response"</span>,
  <span class="s2">"params"</span>: <span class="o">{</span>
    <span class="s2">"Ok"</span>: <span class="o">{</span>
      <span class="s2">"name"</span>: <span class="s2">"len"</span>,
      <span class="s2">"usage"</span>: <span class="s2">"Return the length of a string"</span>,
      <span class="s2">"positional"</span>: <span class="o">[]</span>,
      <span class="s2">"rest_positional"</span>: null,
      <span class="s2">"named"</span>: <span class="o">{}</span>,
      <span class="s2">"is_filter"</span>: <span class="nb">true</span>
    <span class="o">}</span>
  <span class="o">}</span>
<span class="o">}</span>
</code></pre></div></div>

<h3 id="start-and-end-filter">Start and End Filter</h3>

<p>Starting and ending filters pass the method as begin_filter or end_filter.</p>

<blockquote>
  <p>(Nushell) Hey nu_plugin_len, I know you’re a filter, I’m going to start the filter.</p>
</blockquote>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>./nu_plugin_len
<span class="o">{</span><span class="s2">"method"</span>:<span class="s2">"begin_filter"</span><span class="o">}</span>

<span class="c"># okay, I'm good with that.</span>
<span class="o">{</span><span class="s2">"jsonrpc"</span>:<span class="s2">"2.0"</span>,<span class="s2">"method"</span>:<span class="s2">"response"</span>,<span class="s2">"params"</span>:<span class="o">{</span><span class="s2">"Ok"</span>:[]<span class="o">}}</span>
</code></pre></div></div>

<blockquote>
  <p>(Nushell) Thanks! We’re all done now, I’m going to end the filter.</p>
</blockquote>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="o">{</span><span class="s2">"method"</span>:<span class="s2">"end_filter"</span><span class="o">}</span>

<span class="c"># sounds good. See you later!</span>
<span class="o">{</span><span class="s2">"jsonrpc"</span>:<span class="s2">"2.0"</span>,<span class="s2">"method"</span>:<span class="s2">"response"</span>,<span class="s2">"params"</span>:<span class="o">{</span><span class="s2">"Ok"</span>:[]<span class="o">}}</span>
</code></pre></div></div>

<p>The plugin exits after we finish.</p>

<h3 id="calculate-length">Calculate Length</h3>

<p>The actual request to do the filter passes an item, and the item is identified
as a String primitive. I am going to split the stream input into two lines
for your readability:</p>

<blockquote>
  <p>(Nushell) Great! Here is the content to calculate the length for!</p>
</blockquote>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>./nu_plugin_len
<span class="o">{</span><span class="s2">"method"</span>:<span class="s2">"filter"</span>, <span class="s2">"params"</span>: <span class="o">{</span><span class="s2">"item"</span>: <span class="o">{</span><span class="s2">"Primitive"</span>: <span class="o">{</span><span class="s2">"String"</span>: <span class="s2">"oogabooga"</span><span class="o">}}</span>, <span class="se">\</span>
                               <span class="s2">"tag"</span>:<span class="o">{</span><span class="s2">"anchor"</span>:null,<span class="s2">"span"</span>:<span class="o">{</span><span class="s2">"end"</span>:10,<span class="s2">"start"</span>:12<span class="o">}}}}</span>
</code></pre></div></div>

<blockquote>
  <p>The length is 9!</p>
</blockquote>

<p>And here is the response, again printed for easy reading:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">{</span>
  <span class="s2">"jsonrpc"</span>: <span class="s2">"2.0"</span>,
  <span class="s2">"method"</span>: <span class="s2">"response"</span>,
  <span class="s2">"params"</span>: <span class="o">{</span>
    <span class="s2">"Ok"</span>: <span class="o">[</span>
      <span class="o">{</span>
        <span class="s2">"Ok"</span>: <span class="o">{</span>
          <span class="s2">"Value"</span>: <span class="o">{</span>
            <span class="s2">"item"</span>: <span class="o">{</span>
              <span class="s2">"Primitive"</span>: <span class="o">{</span>
                <span class="s2">"Int"</span>: 9
              <span class="o">}</span>
            <span class="o">}</span>,
            <span class="s2">"tag"</span>: <span class="o">{</span>
              <span class="s2">"anchor"</span>: null,
              <span class="s2">"span"</span>: <span class="o">{</span>
                <span class="s2">"end"</span>: 2,
                <span class="s2">"start"</span>: 0
              <span class="o">}</span>
            <span class="o">}</span>
          <span class="o">}</span>
        <span class="o">}</span>
      <span class="o">}</span>
    <span class="o">]</span>
  <span class="o">}</span>
<span class="o">}</span>
</code></pre></div></div>

<p>And actually, when you are testing locally (and perhaps don’t have a tag, since it comes
from the input stream) I’ve designed the plugin so that it will generate one for you.
Here is an example without the tag:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>./nu_plugin_len
<span class="o">{</span><span class="s2">"method"</span>:<span class="s2">"filter"</span>, <span class="s2">"params"</span>: <span class="o">{</span><span class="s2">"item"</span>: <span class="o">{</span><span class="s2">"Primitive"</span>: <span class="o">{</span><span class="s2">"String"</span>: <span class="s2">"oogabooga"</span><span class="o">}}}}</span>

<span class="o">{</span>
  <span class="s2">"jsonrpc"</span>: <span class="s2">"2.0"</span>,
  <span class="s2">"method"</span>: <span class="s2">"response"</span>,
  <span class="s2">"params"</span>: <span class="o">{</span>
    <span class="s2">"Ok"</span>: <span class="o">[</span>
      <span class="o">{</span>
        <span class="s2">"Ok"</span>: <span class="o">{</span>
          <span class="s2">"Value"</span>: <span class="o">{</span>
            <span class="s2">"item"</span>: <span class="o">{</span>
              <span class="s2">"Primitive"</span>: <span class="o">{</span>
                <span class="s2">"Int"</span>: 9
              <span class="o">}</span>
            <span class="o">}</span>,
            <span class="s2">"tag"</span>: <span class="o">{</span>
              <span class="s2">"anchor"</span>: null,
              <span class="s2">"span"</span>: <span class="o">{</span>
                <span class="s2">"end"</span>: 0,
                <span class="s2">"start"</span>: 0
              <span class="o">}</span>
            <span class="o">}</span>
          <span class="o">}</span>
        <span class="o">}</span>
      <span class="o">}</span>
    <span class="o">]</span>
  <span class="o">}</span>
<span class="o">}</span>
</code></pre></div></div>

<p><br /></p>

<h2 id="building-the-plugin-with-nu">Building the Plugin with Nu</h2>

<p>Once local testing had taken me as far as I could go, I knew that I wanted to test with 
nushell. Containers to the rescue!  We are going to build a container first with
a GoLang base to compile the plugin, and then we will copy
the binary into <a href="https://quay.io/repository/nushell/nu-base" target="_blank">quay.io/nushell/nu-base</a> under /usr/local/bin for nushell to discover. Here is the relevant Dockerfile to do that:</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
FROM golang:1.13.1 as builder
# docker build -t vanessa/nushell-plugin-len .
WORKDIR /code
COPY . /code
RUN make
FROM quay.io/nushell/nu-base:devel
LABEL Maintainer vsochat@stanford.edu
COPY --from=builder /code/nu_plugin_len /usr/local/bin

</code></pre></div></div>

<p>Notice that we are using the devel tag of nu-base to ensure we get a recent build.
And then build it!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>docker build <span class="nt">-t</span> vanessa/nu-plugin-len <span class="nb">.</span>
</code></pre></div></div>

<p>Then shell inside - the default entrypoint is already the nushell.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>docker run <span class="nt">-it</span> vanessa/nu-plugin-len
</code></pre></div></div>

<p>Once inside, you can use <code class="highlighter-rouge">nu -l trace</code> to confirm that nu discovered your plugin
on the path. Here we see that it did!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>/code<span class="o">(</span>add/circleci<span class="o">)&gt;</span> nu <span class="nt">-l</span> trace
...
 TRACE nu::cli <span class="o">&gt;</span> Trying <span class="s2">"/usr/local/bin/nu_plugin_len"</span>
</code></pre></div></div>

<p>You can also (for newer versions of nu &gt; 0.2.0) use help to see the command:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>/code<span class="o">(</span>master<span class="o">)&gt;</span> <span class="nb">help </span>len
Return the length of a string

Usage:
  <span class="o">&gt;</span> len 

/code<span class="o">(</span>master<span class="o">)&gt;</span> 
</code></pre></div></div>

<p>Now try calculating the length of something!  Here we pass the string “four”
into the length function, and nushell uses the plugin (with all the json message
passing behind the scenes) to return to us that the answer is 4.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>/code<span class="o">(</span>master<span class="o">)&gt;</span> <span class="nb">echo </span>four | len
━━━━━━━━━━━
 &lt;unknown&gt; 
───────────
         4 
━━━━━━━━━━━
</code></pre></div></div>

<p>The plugin is a filter, which is why we can pipe into it.</p>

<p>Here is a slightly more fun example - here we are in a directory with
one file named “myname” that is empty.</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>/tmp/test&gt; ls
━━━━━━━━┯━━━━━━┯━━━━━━━━━━┯━━━━━━┯━━━━━━━━━━━━━━━━┯━━━━━━━━━━━━━━━━
 name   │ type │ readonly │ size │ accessed       │ modified 
────────┼──────┼──────────┼──────┼────────────────┼────────────────
 myname │ File │          │  —   │ 41 seconds ago │ 41 seconds ago 
━━━━━━━━┷━━━━━━┷━━━━━━━━━━┷━━━━━━┷━━━━━━━━━━━━━━━━┷━━━━━━━━━━━━━━━━
</code></pre></div></div>

<p>Try listing, getting the name, and calculating the length.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>/tmp/test&gt; <span class="nb">ls</span> | get name | len
━━━━━━━━━━━
 &lt;unknown&gt; 
───────────
         6 
━━━━━━━━━━━
</code></pre></div></div>

<p>The above calculated that the length of “myname” is 6. Add another file to see the table get another row</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nb">touch </span>four
</code></pre></div></div>
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>/tmp/test&gt; <span class="nb">ls</span> | get name | len 
━━━┯━━━━━━━━━━━
 <span class="c"># │ &lt;unknown&gt; </span>
───┼───────────
 0 │         4 
 1 │         6 
━━━┷━━━━━━━━━━━
</code></pre></div></div>

<h3 id="docker-hub">Docker Hub</h3>

<p>If you don’t want to build but just want to play with the plugin,
you can pull directly from Docker Hub</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>docker pull vanessa/nu-plugin-len
<span class="nv">$ </span>docker run <span class="nt">-it</span> vanessa/nu-plugin-len
</code></pre></div></div>

<h3 id="logging">Logging</h3>

<p>A lot of figuring this out was trial and error, and then asking for feedback
or help on the nushell discord channel. The biggest help, by far, was
logging to a temporary file at <code class="highlighter-rouge">/tmp/nu-plugin-len.log</code> to fully see what
was being passed from nushell.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>/tmp&gt; <span class="nb">cat</span> /tmp/nu_plugin_len.log
nu_plugin_len 2019/10/16 15:01:02 Request <span class="k">for </span>config map[jsonrpc:2.0 method:config params:[]]
nu_plugin_len 2019/10/16 15:01:16 Request <span class="k">for </span>begin filter map[jsonrpc:2.0 method:begin_filter...
nu_plugin_len 2019/10/16 15:01:16 Request <span class="k">for </span>filter map[jsonrpc:2.0 method:filter params:...
nu_plugin_len 2019/10/16 15:01:16 Request <span class="k">for </span>end filter map[jsonrpc:2.0 method:end_filter params:[]]
</code></pre></div></div>

<p>I’d even go as far to say that nushell
should have some “standard” way for organizing logs for plugins, since you can’t
write logs to stdout.</p>

<h2 id="why">Why?</h2>

<p>The “So What” is that I want other developers and research software engineers to be empowered
to contribute plugins, in whatever language they desire! Along with learning a bit about
nushell and GoLang, this was my main goal for developing this. I hope that it is useful to you!
Please <a href="https://github.com/vsoch/nushell-plugin-len" target="_blank">contribute to nushell-plugin-len</a>
to make it better.</p>