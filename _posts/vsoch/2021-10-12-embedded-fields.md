---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2021-10-12 21:30:00'
layout: post
original_url: https://vsoch.github.io//2021/embedded-fields/
title: TIL- Embedded Fields in Structs for Go
---

<p>Here is a quick “Today I learned” about how to mimic inheritance in Go. For example, in 
Python if we have a class, we can easily inherit attributes and functions from it
for some subclass. Here is a parent class “Fruit” and a child class “Avocado” that inherits the
attributes name and color, and the function IsReady.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="c1">#!/usr/bin/env python3
</span>
<span class="k">class</span> <span class="nc">Fruit</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">name</span><span class="p">,</span> <span class="n">color</span><span class="p">):</span>
        <span class="bp">self</span><span class="p">.</span><span class="n">name</span> <span class="o">=</span> <span class="n">name</span>
        <span class="bp">self</span><span class="p">.</span><span class="n">color</span> <span class="o">=</span> <span class="n">color</span>

    <span class="k">def</span> <span class="nf">Announce</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="k">print</span><span class="p">(</span><span class="sa">f</span><span class="s">"I am </span><span class="si">{</span><span class="bp">self</span><span class="p">.</span><span class="n">color</span><span class="si">}</span><span class="s">, and I am </span><span class="si">{</span><span class="bp">self</span><span class="p">.</span><span class="n">name</span><span class="si">}</span><span class="s">"</span><span class="p">)</span>


<span class="k">class</span> <span class="nc">Avocado</span><span class="p">(</span><span class="n">Fruit</span><span class="p">):</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">name</span><span class="p">,</span> <span class="n">color</span><span class="p">,</span> <span class="n">is_ripe</span><span class="p">):</span>
        <span class="nb">super</span><span class="p">().</span><span class="n">__init__</span><span class="p">(</span><span class="n">name</span><span class="p">,</span> <span class="n">color</span><span class="p">)</span>
        <span class="bp">self</span><span class="p">.</span><span class="n">is_ripe</span> <span class="o">=</span> <span class="n">is_ripe</span>

    <span class="k">def</span> <span class="nf">IsReady</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="k">if</span> <span class="bp">self</span><span class="p">.</span><span class="n">is_ripe</span><span class="p">:</span>
            <span class="k">print</span><span class="p">(</span><span class="sa">f</span><span class="s">"I am </span><span class="si">{</span><span class="bp">self</span><span class="p">.</span><span class="n">color</span><span class="si">}</span><span class="s">, and I am </span><span class="si">{</span><span class="bp">self</span><span class="p">.</span><span class="n">name</span><span class="si">}</span><span class="s">, and I am ripe!"</span><span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="k">print</span><span class="p">(</span><span class="sa">f</span><span class="s">"I am </span><span class="si">{</span><span class="bp">self</span><span class="p">.</span><span class="n">color</span><span class="si">}</span><span class="s">, and I am </span><span class="si">{</span><span class="bp">self</span><span class="p">.</span><span class="n">name</span><span class="si">}</span><span class="s">, and I am NOT ripe!"</span><span class="p">)</span>    

<span class="k">def</span> <span class="nf">main</span><span class="p">():</span>

    <span class="n">avocado</span> <span class="o">=</span> <span class="n">Avocado</span><span class="p">(</span><span class="n">name</span> <span class="o">=</span> <span class="s">"Harry the avocado"</span><span class="p">,</span> <span class="n">color</span> <span class="o">=</span> <span class="s">"green"</span><span class="p">,</span> <span class="n">is_ripe</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
    <span class="n">avocado</span><span class="p">.</span><span class="n">Announce</span><span class="p">()</span>
    <span class="c1"># I am green, and I am Harry the avocado
</span>
    <span class="n">avocado</span><span class="p">.</span><span class="n">IsReady</span><span class="p">()</span>
    <span class="c1"># I am green, and I am Harry the avocado, and I am ripe! 
</span>   
<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="s">"__main__"</span><span class="p">:</span>
    <span class="n">main</span><span class="p">()</span>
</code></pre></div></div>

<p>Notice the constructor in main - we just hand the name, color, and the variable for if it’s ripe
to the Avocado class, and it works! We inherit functions and attributes from the parent class.
But what about Go? I wanted an easy way to do this in Go, because otherwise creating structures with
shared attributes or functionality felt very redundant. So here is the first thing I tried:</p>

<div class="language-go highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">package</span> <span class="n">main</span>

<span class="k">import</span> <span class="p">(</span>
	<span class="s">"fmt"</span>
<span class="p">)</span>

<span class="k">type</span> <span class="n">Fruit</span> <span class="k">struct</span> <span class="p">{</span>
	<span class="n">Name</span> <span class="kt">string</span>
	<span class="n">Color</span>	<span class="kt">string</span>
<span class="p">}</span>

<span class="k">func</span> <span class="p">(</span><span class="n">f</span><span class="o">*</span> <span class="n">Fruit</span><span class="p">)</span> <span class="n">Announce</span><span class="p">()</span> <span class="p">{</span>
	<span class="n">fmt</span><span class="o">.</span><span class="n">Printf</span><span class="p">(</span><span class="s">"I am %s, and I am %s</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="n">f</span><span class="o">.</span><span class="n">Color</span><span class="p">,</span> <span class="n">f</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>
<span class="p">}</span>

<span class="k">type</span> <span class="n">Avocado</span> <span class="k">struct</span> <span class="p">{</span>
	<span class="n">Fruit</span>
	<span class="n">IsRipe</span> <span class="kt">bool</span>
<span class="p">}</span>

<span class="k">func</span> <span class="p">(</span><span class="n">a</span> <span class="o">*</span> <span class="n">Avocado</span><span class="p">)</span> <span class="n">IsReady</span><span class="p">()</span> <span class="p">{</span>
	<span class="k">if</span> <span class="n">a</span><span class="o">.</span><span class="n">IsRipe</span> <span class="p">{</span>
		<span class="n">fmt</span><span class="o">.</span><span class="n">Printf</span><span class="p">(</span><span class="s">"I am %s, and I am %s, and I am ripe!</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="n">a</span><span class="o">.</span><span class="n">Color</span><span class="p">,</span> <span class="n">a</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>
	<span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
		<span class="n">fmt</span><span class="o">.</span><span class="n">Printf</span><span class="p">(</span><span class="s">"I am %s, and I am %s, and I am NOT ripe!</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="n">a</span><span class="o">.</span><span class="n">Color</span><span class="p">,</span> <span class="n">a</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>
	<span class="p">}</span>

<span class="p">}</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
	<span class="c">// avocado := Avocado{Name: "Harry the avocado", Color: "green", IsRipe: true}	</span>
	<span class="n">avocado</span><span class="o">.</span><span class="n">Announce</span><span class="p">()</span>
	<span class="n">avocado</span><span class="o">.</span><span class="n">IsReady</span><span class="p">()</span>
<span class="p">}</span>
</code></pre></div></div>

<p>And that totally didn’t work! I saw this:</p>

<div class="language-go highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">./</span><span class="n">main</span><span class="o">.</span><span class="k">go</span><span class="o">:</span><span class="m">19</span><span class="o">:</span><span class="m">21</span><span class="o">:</span> <span class="n">cannot</span> <span class="n">use</span> <span class="n">promoted</span> <span class="n">field</span> <span class="n">Fruit</span><span class="o">.</span><span class="n">Name</span> <span class="n">in</span> <span class="k">struct</span> <span class="n">literal</span> <span class="n">of</span> <span class="k">type</span> <span class="n">Avocado</span>
<span class="o">./</span><span class="n">main</span><span class="o">.</span><span class="k">go</span><span class="o">:</span><span class="m">19</span><span class="o">:</span><span class="m">48</span><span class="o">:</span> <span class="n">cannot</span> <span class="n">use</span> <span class="n">promoted</span> <span class="n">field</span> <span class="n">Fruit</span><span class="o">.</span><span class="n">Color</span> <span class="n">in</span> <span class="k">struct</span> <span class="n">literal</span> <span class="n">of</span> <span class="k">type</span> <span class="n">Avocado</span>
</code></pre></div></div>

<p>Turns out, I was close. I actually needed to provide Fruit in the constructor for Avocado, like this:</p>

<div class="language-go highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c">// Instead, pass the "base" type to the underlying type</span>
<span class="n">avocado</span> <span class="o">:=</span> <span class="n">Avocado</span><span class="p">{</span><span class="n">Fruit</span><span class="o">:</span> <span class="n">Fruit</span><span class="p">{</span><span class="n">Name</span><span class="o">:</span> <span class="s">"Harry the avocado"</span><span class="p">,</span> <span class="n">Color</span><span class="o">:</span> <span class="s">"green"</span><span class="p">},</span> <span class="n">IsRipe</span><span class="o">:</span> <span class="no">true</span><span class="p">}</span>
</code></pre></div></div>

<p>Is this kind of funky? Yeah, and <a href="https://github.com/golang/go/issues/9859" target="_blank">others think so too</a>.
But I’m really grateful for the functionality because I can create a bunch of different struct types that have most in
common, but maybe don’t share a few things. Could there be other issues that arise? Maybe. I haven’t hit them yet. :)
There’s a nice article <a href="https://go101.org/article/type-embedding.html" target="_blank">here</a> that I’m perusing to learn more.
<a href="https://gist.github.com/vsoch/dd34ac96dc463c0a2f18c53aea67cf37" target="_blank">Here is the entire set of files</a> in a gist if you want to play around.</p>