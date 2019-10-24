---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-10-23 02:24:00'
layout: post
original_url: https://vsoch.github.io//2019/nushell-golang-plugin-library/
title: Nushell Plugin Library for GoLang
---

<p>I decided to take a shot at creating the start to a Nushell Plugin library in GoLang. My <a href="https://vsoch.github.io/2019/nushell-plugin-golang/" target="_blank">first plugin</a> to derive length was a filter, and if you look at the 
<a href="https://github.com/vsoch/nushell-plugin-len/blob/master/main.go" target="_blank">main.go</a>, it 
would be really hard to start with that and try to roll your own plugin. It would be
even harder to try and implement a sink plugin, which has more complicated logic
that hands your program a temporary file to read from. I also didn’t account for
named arguments.</p>

<p>I was able to re-create the same length plugin, and also two new sink plugins (<a href="https://github.com/vsoch/nu-plugin/tree/master/examples/hello" target="_blank">hello</a> and <a href="https://github.com/vsoch/nu-plugin/tree/master/examples/salad" target="_blank">salad</a>)
which are all shown below, and can be found as the library <a href="https://github.com/vsoch/nu-plugin" target="_blank">vsoch/nu-plugin</a>.</p>



<h2 id="sink-plugin">Sink Plugin</h2>

<p>For a quick example of how easy a sink plugin can be, take a look at
this example script! You should <a href="https://www.github.com/vsoch/nu-plugin" target="_blank">see the repository</a>
for details.</p>

<div class="language-go highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="k">package</span> <span class="n">main</span>
 
<span class="k">import</span> <span class="p">(</span>
	<span class="s">"fmt"</span>
	<span class="n">nu</span> <span class="s">"github.com/vsoch/nu-plugin/pkg/plugin"</span>
<span class="p">)</span>

<span class="c">// sink is your function to pass to the plugin to run</span>
<span class="k">func</span> <span class="n">sink</span><span class="p">(</span><span class="n">plugin</span> <span class="o">*</span><span class="n">nu</span><span class="o">.</span><span class="n">SinkPlugin</span><span class="p">,</span> <span class="n">params</span> <span class="k">interface</span><span class="p">{})</span> <span class="p">{</span>
	<span class="c">// a map[string]interface{} with keys, values</span>
	<span class="n">namedParams</span> <span class="o">:=</span> <span class="n">plugin</span><span class="o">.</span><span class="n">Func</span><span class="o">.</span><span class="n">GetNamedParams</span><span class="p">(</span><span class="n">params</span><span class="p">)</span>
	<span class="n">message</span> <span class="o">:=</span> <span class="s">"Hello"</span>
	<span class="k">for</span> <span class="n">name</span><span class="p">,</span> <span class="n">value</span> <span class="o">:=</span> <span class="k">range</span> <span class="n">namedParams</span> <span class="p">{</span>
		<span class="k">if</span> <span class="n">name</span> <span class="o">==</span> <span class="s">"name"</span> <span class="p">{</span>
			<span class="n">message</span> <span class="o">=</span> <span class="n">message</span> <span class="o">+</span> <span class="s">" "</span> <span class="o">+</span> <span class="n">value</span><span class="o">.</span><span class="p">(</span><span class="kt">string</span><span class="p">)</span>
		<span class="p">}</span>
	<span class="p">}</span>
	<span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="n">message</span><span class="p">)</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
	<span class="n">name</span> <span class="o">:=</span> <span class="s">"hello"</span>
	<span class="n">usage</span> <span class="o">:=</span> <span class="s">"A friendly plugin"</span>
	<span class="n">plugin</span> <span class="o">:=</span> <span class="n">nu</span><span class="o">.</span><span class="n">NewSinkPlugin</span><span class="p">(</span><span class="n">name</span><span class="p">,</span> <span class="n">usage</span><span class="p">)</span>
	<span class="n">plugin</span><span class="o">.</span><span class="n">Config</span><span class="o">.</span><span class="n">AddNamedParam</span><span class="p">(</span><span class="s">"name"</span><span class="p">,</span> <span class="s">"Optional"</span><span class="p">,</span> <span class="s">"String"</span><span class="p">)</span>
	<span class="n">plugin</span><span class="o">.</span><span class="n">Run</span><span class="p">(</span><span class="n">sink</span><span class="p">)</span>
<span class="p">}</span>
</code></pre></div></div>

<h2 id="filter-plugin">Filter Plugin</h2>

<p>A filter plugin is even easier, here is an example for length.</p>

<div class="language-go highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="k">package</span> <span class="n">main</span>
 
<span class="k">import</span> <span class="n">nu</span> <span class="s">"github.com/vsoch/nu-plugin/pkg/plugin"</span>

<span class="c">// filter is passed to the plugin to run </span>
<span class="k">func</span> <span class="n">filter</span><span class="p">(</span><span class="n">plugin</span> <span class="o">*</span><span class="n">nu</span><span class="o">.</span><span class="n">FilterPlugin</span><span class="p">,</span> <span class="n">params</span> <span class="k">interface</span><span class="p">{})</span> <span class="p">{</span>
	<span class="c">// can also be getIntPrimitive</span>
	<span class="n">value</span> <span class="o">:=</span> <span class="n">plugin</span><span class="o">.</span><span class="n">Func</span><span class="o">.</span><span class="n">GetStringPrimitive</span><span class="p">(</span><span class="n">params</span><span class="p">)</span>
	<span class="c">// Put your logic here! In this case, we want a length</span>
	<span class="n">intLength</span> <span class="o">:=</span> <span class="nb">len</span><span class="p">(</span><span class="n">value</span><span class="p">)</span>
	<span class="c">// You must also return the tag with your response</span>
	<span class="n">tag</span> <span class="o">:=</span> <span class="n">plugin</span><span class="o">.</span><span class="n">Func</span><span class="o">.</span><span class="n">GetTag</span><span class="p">(</span><span class="n">params</span><span class="p">)</span>
	<span class="c">// This can also be printStringResponse</span>
	<span class="n">plugin</span><span class="o">.</span><span class="n">Func</span><span class="o">.</span><span class="n">PrintIntResponse</span><span class="p">(</span><span class="n">intLength</span><span class="p">,</span> <span class="n">tag</span><span class="p">)</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
	<span class="n">name</span> <span class="o">:=</span> <span class="s">"len"</span>
	<span class="n">usage</span> <span class="o">:=</span> <span class="s">"Return the length of a string"</span>
	<span class="n">plugin</span> <span class="o">:=</span> <span class="n">nu</span><span class="o">.</span><span class="n">NewFilterPlugin</span><span class="p">(</span><span class="n">name</span><span class="p">,</span> <span class="n">usage</span><span class="p">)</span>
	<span class="c">// Run the filter function</span>
	<span class="n">plugin</span><span class="o">.</span><span class="n">Run</span><span class="p">(</span><span class="n">filter</span><span class="p">)</span>
<span class="p">}</span>
</code></pre></div></div>

<p>I’m hoping that this can get others started with creating plugins in GoLang,
specifically for something related to data science. Check out the <a href="https://github.com/vsoch/nu-plugin/tree/master/examples" target="_blank">examples folder</a> for inspiration, and if you make
a plugin, please contribute it to the list in the readme!</p>