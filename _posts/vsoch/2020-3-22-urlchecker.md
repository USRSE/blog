---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2020-03-22 14:47:00'
layout: post
original_url: https://vsoch.github.io//2020/urlchecker/
title: Checking static links with urlchecker
---

<p>There are common needs across open source or academic projects that sometimes 
continually hit us in the face. They become such regular annoyances, in fact,
that we even stop seeing them. Such an example is the simple task of checking
static links in any kind of documentation or code. You know, given that you
have umpteen pages of docs, how could you easily check that the links aren’t
broken?</p>

<p>I can give an example. For the <a href="https://us-rse.org" target="_blank">usrse website</a> one of the original
creators had set it up to use <a href="https://github.com/gjtorikian/html-proofer" target="_blank">html-proofer</a> 
that was using an underlying library called <a href="https://github.com/typhoeus/typhoeus/issues/182" target="_blank">typhoeus</a> 
to implement the checking. For a general request, it worked <strong>most of the time</strong> but as you can see
from the link, since the library has no implementation for a retry, this means that a failing
link is common. It was so common, in fact, that I <a href="https://github.com/USRSE/usrse.github.io/issues/171" target="_blank">started to look into</a> how to go about addressing it. Since I was always one to quickly respond to CI
failures, the burden of re-running these failed checks on merges to master (after
the same commit passed for a pull request) was repeatedly on me.</p>

<h2 id="url-checker">URL Checker</h2>

<p>At this point I started to keep my eyes open for other tools in the ecosystem that
might be able to provide such an easy service, checking urls in static files.
I also had my eye on the lookout for a tool that would best serve the scientific
community, likely meaning something in Python. It’s not to say that other languages aren’t
equally good, but rather if something breaks or needs a look in terms of the code,
if the language is something familar, it’s easier for the community to adopt (e.g.,
html-proofer is in ruby, which isn’t common amongst scientific programmers).
It was after a few months that I stumbled on the <a href="https://github.com/urlstechie" target="_blank">urlstechie</a> organization, 
which was created by <a href="https://github.com/SuperKogito" target="_blank">SuperKogito</a> for this exact purpose.</p>

<div style="padding: 20px;">
<img src="https://raw.githubusercontent.com/urlstechie/urlchecker-python/master/docs/urlstechie.png" />
</div>

<p>I was pumped! I forged ahead to open up issues for the features that I saw important,
and wound up doing a few major pull requests:</p>

<ol class="custom-counter">
  <li><a href="https://github.com/urlstechie/urlchecker-action/pull/17" target="_blank">Adding retry parameter</a></li>
  <li><a href="https://github.com/urlstechie/urlchecker-action/pull/19" target="_blank">Prettify-ing the interface</a></li>
  <li><a href="https://github.com/urlstechie/urlchecker-action/pull/24" target="_blank">Optimizing for GitHub actions (local checkout)</a></li>
  <li><a href="https://github.com/urlstechie/urlchecker-action/pull/30" target="_blank">White listing files</a></li>
  <li><a href="https://github.com/urlstechie/urlchecker-action/pull/36" target="_blank">Refactoring action</a> to use <a href="https://github.com/urlstechie/urlchecker-python" target="_blank">urlchecker-python</a></li>
</ol>

<p>The last one was hugely fun, and I did yesterday. What exactly is urlchecker-python? Let’s
talk about that next.</p>

<h3 id="urlchecker-python">urlchecker-python</h3>

<p>Here’s the thing - although the original repository <code class="language-plaintext highlighter-rouge">urlstechie/URLs-checker</code> was a GitHub action, 
it was sort of a Python module and GitHub action squished into one. This happens sometimes when we 
create small snippets of code intended to run as actions, but then realize we want to extend them
beyond that. Being able to reproduce an action locally using the same exact underlying tooling 
is hugely important for developers to be able to do - if I’m going to run the urlchecker for a GitHub workflow test, 
I want to be able to run it locally to reproduce the same tests. We thus
decided to embark on gutting out the core of the GitHub Action (at that time)
and creating a separate Python library. And tada, <a href="https://github.com/urlstechie/urlchecker-python" target="_blank">here it is!</a>.
You can install it with pip:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>pip <span class="nb">install </span>urlchecker
</code></pre></div></div>

<p>And you can now easily check a repository (documentation and code) locally, using the
same parameters that are plugged into the GitHub action:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>urlchecker check <span class="nb">.</span>
</code></pre></div></div>

<p>Here is an example run. This is the same action that is run for the <a href="https://github.com/rseng/awesome-rseng/blob/master/.github/workflows/urlchecker.yml" target="_blank">awesome-rseng</a> repository. This command says to check all markdown files,
but skip the files in docs in the present working directory (.).</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>urlchecker check <span class="nt">--white-listed-files</span> docs <span class="nt">--file-types</span> .md <span class="nb">.</span>
</code></pre></div></div>



<p>So if you have struggled with checking static links in the past, look no further!
Here is a quick example to get you started:</p>

<div class="language-yaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="na">name</span><span class="pi">:</span> <span class="s">URLChecker</span>
<span class="na">on</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">pull_request</span><span class="pi">]</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">check-urls</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout Actions Repository</span>
      <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v2</span>
    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Test GitHub Action</span>
      <span class="na">uses</span><span class="pi">:</span> <span class="s">urlstechie/urlchecker-action@0.1.7</span>
      <span class="na">with</span><span class="pi">:</span> 
        <span class="na">file_types</span><span class="pi">:</span> <span class="s">.md,.py</span>
        <span class="na">white_listed_files</span><span class="pi">:</span> <span class="s">docs</span>
</code></pre></div></div>

<p>I’m having a lot of fun developing these tools, so please <a href="https://github.com/urlstechie/urlchecker-action/issues" target="_blank">open an issue</a> if you have any questions, feature requests, or just want to say hello! We have
a fabulous <a href="https://urlstechie.github.io/" target="_blank">old school style</a> website
with information about urlstechie.</p>

<div style="padding: 20px;">
<img src="https://raw.githubusercontent.com/vsoch/vsoch.github.io/master/assets/images/posts/urlstechie/urlstechie.png" />
</div>

<p>If you are interested in contributing, please send
us a note!</p>