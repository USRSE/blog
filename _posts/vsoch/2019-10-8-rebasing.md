---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-10-08 05:00:00'
layout: post
original_url: https://vsoch.github.io//2019/rebasing/
title: Rebasing in Practical Terms
---

<p>Rebasing is really useful for creating a clean git history. It technically means
changing the original base commit of a branch, but for this post, I want to talk
about it in practical terms. If you want the technical details, there are
<a href="https://seesparkbox.com/foundry/to_squash_or_not_to_squash" target="_blank">plenty of</a> 
<a href="https://blog.carbonfive.com/2017/08/28/always-squash-and-rebase-your-git-commits/" target="_blank">resources</a> available to you. First, let’s quickly answer some questions. And actually, I’ll do just that, with actual speaking
and showing you!</p>



<p>More details are included in writing below. This post is dedicated to <a href="http://cyphar.com" target="_blank">Aleksa Sarai</a> who was the first to ever show me, with care, how to <a href="https://asciinema.org/a/Cfl6HLqYxpcUfRbBli6SbF5Gg" target="_blank">perform a rebase</a>.</p>

<h2 id="when-might-i-want-to-rebase">When might I want to rebase?</h2>

<p>Generally, when you are working on a branch and want to clean up your git history,
it’s easy to rebase with master.</p>

<h2 id="what-are-the-general-steps">What are the general steps?</h2>

<p>While this isn’t a copy paste and go example, it will walk you through general steps.
We generally start with checkout of a new branch from an updated master:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># checkout master</span>
git checkout master

<span class="c"># make sure your upstream master is up to date</span>
git pull origin master

<span class="c"># now checkout a new branch</span>
git checkout <span class="nt">-b</span> add/cool-feature
</code></pre></div></div>

<p>And then we make an awesome change! And we commit it.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># make some changes here, and then commit</span>
git commit <span class="nt">-a</span> <span class="nt">-s</span> <span class="nt">-m</span> <span class="s2">"This is my awesome change"</span>
</code></pre></div></div>

<p>Then we maybe push, and open up a pull request, and are told to make more changes.
Our commits get sloppy.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>git push origin add/cool-feature

<span class="c"># opens pull request, request for changes</span>

<span class="c"># each of the below is associated with some mistake/more changes</span>
git commit <span class="nt">-a</span> <span class="nt">-s</span> <span class="nt">-m</span> <span class="s1">'making this other change'</span>
git commit <span class="nt">-a</span> <span class="nt">-s</span> <span class="nt">-m</span> <span class="s1">'oops'</span>
git commit <span class="nt">-a</span> <span class="nt">-s</span> <span class="nt">-m</span> <span class="s1">'oops'</span>
git commit <span class="nt">-a</span> <span class="nt">-s</span> <span class="nt">-m</span> <span class="s1">'whyyyyyy'</span>
</code></pre></div></div>

<p>Then our git history is a mess! So we do an interactive rebase against master.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>git rebase <span class="nt">-i</span> master
</code></pre></div></div>

<p>And then change all of the crappy commits to “fixup” to squash and disregard the
squashed commit messages. And we push to the branch with force!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>git push origin add/cool-feature <span class="nt">--force</span>
</code></pre></div></div>

<p>That’s it! Happy rebasing, folks.</p>