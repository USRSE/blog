---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2020-03-11 12:54:00'
layout: post
original_url: https://vsoch.github.io//2020/needs-love/
title: Needs Love
---

<p>Have you ever found that learning is most fun when you are working on something
that you care about? I’ve most definitely realized over the years that I best learn
when I’m working on an actual problem. For much of this learning I’m on my own,
and in very rare cases I get to work on a project that someone else cares about,
and working together (or individually with support) is amazingly fun. So - how do
we do more of that?</p>

<h2 id="needs-love">Needs Love</h2>

<p>I would bet that you can iterate over a conference room of people, and every
single person would have at least one project or idea that they were wanting
to work on, but just didn’t have the time. This is the goal of a new project
that I’ve started called <a href="https://github.com/rseng/needs-love" target="_blank">needs love</a>.</p>

<p>Specifically, it’s a simple repository where you can post an idea or project
that needs a little love, or you can go to find one. The goal is to be able
to  match research software engineers with projects that need them.
I hope that it could be a small peer-to-peer network for learning by way of doing.
It’s based off of the <a href="https://github.com/rseng/good-first-issues" target="_blank">good first issues</a>
action and interface, but scoped and branded for needs-love issues posted
directly to the repository.</p>

<div style="padding: 20px;">
<img src="https://raw.githubusercontent.com/rseng/needs-love/master/img/needs-love.jpg" />
</div>

<p>If you are interested, there are also <a href="https://github.com/rseng/needs-love/tree/master/.github/workflows" target="_blank">GitHub actions</a> 
set up to do simple label management. When an issue is labeled with “needs-love” it is added
to the web interface nightly, and when the “matched” label is added to indicate that
someone is going to work on the issue, the needs-love label is removed.</p>

<h3 id="what-needs-love">What Needs Love?</h3>

<p>You might feel overwhelmed or frustrated by a lack of time to finish
some of your projects, but have you ever thought about the opportunity
that such unfinished work offers? For research software engineers that don’t work in a group, or largely
exist in silos, finding consistent sources of challenge is a challenge
in and of itself. And guess what - an unfinished project and an RSE
hungry to learn is a match made in heaven. If you are submitting a project or idea
as an issue, you get it rolling again, and the RSE has an opportunity to grow. This is a problem
I find myself continually in - hence why I embark on so many seemingly random
projects.</p>

<blockquote>
  <p>In a nutshell, both the projects and the RSEs need a little love.</p>
</blockquote>

<h2 id="who-is-involved">Who is Involved?</h2>

<p>I wouldn’t pass up a chance to make a bread metaphor, so here we go!</p>

<h3 id="starters">Starters</h3>

<p>We aren’t talking about bread, folks, but the pun is quite lovely. A starter
has put some time and thought into a recipe, and very likely started the baking
process, but doesn’t have the bandwidth to finish the dough and put it in the oven.</p>

<h4 id="what-makes-a-good-starter">What makes a good starter?</h4>

<p>A good starter has expertise in some area, or a project started, but doesn’t have time to finish up.
It’s also okay to just have an idea that you want to throw out there for someone to work on.
You might not even know what the best implementation might be! Thus, a good starter
can also just have a good idea.</p>

<h3 id="finishers">Finishers</h3>

<p>Finishers are hungry for opportunity, and fun projects. A finisher can be
very independent and curious, or ask for a bit more help. The finisher wants
to take some project that has been started, and turn it into delicious bread,
or an idea that isn’t baked yet, and write the recipe.</p>

<h4 id="what-makes-a-good-finisher">What makes a good finisher?</h4>

<p>The finisher, regardless of skill, should be motivated to take ownership of a project
or an idea.</p>

<h3 id="submitting-a-project-or-idea">Submitting a Project or Idea</h3>

<p>You can submit a project or idea that needs love simply by opening an issue, and selecting
the “Project or Idea that needs love” template.</p>

<div style="padding: 20px;">
<img src="https://raw.githubusercontent.com/rseng/needs-love/master/img/needs-love-template.png" />
</div>

<p>Once your idea is submit, it will (each night) render to the interface of
projects that need love, which you can <a href="https://rseng.github.io/needs-love/" target="_blank">see here</a>.</p>

<div style="padding: 20px;">
<img src="https://raw.githubusercontent.com/rseng/needs-love/master/img/needs-love-interface.png" />
</div>

<h3 id="matching-yourself">Matching Yourself</h3>

<p>When you navigate the web interface to find an issue that looks interesting,
if you click the issue to navigate to the GitHub interface, you can have discussion
with the author about how you might like to be involved.</p>

<h4 id="what-should-you-discuss">What should you discuss?</h4>

<p>The match between the starter, both project and individual, and the finisher,
is very important! While the criteria might vary by project, it’s generally a good
idea to open up a discussion on one of the <a href="https://github.com/rseng/needs-love/issues?q=is%3Aissue+is%3Aopen+label%3Aneeds-love" target="_blank">needs-love</a> issues and discuss the following:</p>

<ol class="custom-counter">
  <li>if expertise is known to be needed, where will it come from?</li>
  <li>how can both parties best communicate, and ask questions?</li>
  <li>what kind of time frame do you have in mind?</li>
  <li>do you have all the resources that you need?</li>
</ol>

<p><br /></p>

<p>If there is a match, great! You can match yourself simply by applying the <code class="language-plaintext highlighter-rouge">matched</code> label. A GitHub action
will handle removing the <code class="language-plaintext highlighter-rouge">needs-love</code> label so that others know that you’ve taken
charge. The interface will update over night.</p>

<h3 id="updating-a-match">Updating a Match</h3>

<p>If you can no longer work on a project or idea, you should notify the starter 
(the person that originally submit the issue) about this change, and they
can remove the <code class="language-plaintext highlighter-rouge">matched</code> label. Once this is done, a GitHub action will
add back the <code class="language-plaintext highlighter-rouge">needs-love</code> label so that others know that the issue is open to be
worked on.</p>

<h2 id="what-if-someone-doesnt-ask-for-help">What if someone doesn’t ask for help?</h2>

<p>Asking for help can be hard! In some cases, asking for help can look like labeling
an issue with “Good First Issue” or (more directly) “Help Wanted.” For the interested folks that want to
persue a list of open source, research software engineering efforts, the rseng organization also provides the 
<a href="https://github.com/rseng/awesome-rseng" target="_blank">awesome-rseng</a> awesome list.
You can either contribute to these projects directly, or find one or more issues and write
them up under needs-love to be discovered.</p>

<h2 id="how-can-i-contribute">How can I contribute?</h2>

<p>If you want to find a project to help with, you can browse the
<a href="https://github.com/rseng/needs-love/issues?q=is%3Aissue+is%3Aopen+label%3Aneeds-love" target="_blank">issue board</a>
 or the <a href="https://rseng.github.io/needs-love/" target="_blank">interface here</a>.
It’s also very helpful to peruse around some of your projects, or open source projects
you contribute to, and look for good “needs love” issues. Many times, you can also have discussion
with colleagues to find what they are working on (and wish they had more time to work on).</p>