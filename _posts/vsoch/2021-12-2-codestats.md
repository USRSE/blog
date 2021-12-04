---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2021-12-02 10:30:00'
layout: post
original_url: https://vsoch.github.io//2021/codestats/
title: Codestats
---

<p>I love any opportunity to create a project in Go, so when I saw the <a href="https://github.com/bloodorangeio/oci-stats" target="_blank">bloodorangeio/oci-stats</a> tool, I was inspired! This project uses bash scripts and a Makefile to generate basic health stats for a repository, or basically a record of health files like a Contributing markdown or particular continuous integration recipe. If you have a lot of repos you want to quickly check for common community files presence or absence, it can be a useful tool. I liked the idea and wanted to do something similar,
but provide a library in Go that is generalizable to different repository metrics, and that could easily be piped into an interactive interface.
Note that this isn’t a hugely innovative idea, because many people have made these kinds of tools before. This is just a bit of dinosaur fun!</p>

<div style="padding: 20px;">
  <img src="https://raw.githubusercontent.com/vsoch/codestats/main/docs/codestats-entry.png" />
</div>

<p>If you don’t care about the process, skip it and just ⭐️ <a href="https://vsoch.github.io/codestats/" target="_blank">check out the interface</a> ⭐️.
It’s fairly simple with Bootstrap, Jquery Datatables, and a custom font, and I imagine could be styled much better.</p>

<h2 id="codestats">Codestats</h2>

<p>Thus, <a href="https://github.com/vsoch/codestats" target="_blank">vsoch/codestats</a> was born! I always have little pet projects to work
on in evenings and weekends, and this became my interest for the last week or so. For codestats, I wanted to accomplish the following:</p>

<ol class="custom-counter">
  <li>Provide a repository or organization to parse.</li>
  <li>Pair that with a set of metrics, where each metric is some action you can run on the root of a repository</li>
  <li>Extract either single repos or an entire org set to a json structure that renders into an interface.</li>
</ol>

<p>Given that the library provides a pre-determined set of metrics, the user should have the choice to filter down to a custom set, or
(if there is need to add a new metric) to <a href="https://github.com/vsoch/codestats/issues" target="_blank">open an issue</a> and let me know!
Right now the tests are written into the code, but I could totally see adding an ability to specify a custom test in the optional config yaml
for the library.</p>

<h3 id="how-does-it-work">How does it work?</h3>

<p>Well, first you want to clone and build the library. You’ll need Go on your path!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>git clone https://github.com/vsoch/codestats
<span class="nv">$ </span><span class="nb">cd </span>codestats
<span class="nv">$ </span>make
</code></pre></div></div>

<p>This will place an executable, <code class="language-plaintext highlighter-rouge">codestats</code> that you can interact with. Then, here are examples
for how to run extractions. When we add –outfile that will save to the json file you specify, otherwise
we print to the console.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="c"># Extract stats for the repository buildsi/build-abi-containers</span>
<span class="nv">$ </span>./codestats repo buildsi/build-abi-containers

<span class="c"># Do the same, but save to a file</span>
<span class="nv">$ </span>./codestats repo buildsi/build-abi-containers <span class="nt">--outfile</span> examples/repo.json


<span class="c"># Do the same, but pretty print to the screen (and run with go directly)</span>
<span class="nv">$ </span>go run main.go repo buildsi/build-abi-containers <span class="nt">--pretty</span>

<span class="c"># Extract stats for an organization on GitHub</span>
<span class="nv">$ </span>./codestats org buildsi

build-notes
build-si-modeling
Smeagle
build-abi-tests
build-abi-containers
build-sandbox
...

<span class="c"># Only include those that match a pattern</span>
<span class="nv">$ </span>./codestats org buildsi <span class="nt">--pattern</span> build-abi-containers <span class="nt">--pretty</span>


<span class="c"># or again, save to output file:</span>

<span class="nv">$ </span>./codestats org buildsi <span class="nt">--outfile</span> examples/org.json
</code></pre></div></div>

<p>What if you want a custom set? You can generate a yaml file with the identifiers
that you want:</p>

<div class="language-yaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1"># List of stats to include</span>
<span class="na">stats</span><span class="pi">:</span>
<span class="pi">-</span> <span class="s">has-codeowners</span>
<span class="pi">-</span> <span class="s">has-maintainers</span>
<span class="pi">-</span> <span class="s">has-github-actions</span>
<span class="pi">-</span> <span class="s">has-circle</span>
<span class="pi">-</span> <span class="s">has-travis</span>
<span class="pi">-</span> <span class="s">has-pull-approve</span>
<span class="pi">-</span> <span class="s">has-glide</span>
<span class="pi">-</span> <span class="s">has-code-of-conduct</span>
<span class="pi">-</span> <span class="s">has-contributing</span>
<span class="pi">-</span> <span class="s">has-authors</span>
<span class="pi">-</span> <span class="s">has-pull-request-template</span>
<span class="pi">-</span> <span class="s">has-issue-template</span>
<span class="pi">-</span> <span class="s">has-support</span>
<span class="pi">-</span> <span class="s">has-funding</span>
<span class="pi">-</span> <span class="s">has-security```</span>
</code></pre></div></div>

<p>I originally had camel case above to match Go, but realized this was way too hard to remember which to capitalize
and opted for the above. Once we have this file, we hand it over to the command!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>./codestats repo buildsi/build-abi-containers <span class="nt">--config</span> examples/all-stats.yaml 

<span class="c"># I also created one for GitHub "health files"</span>
<span class="nv">$ </span>./codestats repo buildsi/build-abi-containers <span class="nt">--config</span> examples/health-stats.yaml
</code></pre></div></div>

<p>The result of these commands is always a json file, and in a consistent format to include
a list of repositories. Now the next step would be to make this data easy to serach!</p>

<h3 id="web-interface">Web Interface</h3>

<p>I decided to generate a set for the <a href="https://github.com/spack" target="_blank">spack</a> organization, but not including
spack-search because it’s a monster in size. And I tested on the vsoch set, which are a set of files that I think important
for open source projects (although not necessarily spack!). To do this I added a “vsoch-stats.yaml” file to the repository examples:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>./codestats org spack <span class="nt">--skip</span> spack-search <span class="nt">--outfile</span> examples/spack.json <span class="nt">--config</span> examples/vsoch-stats.yaml 
</code></pre></div></div>

<div style="padding: 20px;">
  <img src="https://raw.githubusercontent.com/vsoch/codestats/main/docs/codestats.png" />
</div>

<p>And this is how I generated <a href="https://github.com/vsoch/codestats/blob/main/docs/data.json" target="_blank">this data</a> that renders into the <a href="https://vsoch.github.io/codestats" target="_blank">example interface</a>. As a reminder - a red entry doesn’t always indicate bad - for some of these files, they may just not be appropriate for the organization type. For example, spack (shown at the top) could never have a funding.yaml file. But this is one of my vsoch metrics because I think it’s good to support
open source maintainers.</p>

<p>Pretty neat, huh? It uses Jquery Datatables to provide a searchable table, and you can also sort by columns or search within a column.
Since one repository can span multiple rows, I also added header rows (for only the case when the table is sorted by the repository name),
and a summary table at the very top.</p>

<h3 id="questions">Questions?</h3>

<p>So what else would you like to see? Additional stats or interface support? Please <a href="https://github.com/vsoch/codestats/issues" target="_blank">open an issue</a> and let me know! And thanks for stopping by!</p>