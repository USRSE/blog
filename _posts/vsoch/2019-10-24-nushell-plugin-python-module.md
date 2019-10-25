---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-10-24 04:30:00'
layout: post
original_url: https://vsoch.github.io//2019/nushell-plugin-python-module/
title: Nushell Plugin Library for Python
---

<p>Okay, I’ve been going a little nuts with <a href="https://www.github.com/nushell/nushell" target="_blank">nushell</a>
lately, and the reason is because I have a thought process like:</p>

<ol class="custom-counter">
  <li>Oh, I want to build this thing!</li>
  <li>Maybe there could be a tool to help to build the thing?</li>
  <li>Hmm, who is going to make it...</li>
  <li>I guess I can do it?</li>
</ol>

<p>And then I go off and make the tool! In this case, I had tried creating standalone
plugins with <a href="https://vsoch.github.io/2019/nushell-plugins/" target="_blank">Python</a> 
and <a href="https://vsoch.github.io/2019/nushell-plugin-golang" target="_blank">GoLang</a>, and then I realized that it would be much easier to have a module to help me out. <a href="https://vsoch.github.io/2019/nushell-golang-plugin-library/" target="_blank">Yesterday</a> I did this for GoLang, so today I figured I’d give Python a shot!
And that’s what I’ll briefly show in this post. Note that every feature implemented by
the plugins interface isn’t provided here (for example, I still need to add positional
arguments and support for pipes into a sink) but this is definitely something to get us started!
Check out the module here:</p>

<p><a href="https://github.com/vsoch/nushell-plugin-python" target="_blank">vsoch/nushell-plugin-python</a></p>

<p>Now let’s go through a few examples. For full examples, you can see the <a href="https://github.com/vsoch/nushell-plugin-python/tree/master/examples" target="_blank">examples folder</a>. Each example includes container builds,
a Makefile, a README with instructions, and of course a Python script. When appropriate, there are containers
that will help you build a standalone executable.</p>

<h2 id="filter-plugin">Filter Plugin</h2>

<p>A basic filter plugin will instantiate the <code class="language-plaintext highlighter-rouge">FilterPlugin</code> class, and then
provide a function to run for the filter. Here is a quick example script.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">#!/usr/bin/env python3
</span>
<span class="kn">from</span> <span class="nn">nushell.filter</span> <span class="kn">import</span> <span class="n">FilterPlugin</span>

<span class="c1"># Your filter function will be called by the FilterPlugin, and should
# accept the plugin and the dictionary of params
</span><span class="k">def</span> <span class="nf">runFilter</span><span class="p">(</span><span class="n">plugin</span><span class="p">,</span> <span class="n">params</span><span class="p">):</span>
    <span class="s">'''runFilter will be executed by the calling SinkPlugin when method is "sink"
    '''</span>
    <span class="c1"># Get the string primitive passed by the user
</span>    <span class="n">value</span> <span class="o">=</span> <span class="n">plugin</span><span class="o">.</span><span class="n">get_string_primitive</span><span class="p">()</span>
    <span class="c1"># Calculate the length
</span>    <span class="n">intLength</span> <span class="o">=</span> <span class="nb">len</span><span class="p">(</span><span class="n">value</span><span class="p">)</span>
    <span class="c1"># Print an integer response (can also be print_string_response)
</span>    <span class="n">plugin</span><span class="o">.</span><span class="n">print_int_response</span><span class="p">(</span><span class="n">intLength</span><span class="p">)</span>

<span class="c1"># The main function is where you create your plugin and run it.
</span><span class="k">def</span> <span class="nf">main</span><span class="p">():</span>

    <span class="c1"># Initialize a new plugin
</span>    <span class="n">plugin</span> <span class="o">=</span> <span class="n">FilterPlugin</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">"len"</span><span class="p">,</span> 
                          <span class="n">usage</span><span class="o">=</span><span class="s">"Return the length of a string"</span><span class="p">)</span>

    <span class="c1"># Run the plugin by passing your filter function
</span>    <span class="n">plugin</span><span class="o">.</span><span class="n">run</span><span class="p">(</span><span class="n">runFilter</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="s">'__main__'</span><span class="p">:</span>
    <span class="n">main</span><span class="p">()</span>
</code></pre></div></div>

<p>Notably, your filter function should taken a plugin and parsed command line
parameters (dictionary) as arguments. You can use the plugin to perform
several needed functions to send responses back to nushell, or log to <code class="language-plaintext highlighter-rouge">/tmp/nushell-plugin-&lt;name&gt;.log</code>:
Generally, the functions of interest will be to get or print a string or int response
that is passed to or from Nushell.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1"># basic functions to get / print strings and ints
</span><span class="n">plugin</span><span class="o">.</span><span class="n">get_string_primitive</span><span class="p">()</span>
<span class="n">plugin</span><span class="o">.</span><span class="n">get_int_primitive</span><span class="p">()</span>
<span class="n">plugin</span><span class="o">.</span><span class="n">print_int_response</span><span class="p">()</span>
<span class="n">plugin</span><span class="o">.</span><span class="n">print_string_response</span><span class="p">()</span>
</code></pre></div></div>

<p>or to log something to the logfile:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1"># The logger logs to /tmp/nu_plugin_&lt;name&gt;.log
</span><span class="n">plugin</span><span class="o">.</span><span class="n">logger</span><span class="o">.</span><span class="n">info</span><span class="p">(</span><span class="s">"This is some information"</span><span class="p">)</span>
<span class="n">plugin</span><span class="o">.</span><span class="n">logger</span><span class="o">.</span><span class="n">debug</span><span class="p">(</span><span class="s">"The answer is moo."</span><span class="p">)</span>
<span class="n">plugin</span><span class="o">.</span><span class="n">logger</span><span class="o">.</span><span class="n">warning</span><span class="p">(</span><span class="s">"Stinky socks!"</span><span class="p">)</span>
<span class="n">plugin</span><span class="o">.</span><span class="n">logger</span><span class="o">.</span><span class="n">error</span><span class="p">(</span><span class="s">"It's all crashed."</span><span class="p">)</span>
</code></pre></div></div>

<p>The default level is debug, and you can also disable logging when you create your
plugin.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">plugin</span> <span class="o">=</span> <span class="n">FilterPlugin</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">"len"</span><span class="p">,</span> 
                      <span class="n">usage</span><span class="o">=</span><span class="s">"Return the length of a string"</span><span class="p">,</span>
                      <span class="n">logging</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
</code></pre></div></div>

<h2 id="sink-plugin">Sink Plugin</h2>

<p>A sink plugin will instantiate the <code class="language-plaintext highlighter-rouge">SinkPlugin</code> class, and then hand off
stdin (via a temporary file) to a sink function that you write.
Here is a dummy example.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">#!/usr/bin/env python3
</span>
<span class="kn">from</span> <span class="nn">nushell.sink</span> <span class="kn">import</span> <span class="n">SinkPlugin</span>

<span class="c1"># Your sink function will be called by the sink Plugin, and should
# accept the plugin and the dictionary of params
</span><span class="k">def</span> <span class="nf">sink</span><span class="p">(</span><span class="n">plugin</span><span class="p">,</span> <span class="n">params</span><span class="p">):</span>
    <span class="s">'''sink will be executed by the calling SinkPlugin when method is "sink"
    '''</span>
    <span class="n">message</span> <span class="o">=</span> <span class="s">"Hello"</span>
    <span class="n">excited</span> <span class="o">=</span> <span class="n">params</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">"excited"</span><span class="p">,</span> <span class="bp">False</span><span class="p">)</span>
    <span class="n">name</span> <span class="o">=</span> <span class="n">params</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">"name"</span><span class="p">,</span> <span class="s">""</span><span class="p">)</span>
    <span class="c1"># If we have a name, add to message
</span>    <span class="n">message</span> <span class="o">=</span> <span class="s">"</span><span class="si">%</span><span class="s">s </span><span class="si">%</span><span class="s">s"</span> <span class="o">%</span><span class="p">(</span><span class="n">message</span><span class="p">,</span> <span class="n">name</span><span class="p">)</span>
    <span class="c1"># Are we excited?
</span>    <span class="k">if</span> <span class="n">excited</span><span class="p">:</span>
        <span class="n">message</span> <span class="o">+=</span> <span class="s">"!"</span>
    <span class="k">print</span><span class="p">(</span><span class="n">message</span><span class="p">)</span>


<span class="c1"># The main function is where you create your plugin and run it.
</span><span class="k">def</span> <span class="nf">main</span><span class="p">():</span>

    <span class="c1"># Initialize a new plugin
</span>    <span class="n">plugin</span> <span class="o">=</span> <span class="n">SinkPlugin</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">"hello"</span><span class="p">,</span> 
                        <span class="n">usage</span><span class="o">=</span><span class="s">"A friendly plugin"</span><span class="p">)</span>
    <span class="c1"># Add named arguments (notice we check for in params in sink function)
</span>    <span class="c1"># add_named_argument(name, argType, syntaxShape=None, usage=None)
</span>    <span class="n">plugin</span><span class="o">.</span><span class="n">add_named_argument</span><span class="p">(</span><span class="s">"excited"</span><span class="p">,</span> <span class="s">"Switch"</span><span class="p">,</span> <span class="n">usage</span><span class="o">=</span><span class="s">"add an exclamation point!"</span><span class="p">)</span>
    <span class="n">plugin</span><span class="o">.</span><span class="n">add_named_argument</span><span class="p">(</span><span class="s">"name"</span><span class="p">,</span> <span class="s">"Optional"</span><span class="p">,</span> <span class="s">"String"</span><span class="p">,</span> <span class="n">usage</span><span class="o">=</span><span class="s">"say hello to..."</span><span class="p">)</span>
    <span class="c1"># Run the plugin by passing your sink function
</span>    <span class="n">plugin</span><span class="o">.</span><span class="n">run</span><span class="p">(</span><span class="n">sink</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="s">'__main__'</span><span class="p">:</span>
    <span class="n">main</span><span class="p">()</span>
</code></pre></div></div>

<p>Notice that the main difference here is that we are adding named arguments.
A switch is basically a boolean, and an Optional (or Mandatory) argument can be a 
String, Int, or other <a href="https://github.com/nushell/nushell/blob/master/src/parser/hir/syntax_shape.rs#L49" target="_blank">valid types</a>.</p>

<h2 id="single-binary">Single Binary</h2>

<p>In that you are able to compile your module with <a href="https://pyinstaller.readthedocs.io/en/stable/operating-mode.html" target="_blank">pyinstaller</a> you can build your python script as a simple binary, and one that doesn’t even need nushell installed as a Python module anymore. Why might you want to do this? It will mean that your plugin is a single file (binary) and you don’t need to rely on modules elsewhere in the system. I suspect there are other ways to compile
python into a single binary (e.g., cython) but this was the first I tried, and fairly straight forward.
If you find a different or better way, please contribute to this code base!
The examples for <a href="https://github.com/vsoch/nushell-plugin-python/tree/master/examples/len" target="_blank">len</a>
(filter) and <a href="https://github.com/vsoch/nushell-plugin-python/tree/master/examples/hello" target="_blank">hello</a> (sink) demonstrate this, while <a href="https://github.com/vsoch/nushell-plugin-python/tree/master/examples/pokemon" target="_blank">pokemon</a> didn’t work due to an external data file.</p>

<p>That’s it! I can’t believe I pulled this off in under a day, please contribute to make it better!</p>