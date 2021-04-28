---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2021-03-31 17:30:00'
layout: post
original_url: https://vsoch.github.io//2021/singularity-hub-archive/
title: Singularity Hub Archive
---

<p>As of 5:00pm today, April 26th 2021, the Singularity Hub cloud space, including
storage of over 9TB of containers, a registry server and builders maintained by
me (and literally only me) since 2016, is shut off. Before you worry,
we’ve migrated all the existing containers (before the 19th of this month) to
an archive so they will continue to be pullable at their previous URLs, something 
I’ll discuss briefly later in this post. For now, let’s take a few moments
to appreciate the server previously known as Singularity Hub, now known as 
Singularity Hub archive.</p>

<h2 id="too-long-didnt-read">Too Long, Didn’t Read</h2>

<p>If you have hosted containers on Singularity Hub, you primarily should know that they
will continue to be pullable from the archive, however you won’t be able to build
new containers. You should disable the previous webhook in your GitHub repositories,
an oversight I realized after I revoked all the GitHub tokens for the integration.
If you added a Singularity Hub Badge to your repository, I’ve also
generated a newly linked one that you can use as follows (see the gist for the
markdown):</p>



<p>For the complete details and other options for building containers, you should
read <a href="https://singularityhub.github.io/singularityhub-docs/2021/going-read-only/" target="_blank">the initial news post</a>.
If you are feeling nostalgic, take a look at the <a href="https://singularityhub.github.io/singularityhub-docs/lastday/" target="_blank">last day gallery</a>
that I’ve prepared in Singularity Hub’s honor. 
If you have an issue or question that you want to post for the new archive, you can do so 
<a href="https://github.com/con/shub/issues" target="_blank">here</a>.
This post will continue to give Singularity Hub one last honor before it becomes a
distant memory.</p>

<h2 id="background">Background</h2>

<h3 id="a-need-for-reproducible-science">A Need for Reproducible Science</h3>

<p>You can read the background story <a href="https://singularityhub.github.io/singularityhub-docs/docs/introduction#a-need-for-reproducible-science" target="_blank">here</a>,
or watch an interactive version <a href="https://vsoch.github.io/containers-story/" target="_blank">here</a>.
I started using containers (Docker, specifically) as a naive graduate student in early 2015, quickly fell in love,
and decided it would be my mission to bring containers to our HPC cluster at Stanford. It felt immensely important for scientific reproducibility -
or being able to preserve an analysis pipeline by some means (containers are a good start) to run again
and reproduce it. I built the original version of Singularity Hub, wrote a paper</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>Sochat V, Prybol CJ, Kurtzer GM (2017)
Enhancing reproducibility in scientific computing: Metrics and registry for Singularity containers.
PLoS ONE 12(11): e0188511. https://doi.org/10.1371/journal.pone.0188511
Encapsulation of Environments with Containers¶
</code></pre></div></div>

<p>and charged forward as an advocate and developer for all things containers.
This lasted a few years, until containers “were definitely at thing”  and then I stepped back.
While discussion of the change in the Singularity community itself from 2016 until now is out
of scope for this post, I will say that despite the changes, I still care about supporting the Singularity user base and 
maintain several <a href="https://singularityhub.github.io" target="_blank">Singularity-associated projects</a>.</p>

<h3 id="what-happens-in-the-future">What happens in the future?</h3>

<p>At the time that I designed Singularity Hub the thought that I likely wouldn’t be 
able to maintain it forever loomed in my awareness, but I had hope that the company that grew out
of Singularity would soon build a service to replace it. They technically did, but unfortunately the
free tier was not suitable for what scientific developers would need. And there certainly wasn’t
a business model for (most) academics paying for a service, so ultimately I would keep Singularity Hub
online much longer than I originally anticipated. A year and then two grew into five years, and in
that time I did whatever it took to keep the server and builders running. This meant several
refactors, and pursuring funding for the service. While my group
could pay small amounts every now and then, the burden of costs was not reasonable, so I ultimately
went to Google early on and asked for help. I suspect people have different views about Google,
but before you express any negative sentiment remember that at large companies there are people, 
and I can solidly state that the people who I worked with showed infinite kindness and support for this tiny effort. I am forever grateful to
Google for supporting Singularity Hub for all these years – without them it never would
have happened. Thank you Google!!</p>

<div style="margin: 20px; padding-top: 20px;">
<img src="https://raw.githubusercontent.com/singularityhub/singularityhub-docs/master/assets/img/lastday/1Screenshot_2021-04-25%20singularity-hub(1).png" />
</div>

<p>At the beginning of this year, my time at Stanford was coming to a close. I had finished what I
set out to do, and it was time for me to move on. At this point, I reached out to the community
for help. At best someone could take over the service, and at worst it would simply go away entirely,
including all containers. Thankfully the <a href="https://github.com/con/" target="_blank">Center for Open Neuroscience</a>
and more specifically, <a href="https://github.com/yarikoptic" target="_blank">Yarikoptic</a> took the reins.
We had a simple plan to migrate all the containers and metadata from storage, and then serve
the same containers at an archive at Dartmouth. All previously built public containers before
the 19th of April would still be available to pull, ensuring reproducibility of those
pipelines from all those years. You can in fact now navigate to 
<a href="https://singularity-hub.org" target="_blank">singularity-hub.org</a> to see the new home!
I tried many times to convince Yarik that I should send him ice cream, but I was not successful.</p>

<h2 id="is-reproducibility-possible">Is Reproducibility possible?</h2>

<p>I’ve <a href="https://vsoch.github.io/2017/reproducible-impossible/" target="_blank">written about this before</a>, but 
after maintaining a service and having pretty good knowledge about scientific workflows and software, I’m
not convinced that reproducibility is much more than a pipe dream. Yes, I agree that if you use
the right technologies, have great documentation, and try to ensure your data and code is preserved over
time, the chances that your original analysis will work in 5 years are pretty good. The problem is when 
we scale out beyond our lifetimes. Have you seen all those GitHub accounts that belong to people
that have died because of old age? Probably not, because GitHub has only been around since 2011.
But do you ever wonder what happens to your code, and your various services you use to support it,
when you are no longer around? The answer is that you need people to continue supporting it.</p>

<blockquote>
  <p>This is where I will say that sustainability is less about doing the right thing, and more about survival of the valued.</p>
</blockquote>

<p>Your code <em>will</em> endure if it continues to be valued. Even if technologies change, if your
analysis pipeline or service adds significant value to the community, people will find a way to keep it up
to date and running. If you mysteriously go away and leave a hugely useful project unmaintained, someone
will fork it and keep it alive. Yes, it’s very likely the case that the awareness and branding that you create around your code will
have an impact. For example, you can make the greatest software known to man, but nobody will care
if they don’t know about it. But let’s face it. If software or an analysis pipeline
does not add value to the world, even if you got a shiny paper out of it, why should there be effort
to keep it alive? I used to be a reproducibility stickler and want to save all-the-things, but
now I’m more of a reproducibility evolutionist that expects software to grow, change, and die, and
wants to support an ecosystem around that.</p>

<p>For these reasons, when I create something these days, I take a needs-based development approach. 
Sure - when a piece of software is new and I think it could be useful, I spend some time getting
the word out. But once the word is out and there are crickets on the GitHub issue board? It’s probably
not useful, and I should focus my time on other things. The reality of being a developer is that 99%
of what I do is probably not really useful. I’m okay with that, because the 1% can have a huge impact.
This is why after something is solidly released in a pretty good state, I only continue to work on it if users open issues and ask me to.</p>

<h2 id="was-singularity-hub-valuable">Was Singularity Hub Valuable?</h2>

<p>Putting on my reproducibility evolutionist hat, I’m going to say that Singularity Hub was valuable for
those first few years when there weren’t many other options. Although it was special to me and used by a few thousand, 
I’m then going to say that the environment changed so that it wasn’t absolutely essential.
There are many different kinds of registries, tricks for building and storing containers, and even
new container technologies for HPC that, over time, brought down the overall value.
This isn’t to say that it’s not important to maintain containers that were previously built. In a way
this is a promise I had hoped to keep, and I’m immensely grateful to be able to do that.</p>

<h2 id="a-future-registry">A Future Registry?</h2>

<p>I have a few ideas for how we could continue to serve GitHub built and deployed
containers from the shub unique resource identifier, but will hold off on sharing
until I’ve had some time to think it over, and test. I do think that, other than
the Sylabs library and Docker Hub (or more generally OCI registries) there should
be another simple option.</p>

<h2 id="one-last-look">One Last Look</h2>

<p>Here is a quick glance at the final state of Singularity Hub.
Note that containers, unless they were frozen, would get over-written with a new
build (a user pushing to GitHub) and this is how I managed to keep the number down.
My original design did not do this, and grew much larger much more quickly.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1"># A container is associated with a sif binary
</span><span class="o">&gt;</span> <span class="n">Container</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="n">count</span><span class="p">()</span>
<span class="mi">7202</span>

<span class="c1"># A collection of containers
</span><span class="o">&gt;</span> <span class="n">Collection</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="n">count</span><span class="p">()</span>
<span class="mi">3901</span>

<span class="c1"># An owner of a collection
</span><span class="o">&gt;</span> <span class="n">User</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="n">count</span><span class="p">()</span>
<span class="mi">6530</span>

<span class="c1"># A builder associated with a container
</span><span class="o">&gt;</span> <span class="n">Build</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="n">count</span><span class="p">()</span>
<span class="mi">8208</span>

<span class="c1"># I'm surprised people went out of their way to star collections!
</span><span class="n">Star</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="n">count</span><span class="p">()</span>
<span class="o">&gt;</span> <span class="mi">373</span>

<span class="c1"># This is the number of collections that had recorded pulls in the last year.
</span><span class="o">&gt;</span> <span class="n">APIRequestCount</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="n">count</span><span class="p">()</span>
<span class="mi">2868</span>
</code></pre></div></div>

<p>Finally, just for the records that I have for the last few months, I can
count the number of requests to pull containers across all of Singularity Hub.</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>total requests 695,493
</code></pre></div></div>

<p>That’s pretty good! The oldest container, because of the first refactor in 2017,
is only from 2017.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">&gt;</span> <span class="n">Container</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="n">order_by</span><span class="p">(</span><span class="s">'build_date'</span><span class="p">)[</span><span class="mi">0</span><span class="p">]</span>
<span class="o">&lt;</span><span class="n">Container</span><span class="p">:</span> <span class="n">alaindomissy</span><span class="o">/</span><span class="n">rnashapes</span><span class="p">:</span><span class="n">tag2</span><span class="o">&gt;</span>

<span class="n">datetime</span><span class="p">.</span><span class="n">datetime</span><span class="p">(</span><span class="mi">2017</span><span class="p">,</span> <span class="mi">10</span><span class="p">,</span> <span class="mi">15</span><span class="p">,</span> <span class="mi">18</span><span class="p">,</span> <span class="mi">24</span><span class="p">,</span> <span class="mi">56</span><span class="p">,</span> <span class="mi">188137</span><span class="p">,</span> <span class="n">tzinfo</span><span class="o">=&lt;</span><span class="n">UTC</span><span class="o">&gt;</span><span class="p">)</span>
</code></pre></div></div>

<p>There were surprisingly a handful of demos (documentation, asciinema, or other
content related to a container build) despite the feature being disabled pretty
early on:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">&gt;</span> <span class="n">Demo</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="n">count</span><span class="p">()</span>
<span class="mi">29</span>
</code></pre></div></div>

<p>And very not surprisingly, many labels and apps associated with containers!</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">&gt;</span> <span class="n">Label</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="n">count</span><span class="p">()</span>
 <span class="mi">18326</span>

<span class="o">&gt;</span> <span class="n">App</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="n">count</span><span class="p">()</span>
<span class="mi">1189</span>
</code></pre></div></div>

<p>It would be fun to do a weekend project and parse those over. Finally, we can
look at the all time leaders for API requests (I’ll truncate at N=500).</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">miguelcarcamov</span><span class="o">/</span><span class="n">container_docker</span><span class="p">:</span> <span class="mi">140186</span>
<span class="n">ikaneshiro</span><span class="o">/</span><span class="n">singularityhub</span><span class="p">:</span> <span class="mi">87991</span>
<span class="n">singularityhub</span><span class="o">/</span><span class="n">busybox</span><span class="p">:</span> <span class="mi">49156</span>
<span class="n">vsoch</span><span class="o">/</span><span class="n">singularity</span><span class="o">-</span><span class="n">images</span><span class="p">:</span> <span class="mi">49036</span>
<span class="n">GodloveD</span><span class="o">/</span><span class="n">busybox</span><span class="p">:</span> <span class="mi">48454</span>
<span class="n">vsoch</span><span class="o">/</span><span class="n">hello</span><span class="o">-</span><span class="n">world</span><span class="p">:</span> <span class="mi">26195</span>
<span class="n">GodloveD</span><span class="o">/</span><span class="n">lolcow</span><span class="p">:</span> <span class="mi">19897</span>
<span class="n">tikk3r</span><span class="o">/</span><span class="n">lofar</span><span class="o">-</span><span class="n">grid</span><span class="o">-</span><span class="n">hpccloud</span><span class="p">:</span> <span class="mi">14454</span>
<span class="n">bouthilx</span><span class="o">/</span><span class="n">repro</span><span class="o">-</span><span class="n">hub</span><span class="p">:</span> <span class="mi">14080</span>
<span class="n">singularityhub</span><span class="o">/</span><span class="n">centos</span><span class="p">:</span> <span class="mi">13144</span>
<span class="n">vsoch</span><span class="o">/</span><span class="n">singularity</span><span class="o">-</span><span class="n">hello</span><span class="o">-</span><span class="n">world</span><span class="p">:</span> <span class="mi">9764</span>
<span class="n">singularityhub</span><span class="o">/</span><span class="n">ubuntu</span><span class="p">:</span> <span class="mi">8392</span>
<span class="n">bouthilx</span><span class="o">/</span><span class="n">sgd</span><span class="o">-</span><span class="n">space</span><span class="o">-</span><span class="n">hub</span><span class="p">:</span> <span class="mi">8001</span>
<span class="n">marcc</span><span class="o">-</span><span class="n">hpc</span><span class="o">/</span><span class="n">tensorflow</span><span class="p">:</span> <span class="mi">7786</span>
<span class="n">dominik</span><span class="o">-</span><span class="n">handler</span><span class="o">/</span><span class="n">AP_singu</span><span class="p">:</span> <span class="mi">7246</span>
<span class="n">gem</span><span class="o">-</span><span class="n">pasteur</span><span class="o">/</span><span class="n">Integron_Finder</span><span class="p">:</span> <span class="mi">5757</span>
<span class="n">aardk</span><span class="o">/</span><span class="n">jupyter</span><span class="o">-</span><span class="n">casa</span><span class="p">:</span> <span class="mi">4320</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_tscan</span><span class="p">:</span> <span class="mi">3182</span>
<span class="n">nuitrcs</span><span class="o">/</span><span class="n">fenics</span><span class="p">:</span> <span class="mi">3169</span>
<span class="n">truatpasteurdotfr</span><span class="o">/</span><span class="n">singularity</span><span class="o">-</span><span class="n">alpine</span><span class="p">:</span> <span class="mi">3034</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_mpath</span><span class="p">:</span> <span class="mi">3006</span>
<span class="n">ravi</span><span class="o">-</span><span class="mi">0841</span><span class="o">/</span><span class="n">singularity</span><span class="o">-</span><span class="n">tensorflow</span><span class="o">-</span><span class="mf">1.14</span><span class="p">:</span> <span class="mi">2867</span>
<span class="n">pegasus</span><span class="o">-</span><span class="n">isi</span><span class="o">/</span><span class="n">montage</span><span class="o">-</span><span class="n">workflow</span><span class="o">-</span><span class="n">v2</span><span class="p">:</span> <span class="mi">2745</span>
<span class="n">marcc</span><span class="o">-</span><span class="n">hpc</span><span class="o">/</span><span class="n">pytorch</span><span class="p">:</span> <span class="mi">2479</span>
<span class="n">NBISweden</span><span class="o">/</span><span class="n">K9</span><span class="o">-</span><span class="n">WGS</span><span class="o">-</span><span class="n">Pipeline</span><span class="p">:</span> <span class="mi">2441</span>
<span class="n">staeglis</span><span class="o">/</span><span class="n">HPOlib2</span><span class="p">:</span> <span class="mi">2051</span>
<span class="n">datalad</span><span class="o">/</span><span class="n">datalad</span><span class="o">-</span><span class="n">container</span><span class="p">:</span> <span class="mi">1994</span>
<span class="n">nickjer</span><span class="o">/</span><span class="n">singularity</span><span class="o">-</span><span class="n">rstudio</span><span class="p">:</span> <span class="mi">1939</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_projected_paga</span><span class="p">:</span> <span class="mi">1885</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_ouijaflow</span><span class="p">:</span> <span class="mi">1880</span>
<span class="n">UMMS</span><span class="o">-</span><span class="n">Biocore</span><span class="o">/</span><span class="n">singularitysc</span><span class="p">:</span> <span class="mi">1863</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_waterfall</span><span class="p">:</span> <span class="mi">1858</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_angle</span><span class="p">:</span> <span class="mi">1851</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_scorpius</span><span class="p">:</span> <span class="mi">1844</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_comp1</span><span class="p">:</span> <span class="mi">1838</span>
<span class="n">onuryukselen</span><span class="o">/</span><span class="n">singularity</span><span class="p">:</span> <span class="mi">1824</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_forks</span><span class="p">:</span> <span class="mi">1816</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_projected_tscan</span><span class="p">:</span> <span class="mi">1814</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_embeddr</span><span class="p">:</span> <span class="mi">1814</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_periodpc</span><span class="p">:</span> <span class="mi">1814</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_monocle_ica</span><span class="p">:</span> <span class="mi">1813</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_wanderlust</span><span class="p">:</span> <span class="mi">1809</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_phenopath</span><span class="p">:</span> <span class="mi">1806</span>
<span class="n">truatpasteurdotfr</span><span class="o">/</span><span class="n">singularity</span><span class="o">-</span><span class="n">docker</span><span class="o">-</span><span class="n">miniconda3</span><span class="p">:</span> <span class="mi">1558</span>
<span class="n">nickjer</span><span class="o">/</span><span class="n">singularity</span><span class="o">-</span><span class="n">r</span><span class="p">:</span> <span class="mi">1553</span>
<span class="n">FCP</span><span class="o">-</span><span class="n">INDI</span><span class="o">/</span><span class="n">C</span><span class="o">-</span><span class="n">PAC</span><span class="p">:</span> <span class="mi">1532</span>
<span class="n">ISU</span><span class="o">-</span><span class="n">HPC</span><span class="o">/</span><span class="n">mysql</span><span class="p">:</span> <span class="mi">1469</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_slicer</span><span class="p">:</span> <span class="mi">1426</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_elpilinear</span><span class="p">:</span> <span class="mi">1411</span>
<span class="n">adam2392</span><span class="o">/</span><span class="n">deeplearning_hubs</span><span class="p">:</span> <span class="mi">1328</span>
<span class="n">singularityhub</span><span class="o">/</span><span class="n">hello</span><span class="o">-</span><span class="n">world</span><span class="p">:</span> <span class="mi">1307</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_wishbone</span><span class="p">:</span> <span class="mi">1296</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_fateid</span><span class="p">:</span> <span class="mi">1256</span>
<span class="n">Characterisation</span><span class="o">-</span><span class="n">Virtual</span><span class="o">-</span><span class="n">Laboratory</span><span class="o">/</span><span class="n">CharacterisationVL</span><span class="o">-</span><span class="n">Software</span><span class="p">:</span> <span class="mi">1249</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">dynwrap</span><span class="p">:</span> <span class="mi">1248</span>
<span class="n">brucemoran</span><span class="o">/</span><span class="n">Singularity</span><span class="p">:</span> <span class="mi">1233</span>
<span class="n">DavidEWarrenPhD</span><span class="o">/</span><span class="n">afnipype</span><span class="p">:</span> <span class="mi">1188</span>
<span class="n">onuryukselen</span><span class="o">/</span><span class="n">initialrun</span><span class="p">:</span> <span class="mi">1163</span>
<span class="n">as1986</span><span class="o">/</span><span class="n">own</span><span class="o">-</span><span class="n">singularity</span><span class="p">:</span> <span class="mi">1161</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_projected_monocle</span><span class="p">:</span> <span class="mi">1107</span>
<span class="n">bballew</span><span class="o">/</span><span class="n">NGS_singularity_recipes</span><span class="p">:</span> <span class="mi">1072</span>
<span class="n">ReproNim</span><span class="o">/</span><span class="n">containers</span><span class="p">:</span> <span class="mi">1012</span>
<span class="n">TomHarrop</span><span class="o">/</span><span class="n">singularity</span><span class="o">-</span><span class="n">containers</span><span class="p">:</span> <span class="mi">1006</span>
<span class="n">cokelaer</span><span class="o">/</span><span class="n">graphviz4all</span><span class="p">:</span> <span class="mi">957</span>
<span class="n">UMMS</span><span class="o">-</span><span class="n">Biocore</span><span class="o">/</span><span class="n">initialrun</span><span class="p">:</span> <span class="mi">874</span>
<span class="n">myyoda</span><span class="o">/</span><span class="n">ohbm2018</span><span class="o">-</span><span class="n">training</span><span class="p">:</span> <span class="mi">864</span>
<span class="n">chrarnold</span><span class="o">/</span><span class="n">Singularity_images</span><span class="p">:</span> <span class="mi">805</span>
<span class="n">bsiranosian</span><span class="o">/</span><span class="n">bens_1337_workflows</span><span class="p">:</span> <span class="mi">782</span>
<span class="n">justinblaber</span><span class="o">/</span><span class="n">synb0_25iso_app</span><span class="p">:</span> <span class="mi">767</span>
<span class="n">dynverse</span><span class="o">/</span><span class="n">ti_mfa</span><span class="p">:</span> <span class="mi">751</span>
<span class="n">pegasus</span><span class="o">-</span><span class="n">isi</span><span class="o">/</span><span class="n">fedora</span><span class="o">-</span><span class="n">montage</span><span class="p">:</span> <span class="mi">669</span>
<span class="n">alfpark</span><span class="o">/</span><span class="n">linpack</span><span class="p">:</span> <span class="mi">641</span>
<span class="n">cottersci</span><span class="o">/</span><span class="n">DIRT2_Workflows</span><span class="p">:</span> <span class="mi">641</span>
<span class="n">linzhi2013</span><span class="o">/</span><span class="n">MitoZ</span><span class="p">:</span> <span class="mi">613</span>
<span class="n">lar</span><span class="o">-</span><span class="n">airpact</span><span class="o">/</span><span class="n">bluesky</span><span class="o">-</span><span class="n">framework</span><span class="p">:</span> <span class="mi">589</span>
<span class="n">apeltzer</span><span class="o">/</span><span class="n">EAGER</span><span class="o">-</span><span class="n">GUI</span><span class="p">:</span> <span class="mi">589</span>
<span class="n">datalad</span><span class="o">/</span><span class="n">datalad</span><span class="o">-</span><span class="n">extensions</span><span class="p">:</span> <span class="mi">586</span>
<span class="n">maxemil</span><span class="o">/</span><span class="n">PhyloMagnet</span><span class="p">:</span> <span class="mi">564</span>
<span class="n">elimoss</span><span class="o">/</span><span class="n">lathe</span><span class="p">:</span> <span class="mi">562</span>
<span class="n">mwiens91</span><span class="o">/</span><span class="n">test</span><span class="o">-</span><span class="n">error</span><span class="o">-</span><span class="n">containers</span><span class="p">:</span> <span class="mi">555</span>
<span class="n">ISU</span><span class="o">-</span><span class="n">HPC</span><span class="o">/</span><span class="n">ros</span><span class="p">:</span> <span class="mi">551</span>
<span class="n">mwanakijiji</span><span class="o">/</span><span class="n">lbti_altair_fizeau</span><span class="p">:</span> <span class="mi">533</span>
<span class="n">willgpaik</span><span class="o">/</span><span class="n">centos7_aci</span><span class="p">:</span> <span class="mi">519</span>
<span class="n">jekriske</span><span class="o">/</span><span class="n">lolcow</span><span class="p">:</span> <span class="mi">517</span>
<span class="n">tyson</span><span class="o">-</span><span class="n">swetnam</span><span class="o">/</span><span class="n">osgeo</span><span class="o">-</span><span class="n">singularity</span><span class="p">:</span> <span class="mi">517</span>
<span class="n">ltalirz</span><span class="o">/</span><span class="n">singularity</span><span class="o">-</span><span class="n">recipe</span><span class="o">-</span><span class="n">zeopp</span><span class="p">:</span> <span class="mi">512</span>
</code></pre></div></div>

<p>There is a much longer list that has just about as many lines as collections,
but we’ll leave it at that for now. Thank you to the community for your excitement
about Singularity, and using Singularity Hub! I can’t make any promises, but maybe
keep on the lookout for other ideas and developments in the next few months.
I certainly hope to contribute more to the world of research software than this
small amount of early work.</p>