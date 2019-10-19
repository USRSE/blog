---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-10-18 03:30:00'
layout: post
original_url: https://vsoch.github.io//2019/nushell-plugins/
title: Nushell Pokemon Plugin in Python
---

<p>I was having some trouble figuring out (based on looking at traces and rust source code)
what exactly kind of Json structure was expected by nushell for plugins. After
some random testing, I stumbled on the basics and want to share what I learned.
I also want to say thank you to <a href="https://twitter.com/andras_io" target="_blank">andras_io</a>
who I’ve been chatting up a storm on Discord, and having quite a bit of fun
untangling how this works!</p>

<h2 id="config">Config</h2>

<p>When nushell first discovers a plugin, done by way of being on the path and having
a name like “nu_plugin_*,” it will request to get the plugin config, something like this:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">{</span><span class="s2">"method"</span>: <span class="s2">"config"</span><span class="o">}</span>
</code></pre></div></div>

<p>And then your plugin might return a configuration object (shown below) with 
a set of named arguments, meaning that the user requests them by name (not positional arguments):
Here is an example config that shows three named arguments:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">{</span>
  <span class="s2">"name"</span>: <span class="s2">"pokemon"</span>,
  <span class="s2">"usage"</span>: <span class="s2">"Catch an asciinema pokemon on demand"</span>,
  <span class="s2">"positional"</span>: <span class="o">[]</span>,
  <span class="s2">"rest_positional"</span>: null,
  <span class="s2">"named"</span>: <span class="o">{</span>
    <span class="s2">"switch"</span>: <span class="s2">"Switch"</span>,
    <span class="s2">"mandatory"</span>: <span class="o">{</span>
      <span class="s2">"Mandatory"</span>: <span class="s2">"String"</span>
    <span class="o">}</span>,
    <span class="s2">"optional"</span>: <span class="o">{</span>
      <span class="s2">"Optional"</span>: <span class="s2">"String"</span>
    <span class="o">}</span>
  <span class="o">}</span>,
  <span class="s2">"is_filter"</span>: <span class="nb">false</span>
<span class="o">}</span>
</code></pre></div></div>

<p>What are the arguments under named?</p>

<ol class="custom-counter">
  <li>--switch shows a flag that is a boolean, so it's present (or not).</li>
  <li>--mandatory is an example of a required string</li>
  <li>--optional is an example of an optional string</li>
</ol>

<p>When executed, it looks like this:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>Catch an asciinema pokemon on demand

Usage:
  <span class="o">&gt;</span> pokemon <span class="o">{</span>flags<span class="o">}</span> 

flags:
  <span class="nt">--switch</span>
  <span class="nt">--mandatory</span> &lt;String&gt; <span class="o">(</span>required parameter<span class="o">)</span>
  <span class="nt">--optional</span> &lt;String&gt;
</code></pre></div></div>

<p>Each of these flags is defined under “named” in the plugin configuration above.</p>

<h2 id="sink">Sink</h2>

<p>See the boolean is_filter is set to false in the configuration? There are two kinds of plugins. A filter
is akin to a pipe (for an example see <a href="https://vsoch.github.io/2019/nushell-plugin-golang/" target="_blank">nushell-plugin-golang</a>), and a sink is just going to execute the plugin and give it total access
to dump whatever it likes to stdout (akin to dumping in a sink I suppose?).
I’m not sure this is the best way to describe it, but it’s how I’m trying to understand it.
Now let’s look at an example. Given the above, the minimum valid command would have <code class="highlighter-rouge">--mandatory &lt;value&gt;</code></p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>pokemon <span class="nt">--mandatory</span> avocado
</code></pre></div></div>

<p>Here we are providing all the options!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>pokemon <span class="nt">--switch</span> <span class="nt">--mandatory</span> MANDATORYARG <span class="nt">--optional</span> OPTIONALARG
</code></pre></div></div>

<p>Now let’s look at how nu will take this input from the user, and pass it to the plugin 
(note that I pretty printed this for your viewing, it’s normally a single flattened line):</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">{</span>
  <span class="s2">"jsonrpc"</span>: <span class="s2">"2.0"</span>,
  <span class="s2">"method"</span>: <span class="s2">"sink"</span>,
  <span class="s2">"params"</span>: <span class="o">[</span>
    <span class="o">{</span>
      <span class="s2">"args"</span>: <span class="o">{</span>
        <span class="s2">"positional"</span>: null,
        <span class="s2">"named"</span>: <span class="o">{</span>
          <span class="s2">"switch"</span>: <span class="o">{</span>
            <span class="s2">"tag"</span>: <span class="o">{</span>
              <span class="s2">"anchor"</span>: null,
              <span class="s2">"span"</span>: <span class="o">{</span>
                <span class="s2">"start"</span>: 58,
                <span class="s2">"end"</span>: 64
              <span class="o">}</span>
            <span class="o">}</span>,
            <span class="s2">"item"</span>: <span class="o">{</span>
              <span class="s2">"Primitive"</span>: <span class="o">{</span>
                <span class="s2">"Boolean"</span>: <span class="nb">true</span>
              <span class="o">}</span>
            <span class="o">}</span>
          <span class="o">}</span>,
          <span class="s2">"mandatory"</span>: <span class="o">{</span>
            <span class="s2">"tag"</span>: <span class="o">{</span>
              <span class="s2">"anchor"</span>: null,
              <span class="s2">"span"</span>: <span class="o">{</span>
                <span class="s2">"start"</span>: 20,
                <span class="s2">"end"</span>: 32
              <span class="o">}</span>
            <span class="o">}</span>,
            <span class="s2">"item"</span>: <span class="o">{</span>
              <span class="s2">"Primitive"</span>: <span class="o">{</span>
                <span class="s2">"String"</span>: <span class="s2">"MANDATORYARG"</span>
              <span class="o">}</span>
            <span class="o">}</span>
          <span class="o">}</span>,
          <span class="s2">"optional"</span>: <span class="o">{</span>
            <span class="s2">"tag"</span>: <span class="o">{</span>
              <span class="s2">"anchor"</span>: null,
              <span class="s2">"span"</span>: <span class="o">{</span>
                <span class="s2">"start"</span>: 44,
                <span class="s2">"end"</span>: 55
              <span class="o">}</span>
            <span class="o">}</span>,
            <span class="s2">"item"</span>: <span class="o">{</span>
              <span class="s2">"Primitive"</span>: <span class="o">{</span>
                <span class="s2">"String"</span>: <span class="s2">"OPTIONALARG"</span>
              <span class="o">}</span>
            <span class="o">}</span>
          <span class="o">}</span>
        <span class="o">}</span>
      <span class="o">}</span>,
      <span class="s2">"name_tag"</span>: <span class="o">{</span>
        <span class="s2">"anchor"</span>: null,
        <span class="s2">"span"</span>: <span class="o">{</span>
          <span class="s2">"start"</span>: 0,
          <span class="s2">"end"</span>: 7
        <span class="o">}</span>
      <span class="o">}</span>
    <span class="o">}</span>,
    <span class="o">[]</span>
  <span class="o">]</span>
<span class="o">}</span>
</code></pre></div></div>

<p>The bulk of the above are the requested arguments from the user to pass to sink.
An important note is that if a parameter isn’t provided and is optional, 
it won’t be parsed in the params string, so your code needs to deal with this
appropriately. I’m also not sure why a list is passed to params with the second (index 1) being empty.</p>

<h2 id="pokemon-example">Pokemon Example</h2>

<p>Let’s have a little more fun and show a pokemon example. I developed this in
Python so I could use my <a href="https://github.com/vsoch/pokemon">pokemon</a> library.
The source code for the example is at <a href="https://github.com/vsoch/nushell-plugin-pokemon" target="_blank">vsoch/nushell-plugin-pokemon</a> First I’ll show you how it works:</p>



<p>In the above you’ll notice I also implemented <code class="highlighter-rouge">pokemon --help</code> since people would likely try it,
and I was able to catch a pokemon, list pokemon (sorted and unsorted), generate an avatar,
and request a specific pokemon by name. We can look at the main logic of the Python script.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">for</span> <span class="n">line</span> <span class="ow">in</span> <span class="n">fileinput</span><span class="o">.</span><span class="nb">input</span><span class="p">():</span>

    <span class="n">x</span> <span class="o">=</span> <span class="n">json</span><span class="o">.</span><span class="n">loads</span><span class="p">(</span><span class="n">line</span><span class="p">)</span>
    <span class="n">method</span> <span class="o">=</span> <span class="n">x</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">"method"</span><span class="p">)</span>

    <span class="c1"># Keep log of requests from nu
</span>    <span class="n">logging</span><span class="o">.</span><span class="n">info</span><span class="p">(</span><span class="s">"REQUEST </span><span class="si">%</span><span class="s">s"</span> <span class="o">%</span> <span class="n">line</span><span class="p">)</span>
    <span class="n">logging</span><span class="o">.</span><span class="n">info</span><span class="p">(</span><span class="s">"METHOD </span><span class="si">%</span><span class="s">s"</span> <span class="o">%</span> <span class="n">method</span><span class="p">)</span>

    <span class="c1"># Case 1: Nu is asking for the config to discover the plugin
</span>    <span class="k">if</span> <span class="n">method</span> <span class="o">==</span> <span class="s">"config"</span><span class="p">:</span>
        <span class="n">plugin_config</span> <span class="o">=</span> <span class="n">get_config</span><span class="p">()</span>
        <span class="n">logging</span><span class="o">.</span><span class="n">info</span><span class="p">(</span><span class="s">"plugin-config: </span><span class="si">%</span><span class="s">s"</span> <span class="o">%</span> <span class="n">json</span><span class="o">.</span><span class="n">dumps</span><span class="p">(</span><span class="n">plugin_config</span><span class="p">))</span>
        <span class="n">print_good_response</span><span class="p">(</span><span class="n">plugin_config</span><span class="p">)</span>
        <span class="k">break</span>

    <span class="c1"># Case 3: A filter must return the item filtered with a tag
</span>    <span class="k">elif</span> <span class="n">method</span> <span class="o">==</span> <span class="s">"sink"</span><span class="p">:</span>

        <span class="c1"># Parse the parameters into a simpler format, example for each type
</span>        <span class="c1"># {'switch': True, 'mandatory': 'MANDATORYARG', 'optional': 'OPTIONALARG'}
</span>        <span class="n">params</span> <span class="o">=</span> <span class="n">parse_params</span><span class="p">(</span><span class="n">x</span><span class="p">[</span><span class="s">'params'</span><span class="p">])</span>
        <span class="n">logging</span><span class="o">.</span><span class="n">info</span><span class="p">(</span><span class="s">"PARAMS </span><span class="si">%</span><span class="s">s"</span> <span class="o">%</span> <span class="n">params</span><span class="p">)</span>

        <span class="k">if</span> <span class="n">params</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'catch'</span><span class="p">,</span> <span class="bp">False</span><span class="p">):</span>
            <span class="n">logging</span><span class="o">.</span><span class="n">info</span><span class="p">(</span><span class="s">"We want to catch a random pokemon!"</span><span class="p">)</span>
            <span class="n">catch_pokemon</span><span class="p">()</span>

        <span class="k">elif</span> <span class="n">params</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'list'</span><span class="p">,</span> <span class="bp">False</span><span class="p">):</span>
            <span class="n">logging</span><span class="o">.</span><span class="n">info</span><span class="p">(</span><span class="s">"We want to list Pokemon names."</span><span class="p">)</span>
            <span class="n">list_pokemon</span><span class="p">()</span>

        <span class="k">elif</span> <span class="n">params</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'list-sorted'</span><span class="p">,</span> <span class="bp">False</span><span class="p">):</span>
            <span class="n">logging</span><span class="o">.</span><span class="n">info</span><span class="p">(</span><span class="s">"We want to list sorted Pokemon names."</span><span class="p">)</span>
            <span class="n">list_pokemon</span><span class="p">(</span><span class="n">do_sort</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>

        <span class="k">elif</span> <span class="n">params</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'avatar'</span><span class="p">,</span> <span class="s">''</span><span class="p">)</span> <span class="o">!=</span> <span class="s">''</span><span class="p">:</span>
            <span class="n">logging</span><span class="o">.</span><span class="n">info</span><span class="p">(</span><span class="s">"We want a pokemon avatar!"</span><span class="p">)</span>
            <span class="n">catch</span> <span class="o">=</span> <span class="n">get_avatar</span><span class="p">(</span><span class="n">params</span><span class="p">[</span><span class="s">'avatar'</span><span class="p">])</span>

        <span class="k">elif</span> <span class="n">params</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'pokemon'</span><span class="p">,</span> <span class="s">''</span><span class="p">)</span> <span class="o">!=</span> <span class="s">''</span><span class="p">:</span>
            <span class="n">get_ascii</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="n">params</span><span class="p">[</span><span class="s">'pokemon'</span><span class="p">])</span>

        <span class="k">elif</span> <span class="n">params</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'help'</span><span class="p">,</span> <span class="bp">False</span><span class="p">):</span>
            <span class="k">print</span><span class="p">(</span><span class="n">get_usage</span><span class="p">())</span>

        <span class="k">else</span><span class="p">:</span>
            <span class="k">print</span><span class="p">(</span><span class="n">get_usage</span><span class="p">())</span>

        <span class="k">break</span>

    <span class="k">else</span><span class="p">:</span>
        <span class="k">break</span>
</code></pre></div></div>

<p>The above is fairly simple - we are basically parsing what is passed from nu, and deriving the method to
determine what to do! Instead of needing begin_filter, end_filter, and filter
(if is_filter was true) we just need to have a method to return the config
and then deal with parsing the parameters for the config.</p>

<p>Importantly, each of the functions to get an avatar, list_pokemon and catch_pokemon
just prints content to the terminal (stdout). This we can do because the plugin
is a sink (is_filter is False).</p>

<p>For the complete code and more description for how to interact with the
container, view logs, and debug, see <a href="https://github.com/vsoch/nushell-plugin-pokemon" target="_blank">vsoch/nushell-plugin-pokemon</a>.</p>

<h2 id="so-what">So What?</h2>

<p>I think nu could be really awesome for creating tools (and scripts that use them) 
for research and science. I’m hoping that by creating examples that might help you to get started,
you can do awesome things, because I certainly am not a very good scientist.</p>