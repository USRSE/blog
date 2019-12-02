---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-12-01 10:30:00'
layout: post
original_url: https://vsoch.github.io//2019/sourcecred/
title: Showcase your Contributors!
---

<p>How do we measure and share contributions? If you’re familiar with GitHub, you’ve probably
seen the contributors graphic. For example, for Singularity Registry Server, it looks like this:</p>

<div style="padding: 20px;">
   <img src="https://vsoch.github.io/assets/images/posts/sourcecred/github-commits.png" style="width: 100%;" />
</div>

<p>What’s the problem here? It only focuses on commits, additions or deletions, or otherwise
changes to the code. The reality is that <span style="font-style: italic;">there are many
ways for individuals to contribute to a community</span>. This graph is completely missing
the number of people that have posted issues, asked questions, requested features,
helped to test, and worked on the documentation. Especially for Singularity Registry Server,
where development is completely driven by user needs, this graphic is totally disappointing.</p>

<h2 id="sourcecred">Sourcecred</h2>

<p>I stumbled on <a href="https://sourcecred.io" target="_blank">Sourcecred</a> earlier in
the year through common people in the Sustainable Free and Open Source Communities, also known as 
<a href="https://sfosc.org/" target="_blank">sfosc.</a> It was a beautiful idea - a tool that would capture all kinds of contributions to a community, and (for advanced usage) even let you customize how you weighted those contributions. The core
tool produces a more detailed contributors graph, but what I got really excited about was a widget
extension to convert that to a grid of contributors. I probably wasn’t going to run server to host
sourcecred anytime soon, but I definitely could generate and statically display a vector graphic. 
With this goal, I helped over a few months to set up continuous integration and container builds, and create an unnoficial GitHub workflow example for how to easily generate a graphic for a repository readme, <a href="https://github.com/sourcecred/sourcecred-action" target="_blank">sourcecred-action</a>. Here’s the final, and what I believe to
be more proper representation of all the folks that have been involved with Singularity
Registry Server:</p>

<div style="padding: 20px;">
   <img src="https://vsoch.github.io/assets/images/posts/sourcecred/contributors.png" style="width: 100%;" />
</div>

<p>And see it live <a href="https://github.com/singularityhub/sregistry" target="_blank">here</a>.
A lot of our smaller projects may never be grand like some of the larger ones on GitHub hosted by
companies, but we should be proud of our small communities nonetheless.</p>

<p><br /></p>
<hr />

<h3 id="how-does-it-work">How does it work?</h3>

<h4 id="1-add-the-workflow">1. Add the workflow</h4>
<p>Briefly, you add one of two workflows to your repository.</p>

<ol class="custom-counter">
 <li><a href="https://github.com/sourcecred/sourcecred-action/blob/master/.github/workflows/pull_request.yml" target="_blank">.github/workflows/pull_request.yml</a> shows an example workflow to run a scheduled job to generate the graphic, and open a pull request for you to review.</li>
 <li><a href="https://github.com/sourcecred/sourcecred-action/blob/master/.github/workflows/automated.yml" target="_blank">.github/workflows/automated.yml</a> is the same process, but pushes directly to some branch.</li>
</ol>

<p>You would of course choose the frequency and method that is most appropriate for your repository. For example,
for Singularity Registry Server I chose the pull request approach, and I only want it to run once a month for my
own maintainer sanity.</p>

<h4 id="2-link-the-graphic">2. Link the Graphic!</h4>

<p>And of course you can link to the graphic in the README! <a href="https://www.github.com/beanow" target="_blank">@Beanow</a>
showed me this trick to link to an svg:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">![</span>contributors.svg]<span class="o">(</span>./contributors.svg<span class="o">)</span>
</code></pre></div></div>

<p><br /></p>

<h4 id="3-running-the-action">3. Running the Action</h4>

<p>When the schedule triggers, your GitHub action will run! In the picture below, we see the expected steps
of setting up the job and checking out the repository. The next step caches a folder (sourcecred_data) 
to help folks out with larger repositories (the GitHub API is used to summarize contributions across contribuors).</p>

<div style="padding: 20px;">
   <img src="https://vsoch.github.io/assets/images/posts/sourcecred/github-action.png" style="width: 100%;" />
</div>

<p>In the build step (we could better name this step) we then use the <a href="https://hub.docker.com/r/sourcecred/sourcecred/tags" target="_blank">sourcecred/sourcecred:dev</a> 
and <a href="https://hub.docker.com/r/sourcecred/widgets/tags" target="_blank">sourcecred/widgets</a> containers
to generate the sourcecred data (the first container), and then the graphic (the widgets container). We checkout
a new branch (stamped by the prefix “update/sourcecred-“ and ending with the date and push to the repository,
and then open a pull request with it. The updated data is then cached, and the job complete.</p>

<h4 id="3-reviewing-the-pull-request">3. Reviewing the Pull Request</h4>

<p>As the maintainer, you get a nice email that the pull request is open!</p>

<div style="padding: 20px;">
   <img src="https://vsoch.github.io/assets/images/posts/sourcecred/github-actions-email.png" style="width: 100%;" />
</div>

<p>And then can review the pull request.</p>

<div style="padding: 20px;">
   <img src="https://vsoch.github.io/assets/images/posts/sourcecred/github-pr.png" style="width: 100%;" />
</div>

<p>Actually, the pull request will only have one changed file - the one above was generated when I was adding the action, which had two new files for the workflow, an update to the README to link the graphic, and the graphic itself. So in practice this should show just one changed file. And of course if you wanted to commit directly to some GitHub pages (or other) branch, you could do that too, and skip step 4 entirely.</p>

<h2 id="why-is-this-important">Why is this important?</h2>

<p>There is an over-emphasis sometimes on contributions in the way of code. There isn’t enough
attention given to people who are generally nice, and serve to welcome new members, answer questions,
and even post little emoticons on posts to make the other person feel valued. There isn’t enough emphasis
on folks who are spending a lot of time posting issues, or resolving them. By way of using a tool
like sourcecred, we can value them too. And possibly someday, open source contributions can be better
acknowledged, financially or otherwise. But that is a conversation for another day. Even if you don’t
use sourcecred, I hope that you take some time to evaluate how your community is showing appreciation 
for its contributors, even if it’s just a graphic to show smiling faces on your
repository.</p>