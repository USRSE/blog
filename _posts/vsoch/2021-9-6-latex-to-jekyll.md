---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2021-09-06 19:30:00'
layout: post
original_url: https://vsoch.github.io//2021/latex-to-jekyll/
title: LaTex To Jekyll
---

<p>I recently wrote a massive paper. It might be some rare disease that I have, but
I’m able to produce texty content at a fairly rapid pace that covers many ideas.
In most cases this is a good thing, because it provides a structure to start adding
meat, and pruning away some, but in some cases, people will look and say “Ohno this is just
too long!” This happened to me recently, as I had 27 pages of thinking (not including references)
and more of a desire to share it in a digestable way than to figure out how and what to cut.
Note that we will eventually have some much shorter version, but to start I am more
excited to just share all the ideas and get people working together.</p>

<p>So what to do? I decided I wanted to make a website. To cut to the chase, I ultimately wanted to
generate a website <em>and</em> LaTeX paper (pdf) from the same content, and I wanted it
to be automated and easy. So this small post will walk through how I started with
a LaTex thing and wound up with a fully rendered and organized site.
Now I don’t have a totally automated way to generate everything because your site
will be organized differently from mine, but I’ll walk through the steps that you too
can take!</p>

<h2 id="step-0-have-a-jekyll-template">Step 0: Have a Jekyll Template</h2>

<p>You’ll want to start with a Jekyll template, and follow the basic steps 
for <a href="https://github.com/inukshuk/jekyll-scholar" target="_blank">jekyll-scholar</a> so that you
have a collection for paper details, “paper-details” and an entire section in
your “_config.yaml” that describes your citation files. A good example of me
doing this is in an old project, <a href="https://github.com/vsoch/notes-jekyll/" target="blank">notes jekyll</a>.
If you have any questions, please don’t hesitate to open an issue.</p>

<h2 id="step-1-write-the-paper">Step 1: Write the Paper</h2>

<p>Step one is of course obvious - you should first write your paper! However,
the trick to making this easy to convert to modular parts of a website will be
how you design the LaTeX. Let’s say you normally have some “main.tex.” Instead of this one
file, you actually want to “input” sections so it’s modular. So, for example, a snippet
of my main.tex might look like this:</p>

<div class="language-latex highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="k">\section</span><span class="p">{</span>Abstract<span class="p">}</span>
<span class="k">\input</span><span class="p">{</span>abstract<span class="p">}</span>

<span class="k">\section</span><span class="p">{</span>Introduction<span class="p">}</span>
<span class="k">\input</span><span class="p">{</span>introduction<span class="p">}</span>

<span class="k">\subsection</span><span class="p">{</span>What is DevOps?<span class="p">}</span>
<span class="k">\input</span><span class="p">{</span>what-is-devops<span class="p">}</span>
<span class="k">\subsection</span><span class="p">{</span>What is RSE-ops?<span class="p">}</span>
<span class="k">\input</span><span class="p">{</span>what-is-rseops<span class="p">}</span>
</code></pre></div></div>

<p>In the above, you’ll see that instead of just writing sections in the introduction
and the abstract, they are each contained in their own files, “abstract.tex,” “introduction.tex,”
“what-is-rseops.tex,” and “what-is-devops.tex.” These files will be plopped in a “src”
folder of our repository, and will be easy to render into a standard pdf, but <em>also</em>
will be easy to run a few steps to generate html with citations for the site. So the first step is download
those files (get them out of Overleaf or whatever!) and put them in a “src” folder
alongside your jekyll site. Mine looks like this:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>tree src/
src/
├── abstract.tex
...
├── rseops.bib
├── introduction.tex
├── what-is-devops.tex
└── what-is-rseops.tex
</code></pre></div></div>

<p>Do you notice the bib (bibliography) file? Notably, that is going to need to go
in this folder <em>and</em> also be present in “_bibliography” for the jekyll plugin.
So what I did is to create a link:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span><span class="nb">ln </span>src/rseops.bib _bibliography/rseops.bib
</code></pre></div></div>

<h2 id="step-2-prepare-structure">Step 2: Prepare structure</h2>

<p>I decided that, for some number of my *.tex files, I would want to render them
into pages. What would be an easy way to do that? With <a href="https://jekyllrb.com/docs/includes/" target="_blank">Jekyll includes</a>
of course! I tested both rendering markdown and html, and html includes work much better.
To prepare the structure of my site, I created a “_includes/paper” folder</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span><span class="nb">mkdir</span> <span class="nt">-p</span> _includes/paper
</code></pre></div></div>

<p>and then wrote a simple text file list of the names of those I wanted to include, includes.txt,
an example shown below:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
abstract
accessibility
comparison
containers
continuous-deployment
continuous-integration
culture
dependencies
distribution
follow-devops
introduction
</code></pre></div></div>

<p>These are just prefixes for what will eventually go from LaTeX (tex) to html.
E.g., “abstract” will render an include for “abstract.tex.”</p>

<h2 id="step-2-prepare-a-container">Step 2: Prepare a container</h2>

<p>If you are like me, you’ll want a container that can easily run pandoc to
convert from LaTex to markdown (or html, actually). Here is the container I threw
together:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
FROM pandoc/latex

# docker build -t latex2md .
# docker run -it -v $PWD:/code latex2md

LABEL maintainer "@vsoch"
ENV DEBIAN_FRONTEND noninteractive

RUN apk update &amp;&amp; apk add python3 &amp;&amp; ln /usr/bin/python3 /usr/bin/python
ADD . /code
RUN chmod u+x /code/entrypoint.sh
WORKDIR /code

ENTRYPOINT ["/bin/sh", "/code/entrypoint.sh"]
</code></pre></div></div>

<p>You’ll notice the commands to build and run are on the top lines after the FROM.
This is something I usually do because Vanessa of the future never remembers what
is going on :)</p>

<h3 id="makefile">Makefile</h3>

<p>The build and run commands are shown above, but I also created a Makefile to “make”
my life easy, lol.</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
.PHONY: all build run
PROJECT_ROOT = $(shell pwd)

all: build
	docker run -it -v ${PROJECT_ROOT}:/code latex2md

build:
	docker build -t latex2md .
	
run:
	bundle exec jekyll serve
</code></pre></div></div>

<p>This meant that “make build” would build my container, “make” would render LaTeX
into the includes, and then “make run” would run my server to preview. Easy peezy!
But I haven’t shown you what’s in <code class="language-plaintext highlighter-rouge">entrypoint.sh</code>, which is arguably the logic of the
operation! Let’s do that.</p>

<h3 id="entrypoint">Entrypoint</h3>

<p>Ok, this is a fairly simple bash script. We are going to, from the “src” directory, 
loop through each entry in the includes.txt file, run pandoc to convert from LaTex
to html, and output in the _includes/paper folder, and then run a custom Python
script that is going to finish some extra parsing of our entry. At the end we will
also render all the files into a single PDF that we can also serve from the site.</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
#!/bin/bash

cd src
for include in $(cat ../includes.txt); do
    printf "Parsing $include\n"
    pandoc $include.tex -o ../_includes/paper/$include.html --bibliography rseops.bib 
    python ../update-citations.py ../_includes/paper/$include.html
done

# Also render the updated PDF
pandoc -s main.tex -o "rse-ops.pdf" --pdf-engine=xelatex  --bibliography rseops.bib --citeproc

# Fix permissions
cd ..
chmod 0777 _includes/*
echo "Files generated:"
ls _includes/paper
</code></pre></div></div>

<h3 id="custom-python-script">Custom Python Script</h3>

<p>I chose Python because it was quick, but  you can obviously use another language.
For this final step, our include files are almost perfect, but we have to replace
the (sort of useless) citation span elements with something that 
<a href="https://github.com/inukshuk/jekyll-scholar" target="_blank">jekyll-scholar</a>
can understand. This means adding the bibliography to the bottom, which looks like this:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>{% bibliography --cited %}
</code></pre></div></div>

<p>And then replacing the useless span elements that have ids of citations with
something that looks like:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>{% cite github %}
</code></pre></div></div>

<p>Where “github” might be the name of a reference in our bibliography file.</p>

<h2 id="step-3-generate-and-customize">Step 3: Generate and Customize!</h2>

<p>Once you have all that, you can build the container, generate the includes,
and then start working on your site! Basically, if you generate “_includes/paper/containers.html”
you can insert this (which includes pretty linked references) anywhere in your markdown
and it will magically just work. E.g.,</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>{% include paper/containers.html %}
</code></pre></div></div>

<p>There are a few more details like customizing
a “_layout/bibitem-template.html,” “_layouts/details.html” and “_bibliography/my-ieee.cls”
to get the references looking to your preference, and you can see an example in
another of my old notes sites, <a href="https://github.com/buildsi/build-notes" target="_blank">here</a>
for that. The resulting site will render pages (styling is up to you!) that have content from the paper and references,
directly in the site:</p>

<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/rse-ops/portability.png" />
</div>

<p>And then the exact same content still renders into a beautiful PDF (this is only partially shown,
as I am not done the site or work yet).</p>

<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/rse-ops/paper.png" />
</div>

<p>I’ll note that another step I took that was custom for my site (and you might
not want or need) was to have some content in “_data/” (yaml files in my case, and you can read more about it
<a href="https://jekyllrb.com/docs/datafiles/" target="_blank">here</a> that also would make it easy to loop over these various
sections.</p>

<h2 id="step-4-automate">Step 4: Automate!</h2>

<p>All of this would be totally awful if we didn’t automate it. We want to be able
to have any pull request be checked that everything runs and builds, and then
a merge to main deploys to GitHub pages. Since we are using a custom plugin,
this means that we will need to build the site on our own. And then it’s easiest to
push to the gh-pages branch and deploy from there. So here is the workflow I came up with for that:</p>



<p>Notice that we are building the same container to generate any updated content,
building with a jekyll container, and force pushing to the gh-pages branch on merge/push
to main. So this means all you’d need to do is add the workflow, and make sure to turn
on your gh-pages branch to be linked to GitHub pages!</p>

<h2 id="step-5-review">Step 5: Review</h2>

<p>In summary, with this setup you can:</p>

<ol class="custom-counter">
  <li> Edit the same LaTeX files and have them render into your PDF and website</li>
  <li> Keep everything safe in version control</li>
  <li> Have any contributions peer reviewed via GitHub, and everything updated on merge</li>
  <li> Pretty print references in HTML and the PDF</li>
</ol>

<p>I thought this was pretty cool, and I’m looking forward to seeing what folks can come up
with. If you make something cool, please share! ❤️</p>