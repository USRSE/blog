---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2021-10-20 21:30:00'
layout: post
original_url: https://vsoch.github.io//2021/jekyll-next-navigation/
title: Automatic Jekyll Navigation
---

<p>One thing I’m quite lazy about is wanting to update navigation. There are several approaches I’ve taken in 
the past to handle this, the most labor intensive being maintaining several (possibly related) navigation
menus via YAML files or directly in Jekyll’s <code class="language-plaintext highlighter-rouge">_config.yaml</code>. For individual files that aren’t posts
or part of a logic collection, this often means manually defining the <code class="language-plaintext highlighter-rouge">permalink:</code> attribute, which can
also get tiring.</p>

<p>When I designed <a href="https://vsoch.github.io/docsy-jekyll/" target="_blank">Docsy Jekyll</a> (and note that by “design” I mean write the logic for the Jekyll site, the design itself is done by the authors
of  <a href="https://www.docsy.dev/" target="_blank">the original Docsy</a>) I did an intermediate approach -
I had a few upper levels of links “hard coded” in the sidebar (via YAML) and then for any content in <code class="language-plaintext highlighter-rouge">_docs</code>, by
way of being a collection, they would general URLs that logically corresponded to their organization. For example:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>_docs/
├── ci-cd
├── compilers
├── containers
│   ├── docker.md       (- /docs/containers/docker
│   ├── docker
│   │   └── getting-started.md  (- /docs/containers/docker/getting-started
│   ├── subpage         (- /docs/containers/subpage
│   └── singularity     (- /docs/containers/singularity
</code></pre></div></div>

<p>This was annoying, of course, because instead of doing this to define an “index.html” equivalent:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>_docs/
├── ci-cd
├── compilers
├── containers
│   ├── docker
│   │   ├── index.md       (?? /docs/containers/docker/
│   │   └── getting-started.md  (- /docs/containers/docker/getting-started
</code></pre></div></div>

<p>I had to do what I showed above - a docker.md alongside the docker folder. But I could actually use a <code class="language-plaintext highlighter-rouge">permalink</code> to get around that if I wanted.
But I digress! With the organization above, I had a fairly auto-generating site, and although I had a few “summary” or top level pages to loop through all links, I still largely had to link from page to page. That was tiring.</p>

<h2 id="automatic-generation-of-direct-child-pages">Automatic Generation of Direct Child Pages</h2>

<p>It occurred to me that given the structure of the url, I could easily derive pages that were children of other pages,
and then at the bottom of each page, automatically provide navigation to any page on the next level. For example, if I have these pages:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
/docs/
/docs/containers
/docs/containers/singularity
/docs/containers/docker
/docs/containers/docker/tutorial
/docs/ci
</code></pre></div></div>

<p>When I’m at the root, <code class="language-plaintext highlighter-rouge">docs</code>, I’d only want to see links to</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
/docs/containers
/docs/ci
</code></pre></div></div>
<p>When I navigate then to <code class="language-plaintext highlighter-rouge">/docs/containers</code> I’d only want to see:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
/docs/containers/singularity
/docs/containers/docker
</code></pre></div></div>

<p>You get it. This seems obvious in retrospect, but I don’t see if often on Jekyll sites!
So the easy logic is that when we are looping through pages in docs, we do a comparison
to a potential “next page” and the current page. We check:</p>

<ol class="custom-counter">
  <li>Is the contender page url included in the current url?</li>
  <li>Is the nesting of the contender page the current nesting + 1?</li>
</ol>

<p>For the first point, you can easily just check if the contender page url contains the current page url.
E.g., “/docs/containers/docker” contains “/docs/containers” and therefore should be included.
And for the second, we basically do a bunch of splitting and counting of slashes.
In case this is useful for anyone else wanting similar functionality, here is what I came up with.</p>

<div class="language-html highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nt">&lt;h3&gt;</span>Read Next<span class="nt">&lt;/h3&gt;</span>
{% assign page_url = page.url | replace: "index", "" | split: "/" | join: "/" %}

<span class="c">&lt;!-- We compare lengths - the next level should be current +1.--&gt;</span>
{% assign current_level = page_url | split: "/" | size %}
{% assign next_level = current_level | plus: 1 %}

<span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"section-index"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;hr</span> <span class="na">class=</span><span class="s">"panel-line"</span><span class="nt">&gt;&lt;div</span> <span class="na">class=</span><span class="s">"card-columns"</span><span class="nt">&gt;</span>{% for doc in site.docs %}
   {% assign doc_url = doc.url | replace: "index", "" %}
    {% unless doc.url == page.url %}
    {% assign comparison_count = doc_url | split: "/" | size %}
    {% if next_level == comparison_count %}{% if doc_url contains page_url %}
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card next-card"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;h5&gt;&lt;a</span> <span class="na">href=</span><span class="s">"{{ site.baseurl }}{{ doc.url }}"</span><span class="nt">&gt;</span>{{ doc.title }}<span class="nt">&lt;/a&gt;&lt;/h5&gt;</span>
      <span class="nt">&lt;p&gt;</span>{{ doc.description }}<span class="nt">&lt;/p&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
    {% endif %}{% endif %}{% endunless %}{% endfor %}<span class="nt">&lt;br&gt;</span>
<span class="nt">&lt;/div&gt;</span>
</code></pre></div></div>

<p>Mind you that how you render your pages is up to you - I have little cards that include titles and descriptions.
The result looks something like this:</p>

<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/rseng/example-1.png" />
</div>

<p>And the example we talked about:</p>

<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/rseng/example-2.png" />
</div>

<p>Just a tiny thing - but if this is useful to you, I’d like to help! Happy Jekyll-ing!</p>