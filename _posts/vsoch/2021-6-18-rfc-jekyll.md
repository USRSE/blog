---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2021-06-18 22:30:00'
layout: post
original_url: https://vsoch.github.io//2021/rfc-jekyll/
title: RFC Jekyll
---

<p>As software engineers we often have to make choices between ideal states.
For example, modular development can better assign responsibility, and make
development more scoped and thus easier to do. But what are the drawbacks?
It could be that choosing a modular approach means that it’s harder to bring
things together – things such as documentation, or in some cases, even community.
But really I’m thinking about the first, because recently I’ve been crafting a plan
to better contribute to the <a href="https://github.com/opencontainers" target="_blank">Open Containers Specifications</a>.
We’ve assembled a group of talented container enthusiasts, and while we are discussing many 
<a href="https://supercontainers.github.io/containers-wg/" target="_blank">interesting ideas</a>,
during our discussion it keeps dawning on me that it’s really hard to keep track of things.
What do I mean? Well, I keep hearing the same story:</p>

<blockquote>
  <p>Is that part of the spec?
Where does it say that?</p>
</blockquote>

<p>The issue is that each single specification is stored in multiple markdown files, and you have to jump
between files and even repositories to find what you are looking for. This might be okay
if you know exactly where to look and have a strong understanding of the belonging of files
(e.g., knowing if a linked markdown is part of the spec or just a reference) but for a new 
contributor, it’s akin to being lost in the markdown wilderness.</p>

<h3 id="rfc-jekyll">RFC Jekyll</h3>

<p>Early in our discussions, <a href="https://github.com/reidpr" target="_blank">@reidpr</a>
had a fantastic idea.</p>

<blockquote>
  <p>I’d love consolidation of the standard into a single coherent document, e.g. RFC style.</p>
</blockquote>

<p>I quickly <a href="https://github.com/supercontainers/containers-wg/issues/9" target="_blank">opened an issue</a>,
and by chance the topic of being able to share spec content was brought up at the next OCI meeting.
I <a href="https://hackmd.io/El8Dd2xrTlCaCG59ns5cwg#Notes1" target="_blank">brought up</a>
the idea and it wasn’t rejected, so I got to work! This is thus the creation of the rfc-jekyll template.
When you arrive at the root, you can choose to view a specification:</p>

<div style="margin: 20px;">
 <a href="https://vsoch.github.io/rfc-jekyll/" target="_blank"><img src="https://raw.githubusercontent.com/vsoch/rfc-jekyll/main/assets/img/rfc-jekyll.png" /></a>
</div>

<p>And then browsing to the specification shows content in the center, a left side navigation with
other files that are defined for the spec, and the right side is a dynamically generated
navigation.</p>

<div style="margin: 20px;">
 <a href="https://vsoch.github.io/rfc-jekyll/runtime-spec/" target="_blank"><img src="https://raw.githubusercontent.com/vsoch/rfc-jekyll/main/assets/img/runtime-spec.png" /></a>
</div>

<p>The content from the many different specs and files are retrieved across files and GitHub repositories
for a holistic, cohesive experience. I find this much easier to navigate than the spec markdown files 
on their own.</p>

<h4 id="design">Design</h4>

<p>I wanted to emulate the style of a modern <a href="https://www.rfc-editor.org/rfc/rfc8843.html" target="_blank">RFC memo</a>,
but rendered totally dynamically. I also wanted to add some nice features, like being able to click to quickly open
a link to edit the document, or open an issue. For the use case of Open Containers, I wanted to render content 
not only from the repository that served the template, but from multiple repositories across GitHub! For
the design that I am prototyping:</p>

<ol class="custom-counter">
<li>You can combine many different resources across multiple repos to look like one holistic documentation base</li>
<li>Content is retrieved dynamically directly from the specs and you don't need to rebuild.</li>
<li>Every section of a spec page has a permalink, and the page has a direct link to edit on GitHub or open an issue.</li>
<li>Adding new pages comes down to adding metadata for the pages you want.</li>
</ol>

<p>I won’t go into details of the template, because if you are interested you can read about it
<a href="https://github.com/vsoch/rfc-jekyll" target="_blank">here</a> and see the preview
<a href="https://vsoch.github.io/rfc-jekyll" target="_blank">here</a>.</p>

<h4 id="questions-for-you">Questions for you!</h4>

<p>Primarily I’m interested in feedback from the community for the following questions:</p>

<ol class="custom-counter">
<li>Would it be useful to still render content from local files?</li>
<li>What kind of supplementary pages would be useful to go alongside specs (working groups, ideas, images?)</li>
<li>What other features would be useful to have?</li>
</ol>

<p>And of course please <a href="https://github.com/vsoch/rfc-jekyll/issues">report any issues or bugs!</a>.
I made this is evenings over a few days, and I consider it early in development.</p>

<h3 id="summary">Summary</h3>

<p>TLDR: This template might be useful to you if you find yourself in similar circumstances of
having scattered markdown files across repositories that you want to make more cohesive.
It might be useful if you want RFC style documentation for Jekyll, which I didn’t see
anyone has created yet. And that’s all I got! I hope that you take a look and can give feedback to make it better.
I’ve made <a href="https://github.com/supercontainers/containers-wg/discussions/34" target="_blank">a GitHub discussion</a>
if you want a more formal discussion board other than issues.</p>