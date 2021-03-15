---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2021-03-14 12:30:00'
layout: post
original_url: https://vsoch.github.io//2021/dinosaur-logic-program/
title: My First Logic Program
---

<p>Today I decided I wanted to write my first logic program. What the heck is a logic program, or (more appropriately)
Answer Set Programming (ASP)? Let’s dive in!</p>

<h3 id="what-is-answer-set-programming">What is Answer Set Programming?</h3>

<p>When you use a package manager, how does it find a solution? If you are familiar with <a href="https://github.com/spack/spack" target="_blank">spack</a>, you’ll know that there is a concretization step (see <a href="https://spack.readthedocs.io/en/latest/command_index.html?highlight=clingo#spack-solve" target="_blank">spack solve</a>) and we somehow find a solution to a bunch of version constraints. One of the (newly added) solvers is called <a href="https://github.com/potassco/clingo" target="_blank">clingo</a>. It’s a grounder and solver for logic programs, and is going to empower us to do Answer Set Programming!</p>

<h3 id="why-should-i-care">Why should I care?</h3>

<h4 id="what-is-clingo">What is Clingo?</h4>

<p>The <a href="https://en.wikipedia.org/wiki/University_of_Potsdam" target="_blank">University of Potsdam</a> in Germany has 
a collection of tools for Answer Set Programming. The set of tools is referred to
as “Potassco” (Potsdam Answer Set Solving Collection) and Clingo is one of these tools. 
There is a <a href="https://www.cs.utexas.edu/users/vl/teaching/378/ASP.pdf" target="_blank">great reference</a>
that can walk you through all the details, and there are even
online <a href="https://teaching.potassco.org/" target="_blank">teaching resources</a>
that you can use to jump in and learn.</p>

<h4 id="what-does-it-do">What does it do?</h4>

<p>From a very high level standpoint, you as a developer can express combinatorial problems as logic programs (files with extension “lp”) that
define statements or facts (atoms) and then a set of rules. Once we have these statements and rules,
we can ask Clingo to find a stable model. While the details are out of scope for this post, I recommend reading
<a href="https://www.cs.utexas.edu/users/vl/teaching/378/ASP.pdf" target="_blank">this guide</a> to learn more or
checking out the <a href="https://potassco.org/doc/start/" target="_blank">getting started</a> pages on the site.</p>

<h4 id="ok-so-why-should-i-care">Ok, so why should I care?</h4>

<p>This isn’t just learning a new language. It’s learning a new way of thinking.
I think that it’s important for research software engineers to add these concepts to their toolbelt for several reasons.</p>

<h5 id="1-new-models-for-problem-solving">1. New Models for Problem Solving</h5>

<p>The first is that we have become so accustomed to writing functions and carrying around logic in data structures
that we may have forgotten that there are entirely different ways to model problems. For example, instead of trying
to hard code some logic and navigate a data structure, we could flatten the entire thing into statements (atoms and rules),
and ask clingo (or more generally, a solver) for a solution.</p>

<h5 id="2-generalizability">2. Generalizability</h5>
<p>The second reason is that this kind of approach allows for generalizability. What I mean is that we might have similar problems,
perhaps represented in different programming languages, and they might be generalized into logic statements
and then used across languages. I say this because I think logic can be viewed as a basis for thinking, and
for algorithms. It’s (I think) one of the lowest levels we can use to model a problem.</p>

<h5 id="2-reduction">2. Reduction</h5>

<p>I would say using a solver like Clingo is taking a reductionist approach because we are
forced to break down problems into the smallest components. As I alluded to earlier,
I think having this new way of thinking can be hugely useful for a software engineer.
Being a good software engineer doesn’t always mean knowing every fact, or being an expert in every domain,
but rather being creative when it comes to solving problems. The more ways that you can
think about a problem, arguably the more creative you can be.</p>

<h3 id="my-first-program">My First Program!</h3>

<p>Let’s write a logic program! For my first shot (knowing almost nothing but getting the gist of
what they look like) I absolutely had to ask a very compelling question:</p>

<blockquote>
  <p>Am I a dinosaur?</p>
</blockquote>

<p>I decided to use the clingo container provided by
the <a href="https://github.com/orgs/autamus/packages/container/package/clingo" target="_blank">autamus registry</a>,
which is an effort to create a registry of spack package containers hosted on GitHub packages.
I want to give a shoutout to <a href="https://github.com/alecbcs" target="_blank">alecbcs</a> who has championed
this idea and let me help along the way! We have this clingo container that is absolutely perfect
for the quick project that I wanted to do today to write this script. Let’s first pull the 
Clingo container so we have an interactive environment to work in.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>docker run <span class="nt">--rm</span> <span class="nt">-it</span> ghcr.io/autamus/clingo:latest bash
</code></pre></div></div>

<p>Clingo will be installed via spack, and already on our path! It’s installed in
what is called a <a href="https://spack.readthedocs.io/en/latest/workflows.html#filesystem-views" target="_blank">spack view</a>, 
which I suspect is modeled after a database view, if you are interested. Do we have
clingo in the container?</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># which clingo</span>
/opt/view/bin/clingo
</code></pre></div></div>

<p>Yes! Now let’s write a small logic program, which we will put in a file called <code class="language-plaintext highlighter-rouge">dinosaur.lp</code>.
It will help us determine if a living thing is a dinosaur. It will start with a set of
statements (truths about what we know) and then end with a rule that defines what it means
to be a dinosaur based on these statements. Keep in mind that this is a hugely simple
example, and clingo has much more functionality than this.</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
% These are blanket facts, statements that each of these is living
% I think these are called atoms
living(vanessa).
living(fernando).
living(maria).

% This tells use size of arms for each living thing
armsize(vanessa, "small").
armsize(fernando, "large").
armsize(fernando, "small").

% A boolean to say we can roar!
canroar(vanessa).

% An entity is a dinosaur if they are living, have tiny arms, and can roar.
dinosaur(Entity) :- living(Entity), armsize(Entity, "small"), canroar(Entity).

</code></pre></div></div>

<p>To look for a solution, we can run clingo (note that I broke the output below
apart into separate lines for easier readability):</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># clingo dinosaur.lp </span>
clingo version 5.5.0
Reading from dinosaur.lp
Solving...
Answer: 1
canroar<span class="o">(</span>vanessa<span class="o">)</span> armsize<span class="o">(</span>vanessa,<span class="s2">"small"</span><span class="o">)</span> 
                 armsize<span class="o">(</span>fernando,<span class="s2">"large"</span><span class="o">)</span>
                 armsize<span class="o">(</span>fernando,<span class="s2">"small"</span><span class="o">)</span>
                 living<span class="o">(</span>vanessa<span class="o">)</span>
                 living<span class="o">(</span>fernando<span class="o">)</span>
                 living<span class="o">(</span>maria<span class="o">)</span>
                 dinosaur<span class="o">(</span>vanessa<span class="o">)</span>
SATISFIABLE

Models       : 1
Calls        : 1
Time         : 0.003s <span class="o">(</span>Solving: 0.00s 1st Model: 0.00s Unsat: 0.00s<span class="o">)</span>
CPU Time     : 0.001s
</code></pre></div></div>

<p>See the last entry in the list? Clingo has told us that vanessa is a dinosaur, so I must be a dinosaur! I can’t
explain why but writing this (and having it work) gave me immense joy. It’s also
Pi day (Happy Pi Day!) If you are interested, I recommend the pdf I linked previously,
and also checking out how the <a href="https://github.com/spack/spack/blob/develop/lib/spack/spack/solver">solver for spack</a>
generally works. That’s all I got!</p>