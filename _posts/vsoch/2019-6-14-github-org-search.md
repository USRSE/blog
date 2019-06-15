---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-06-14 11:34:00'
layout: post
original_url: https://vsoch.github.io//2019/github-org-search/
title: GitHub Organization Search
---

<p>If you maintain or belong to a GitHub organization, it can be hard to find things.
Let’s use my example. I was developing a <a href="https://us-rse.org/community-template/" target="_blank">Community Template</a>
for the <a href="http://us-rse.org" target="_blank">US Research Software Engineers</a> 
(US-RSE) community, and specifically, I imagined some future
where a user would go to a search portal, and find a Research Software Engineer
with specific expertise, or affiliation to help. 
Given that there are a collection of community repositories (each describing
RSEs for some group) I’d want to not just search on one site, but across sites. How could I do this?</p>

<h2 id="in-a-nutshell">In a Nutshell</h2>

<p>The best way to start is to show you the result! <a href="https://us-rse.org/find-an-rse/" target="_blank">Here</a>
is the simple interface for a repository that serves a single purpose to aggregate community
data. The sites aren’t developed yet, but if you search for “Singularity” you should find
yours truly.</p>

<div style="padding:20px">
<a href="https://vsoch.github.io/assets/images/posts/usrse/find-an-rse.png"><img src="https://vsoch.github.io/assets/images/posts/usrse/find-an-rse.png" /></a>
</div>

<p>I’ll first quickly review the steps that made this possible, and then show you the details.</p>

<ol class="custom-counter">
 <li>For each repository, generate a <a href="https://gist.github.com/vsoch/9d1e6253740e67f6c9cc00e062f0d435" target="_blank">data.json</a> template to expose a subset of data to search.</li>
 <li>On a central search page, use the GitHub API to retrieve some filtered set of repository names</li>
 <li>Loop through the set and read the data.json</li>
 <li>Plug into JavaScript Search</li>
</ol>

<p>Now let’s discuss these in detail.</p>

<h2 id="1-generate-a-datajson">1. Generate a data.json</h2>

<p>Let’s step back and start with a GitHub repository. For any organization,
if you use a static site generator like <a href="https://jekyllrb.com/">Jekyll</a>, you can put
a bunch of style and markdown files on GitHub, enable GitHub pages
to deploy from the master branch, and like <strong>magic</strong> you have a website!</p>

<p>Here is the cool bit - just like you can create a template to render an html page, you
can <strong>also</strong> generate one for a text file, or a json file. Yes, 
this also means that technically, GitHub pages can serve static (GET) APIs. This is the template
that <a href="https://gist.github.com/vsoch/9d1e6253740e67f6c9cc00e062f0d435" target="_blank">we have here.</a>
It might look like a mess, but when we copy pasta the result it renders to into a json validator, we are good:</p>

<div style="padding:20px">
<a href="https://vsoch.github.io/assets/images/posts/usrse/valid-json.png"><img src="https://vsoch.github.io/assets/images/posts/usrse/valid-json.png" /></a>
</div>

<p>If you aren’t familiar with the <a href="https://jekyllrb.com/docs/liquid/">Liquid</a> templating language,
a statement like <code class="highlighter-rouge">{{{ var }}}</code> indicates a variable “var”, and a statement like <code class="highlighter-rouge">{{% if condition %}}</code>
maps to the opening of a for loop. You can read more about the templating language via the link above,
and here I’ll walk you through some pseudo code for how we parse through the site content.</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
For each page in the site that aren't flagged for exclusion:
    Filter down to some specific subset based on metadata in the front matter
        Render an entry in json that includes a key, id, title, categories, url, and content

</code></pre></div></div>

<p><a href="https://jekyllrb.com/docs/front-matter/">Front matter</a> is metadata that is defined for a post or page.
You can see examples of how this renders in the validated json image above. For the
actual names of the fields that you render, this is totally up to you! You’ll just
need to customize whatever plugs the data into search to use them (more on this later).</p>

<h2 id="2-discover-search-data">2. Discover Search Data</h2>

<p>Great! So let’s imagine we have a few sites spitting out <a href="https://us-rse.org/community-template/data.json">data like this</a>.
You’ll notice a bunch of newlines <code class="highlighter-rouge">\n</code> in the content, and this is because it’s the actual
rendering of a page, and this (along with categories and the name) is exactly what we want to search over.
How do we do that?</p>

<h3 id="github-organization-api">GitHub Organization API</h3>

<p>We first make use of <a href="https://api.github.com/orgs/usrse/repos">this API endpoint</a>
that gives us more metadata about the repositories in the usrse organization than
we know to do with. <a href="https://github.com/USRSE/find-an-rse">This repository</a> has
<a href="https://gist.github.com/vsoch/9d1e6253740e67f6c9cc00e062f0d435#file-index-html">this html template</a> that does this.
Specifically, we add the entries in the data.json to window.data.</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nb">window</span><span class="p">.</span><span class="nx">data</span> <span class="o">=</span> <span class="p">{}</span>

<span class="c1">// Here we query the GitHub API for an organization name rendered from _config.yml</span>
<span class="nx">$</span><span class="p">.</span><span class="nx">getJSON</span><span class="p">(</span><span class="s2">"https://api.github.com/orgs/{{ site.github_username }}/repos"</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">format</span><span class="p">:</span> <span class="s2">"json"</span>
<span class="p">}).</span><span class="nx">done</span><span class="p">(</span><span class="kd">function</span><span class="p">(</span><span class="nx">data</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="s2">"Found repos starting with {{ site.prefix }}:"</span><span class="p">)</span>
  <span class="nx">$</span><span class="p">.</span><span class="nx">each</span><span class="p">(</span><span class="nx">data</span><span class="p">,</span> <span class="kd">function</span><span class="p">(</span><span class="nx">key</span><span class="p">,</span> <span class="nx">value</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if</span> <span class="p">(</span><span class="nx">value</span><span class="p">.</span><span class="nx">name</span><span class="p">.</span><span class="nx">startsWith</span><span class="p">(</span><span class="s2">"{{ site.prefix }}"</span><span class="p">))</span> <span class="p">{</span>

       <span class="c1">// do something with data here</span>

     <span class="p">});</span>
    <span class="p">}</span>
  <span class="p">})</span>

</code></pre></div></div>

<p>For those of you not familiar with Jekyll, the <code class="highlighter-rouge">_config.yml</code> file can also
hold global metadata about the site. In the example above, <code class="highlighter-rouge">site.github_username</code>
would correspond to the <code class="highlighter-rouge">github_username</code> defined in the <code class="highlighter-rouge">_config.yml</code>. <code class="highlighter-rouge">site.prefix</code> 
corresponds to <code class="highlighter-rouge">prefix</code> in the same file, and we use it as a filter for repository 
names. Any repository that starts with the prefix “community-“ is a community site. Take
a <a href="https://github.com/USRSE/find-an-rse/blob/master/_config.yml">look here</a> for an example
configuration file with these variables.</p>

<h3 id="assemble-compiled-data">Assemble Compiled Data</h3>

<p>What do we do in the center of the loop? We add each entry in data.json to
the window.data. One important detail I forgot to mention is that the keys
in the data.json dictionary are namespaced. This means that they are prefixed
with a unique identifier based on the repository they are served from.
For example, <code class="highlighter-rouge">us-rse-stanford-community</code> (site identifier) + <code class="highlighter-rouge">people-vsoch</code>
(page identifier) would identify the page with my profile.  We do this
because if different sites served the same key, one would overwrite the other,
and we don’t want to do that. It’s actually up to you how you choose to implement this.
I had the serving page automatically write the identifier into the data.json
so the search page could parse it without thinking, but if you wanted more control over
the prefixes you could could also generate the prefix during the parsing itself.
Speaking of, let’s take a look at that.</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
...
      var dataurl = "{{ site.domain }}/" + value.name + '/data.json'
      $.getJSON(dataurl, {
         format: "json"
       }).done(function(pages, status) {
       if (status == "success") {
         $.each(pages, function(key, value) {
             window.data[key] = value;           
         });
        }

...

</code></pre></div></div>

<p>In the above snippet, we assemble a data url to the data.json (given the same
GitHub organization, the base URL is the same, usually “[name].github.io”, or
a custom CNAME), perform a GET, check for a successful response, and then parse through it to add to the window.data.
It’s important to point out that we can do this in the first place because
it’s not considered across origin. If you tried to do this across
different GitHub organizations, or repositories under different CNAMEs, you
would get an ugly cross origin error message. At this point we have assembled 
data across sites, woot! What do we do now?</p>

<h2 id="3-javascript-search">3. JavaScript Search</h2>

<p>I’ve taken to <a href="https://lunrjs.com/">Lunr.js</a> over the years as a really 
simple solution to providing search on a static site. You can look at 
<a href="https://github.com/USRSE/find-an-rse">the entire repository</a> to trace how it
works, and I’ll again summarize here.</p>

<p>We obviously have to add <a href="https://github.com/USRSE/find-an-rse/blob/master/assets/js/lunr.min.js">lunr.min.js</a>
to our static files, and then a <a href="https://github.com/USRSE/find-an-rse/blob/master/assets/js/search.js">search.js</a>
that is going to expect the data in window.data, and then generate results to
append to a div in the page. We start our journey in the 
<a href="https://gist.github.com/vsoch/9d1e6253740e67f6c9cc00e062f0d435#file-index-html">same template</a>
 that prepares the data. The HTML is fairly simple - we create a form to provide
a search box. Notice that when the user submits, it performs a GET to itself, and 
sends the query term to the browser as a <a href="https://en.ryte.com/wiki/GET_Parameter">GET parameter</a>.</p>

<div class="language-html highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nt">&lt;form</span> <span class="na">action=</span><span class="s">"{{ site.baseurl }}/"</span> <span class="na">method=</span><span class="s">"get"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"search"</span> <span class="na">name=</span><span class="s">"q"</span> <span class="na">id=</span><span class="s">"search-input"</span> <span class="na">placeholder=</span><span class="s">"Find an RSE?"</span> <span class="na">style=</span><span class="s">"margin-top:5px"</span> <span class="na">autofocus</span><span class="nt">&gt;</span>
  <span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">value=</span><span class="s">"Search"</span> <span class="na">style=</span><span class="s">"display: none;"</span><span class="nt">&gt;</span>
<span class="nt">&lt;/form&gt;</span>

</code></pre></div></div>

<p>Note that the input variable name for the search result is “q.” This is what is going
to be sent to the page in the browser as the search query, e.g <code class="highlighter-rouge">{{{ site.baseurl }}}/?q=query</code>.
Then we have a “search-process” div that the search.js is going to update with results,
and a “search-query” span that our term will be added to.</p>

<div class="language-html highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nt">&lt;p&gt;&lt;span</span> <span class="na">id=</span><span class="s">"search-process"</span><span class="nt">&gt;</span>Loading<span class="nt">&lt;/span&gt;</span> results 
   <span class="nt">&lt;span</span> <span class="na">id=</span><span class="s">"search-query-container"</span> <span class="na">style=</span><span class="s">"display: none;"</span><span class="nt">&gt;</span>for "
      <span class="nt">&lt;strong</span> <span class="na">id=</span><span class="s">"search-query"</span><span class="nt">&gt;&lt;/strong&gt;</span>"
   <span class="nt">&lt;/span&gt;&lt;/p&gt;</span>
<span class="nt">&lt;ul</span> <span class="na">id=</span><span class="s">"search-results"</span><span class="nt">&gt;&lt;/ul&gt;</span>

</code></pre></div></div>

<p>And finally, we add the javascript files to the end of the page, triggering the entire process.</p>

<div class="language-html highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"{{ site.baseurl }}/assets/js/lunr.min.js"</span><span class="nt">&gt;&lt;/script&gt;</span>
<span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"{{ site.baseurl }}/assets/vendor/jquery/jquery.min.js"</span> <span class="nt">&gt;&lt;/script&gt;</span>

</code></pre></div></div>

<p>What happens when search.js runs? Let’s now jump into search.js. First, lunr.js is instantiated at window.index.</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
	<span class="nb">window</span><span class="p">.</span><span class="nx">index</span> <span class="o">=</span> <span class="nx">lunr</span><span class="p">(</span><span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
		<span class="k">this</span><span class="p">.</span><span class="nx">field</span><span class="p">(</span><span class="s2">"id"</span><span class="p">);</span>
		<span class="k">this</span><span class="p">.</span><span class="nx">field</span><span class="p">(</span><span class="s2">"title"</span><span class="p">,</span> <span class="p">{</span><span class="na">boost</span><span class="p">:</span> <span class="mi">10</span><span class="p">});</span>
		<span class="k">this</span><span class="p">.</span><span class="nx">field</span><span class="p">(</span><span class="s2">"categories"</span><span class="p">);</span>
		<span class="k">this</span><span class="p">.</span><span class="nx">field</span><span class="p">(</span><span class="s2">"url"</span><span class="p">);</span>
		<span class="k">this</span><span class="p">.</span><span class="nx">field</span><span class="p">(</span><span class="s2">"content"</span><span class="p">);</span>
	<span class="p">});</span>

</code></pre></div></div>

<p>This is likely where we set the fields from the data that we want to search,
along with other variables for each. Take a look at the <a href="https://lunrjs.com/docs/index.html">documentation</a>
to see much simpler examples. It’s a fairly awesome little library.</p>

<p>We then grab the query from the URL, the <code class="highlighter-rouge">?q=mysearchterm</code>. If it doesn’t exist, 
we default to the empty string. We also grab the “search-query-container” and
the “search-query” div, and we update both:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="kd">var</span> <span class="nx">query</span> <span class="o">=</span> <span class="nb">decodeURIComponent</span><span class="p">((</span><span class="nx">getQueryVariable</span><span class="p">(</span><span class="s2">"q"</span><span class="p">)</span> <span class="o">||</span> <span class="s2">""</span><span class="p">).</span><span class="nx">replace</span><span class="p">(</span><span class="sr">/</span><span class="se">\+</span><span class="sr">/g</span><span class="p">,</span> <span class="s2">"%20"</span><span class="p">)),</span>
		<span class="nx">searchQueryContainerEl</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">getElementById</span><span class="p">(</span><span class="s2">"search-query-container"</span><span class="p">),</span>
		<span class="nx">searchQueryEl</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">getElementById</span><span class="p">(</span><span class="s2">"search-query"</span><span class="p">);</span>

<span class="nx">searchQueryEl</span><span class="p">.</span><span class="nx">innerText</span> <span class="o">=</span> <span class="nx">query</span><span class="p">;</span>
<span class="k">if</span> <span class="p">(</span><span class="nx">query</span> <span class="o">!=</span> <span class="s2">""</span><span class="p">){</span>
    <span class="nx">searchQueryContainerEl</span><span class="p">.</span><span class="nx">style</span><span class="p">.</span><span class="nx">display</span> <span class="o">=</span> <span class="s2">"inline"</span><span class="p">;</span>
<span class="p">}</span>

</code></pre></div></div>

<p>Finally, we give the window.data (the json entries, one per external page)
to the lunr instantiation we created (window.index):</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="k">for</span> <span class="p">(</span><span class="kd">var</span> <span class="nx">key</span> <span class="k">in</span> <span class="nb">window</span><span class="p">.</span><span class="nx">data</span><span class="p">)</span> <span class="p">{</span>
	<span class="nb">window</span><span class="p">.</span><span class="nx">index</span><span class="p">.</span><span class="nx">add</span><span class="p">(</span><span class="nb">window</span><span class="p">.</span><span class="nx">data</span><span class="p">[</span><span class="nx">key</span><span class="p">]);</span>
<span class="p">}</span>

</code></pre></div></div>

<p>And we call the function “displaySearchResults” that is going to run the search,
and then render the matches to the page.</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nx">displaySearchResults</span><span class="p">(</span><span class="nb">window</span><span class="p">.</span><span class="nx">index</span><span class="p">.</span><span class="nx">search</span><span class="p">(</span><span class="nx">query</span><span class="p">),</span> <span class="nx">query</span><span class="p">);</span>
</code></pre></div></div>

<p>It goes without saying that for the above, we search across fields, and for each match, 
since we have a url field, we render that as a link that the user can click.
The result is <a href="https://us-rse.org/find-an-rse/">this page</a>.</p>

<h2 id="summarize">Summarize!</h2>

<p>Wasn’t this fun? Javascript is a bit hairy, and I certainly don’t claim to
follow best practices. For example, a lot of (actually proficient) JavaScript
developers would not be happy with my use of the window for variables,
or even using JQuery to begin with. Regardless, the above is a nice example for how easy it is to
break a fugly thing (a combination of jekyll, templates, and scripts) into
a story that can be understood.</p>

<p>And guess what, I showed you the more complicated version first! You can
totally skip the GitHub API, and parsing data.json files entirely if you
just want to add search to a single respository’s site. For example, in the process of
designing this, I added a <a href="https://us-rse.org/search/">simple search page</a> to us-rse.org.
And if you look at the <a href="https://github.com/USRSE/usrse.github.io/blob/master/pages/search.html">template</a> 
that renders it, you’ll notice we are writing the data object directly into the page. 
This is hugely easier.</p>

<h2 id="why-is-this-important">Why is this important?</h2>

<p>I suspect that most GitHub organizations, or even single repositories, don’t
provide an easy means to find things. Whether you are serving content that
is documentation for your software, or information about people that might
provide support, providing this kind of search is essential for the usability
of your resource.</p>

<p>Is there only one way to fulfill a Meeseeks? Of course not!
I encourage you to explore other ways to implement static search, one example
being <a href="http://www.tipue.com/search/">tipuesearch</a> that I use on the site that
you are reading right now. And of course, if you have questions or would
like help, please reach out.</p>