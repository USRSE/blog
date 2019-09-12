---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-09-10 05:22:00'
layout: post
original_url: https://vsoch.github.io//2019/resource-explorer/
title: The Resource Explorer
---

<p>I was having a chat with the amazing, the diabolical <a href="https://github.com/griznog" target="_blank">griznog</a>, and he had an idea up his sleeve for some kind of online form where a user could make selections, and filter down to some set of resources. I loved the idea, and decided to give it a shot.</p>

<blockquote>
  <p>Insert struggles with JavaScript frameworks…</p>
</blockquote>

<p>This resulted in <strong><em>drumroll</em></strong>…</p>

<p><br />
<a href="https://vsoch.github.io/resource-explorer">
<img src="https://raw.githubusercontent.com/vsoch/resource-explorer/master/img/resource-explorer.png" />
</a>
<br /></p>

<p>The resource explorer!!</p>

<h2 id="show-me">Show me!</h2>

<p>If you don’t want to read about it, just <a href="https://vsoch.github.io/resource-explorer" target="_blank">see it here</a> or
<a href="https://www.github.com/vsoch/resource-explorer" target="_blank">use the code</a> for your own group.</p>

<h2 id="what-is-this-thing">What is this thing?</h2>

<p>It’s really quite simple - it allows for interactive selection of a storage, compute, service,
or other resource provided by your group. In the implementation linked here, we use resources and services relevant for research computing. I want to note that while I plan to deploy a version of
this for my group, <strong>this particular instance is for demonstration purposes only and does not fully represent
what we offer!</strong> I need the help of my group to finish up the details, and we will be deploying
a “for realsies” resource explorer likely at <a href="https://stanford-rc.github.io">our documentation portal</a>. For now, let’s walk through how it works.</p>

<h3 id="resources-and-questions">Resources and Questions</h3>

<p>The <a href="https://github.com/vsoch/resource-explorer/blob/master/resource-explorer.js">resource-explorer.js</a> 
file contains both the script and the data structures that drive it. At the top, you’ll see
variables for questions and resources that drive the logic of the interface. The user will be presented
with a series of easy to answer questions, and the selection of resources
will be narrowed down based on the answers. Specifically:</p>

<h4 id="questions">Questions</h4>

<p>Include multiple-choice, single-choice, boolean, and enumerate (maximum or minumum) choices.</p>

<ol class="custom-counter">
 <li> <strong>multiple-choice</strong> means that the user can select zero through N choices. For example, I might want to select that a resource is for staff, faculty, and students.</li>
 <li> <strong>single-choice</strong>: is a choice question where the user is only allowed to select one answer. If you need to implement a boolean, use a single-choice with two options.</li>
 <li> <strong>minimum-choice</strong>: indicates a single choice field where the choices have integer values, and the user is selecting a minimum. For example, if the user selects a minimum storage or memory size, all choices above that will remain.</li>
 <li> <strong>maximum-choice</strong>: is equivalent to minimum-choice, but opposite in direction. We select a maximum.</li>
</ol>

<p>In the above, an enumerate choice (minimum or maximum) implies there is an ordering to the logic. The user might want a minimum or maximum amount of memory, for example. Each question should be under the questions variable (a list), and have a title, description, required (true or false) and then options. For example, here is a question from the list:</p>

<div class="language-json highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="w">      </span><span class="p">{</span><span class="w">
         </span><span class="s2">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Who is the resource for?"</span><span class="p">,</span><span class="w">
         </span><span class="s2">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"q-who"</span><span class="p">,</span><span class="w">
         </span><span class="s2">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Select one or more groups that the resource is needed for."</span><span class="p">,</span><span class="w">
         </span><span class="s2">"required"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
         </span><span class="s2">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"multiple-choice"</span><span class="p">,</span><span class="w">
         </span><span class="s2">"options"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
            </span><span class="p">{</span><span class="w">
               </span><span class="s2">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"faculty"</span><span class="p">,</span><span class="w">
               </span><span class="s2">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"who-faculty"</span><span class="w">
            </span><span class="p">},</span><span class="w">
            </span><span class="p">{</span><span class="w">
               </span><span class="s2">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"staff"</span><span class="p">,</span><span class="w">
               </span><span class="s2">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"who-staff"</span><span class="w">
            </span><span class="p">},</span><span class="w">
            </span><span class="p">{</span><span class="w">
               </span><span class="s2">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"student"</span><span class="p">,</span><span class="w">
               </span><span class="s2">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"who-student"</span><span class="w">
            </span><span class="p">}</span><span class="w">
         </span><span class="p">]</span><span class="w">
      </span><span class="p">}</span><span class="w">
</span></code></pre></div></div>

<p>For a minimum-* or maximum-* choice, the ids must end in an integer value:</p>

<div class="language-json highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="w">      </span><span class="p">{</span><span class="w">
         </span><span class="s2">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"What size of storage are you looking for?"</span><span class="p">,</span><span class="w">
         </span><span class="s2">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"q-size"</span><span class="p">,</span><span class="w">
         </span><span class="s2">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"If applicable, give an approximate unit of storage."</span><span class="p">,</span><span class="w">
         </span><span class="s2">"required"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
         </span><span class="s2">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"minimum-choice"</span><span class="p">,</span><span class="w">
         </span><span class="s2">"options"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
            </span><span class="p">{</span><span class="w">
               </span><span class="s2">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"gigabytes"</span><span class="p">,</span><span class="w">
               </span><span class="s2">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"size-gigabytes-1"</span><span class="w">
            </span><span class="p">},</span><span class="w">
            </span><span class="p">{</span><span class="w">
               </span><span class="s2">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"terabytes"</span><span class="p">,</span><span class="w">
               </span><span class="s2">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"size-terabytes-2"</span><span class="w">
            </span><span class="p">},</span><span class="w">
            </span><span class="p">{</span><span class="w">
               </span><span class="s2">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"petabytes"</span><span class="p">,</span><span class="w">
               </span><span class="s2">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"size-petabytes-3"</span><span class="w">
            </span><span class="p">}</span><span class="w">
         </span><span class="p">]</span><span class="w">
      </span><span class="p">},</span><span class="w">
</span></code></pre></div></div>

<p>We do this so we can parse the ids and then rank order them. For a boolean choice, you can just use a single-choice
with two choices:</p>

<div class="language-json highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="w">      </span><span class="p">{</span><span class="w">
         </span><span class="s2">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Do you require backup?"</span><span class="p">,</span><span class="w">
         </span><span class="s2">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"q-backups"</span><span class="p">,</span><span class="w">
         </span><span class="s2">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Some or all of your files will be copied on a regular basis in case you need restore."</span><span class="p">,</span><span class="w">
         </span><span class="s2">"required"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
         </span><span class="s2">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"single-choice"</span><span class="p">,</span><span class="w">
         </span><span class="s2">"options"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
            </span><span class="p">{</span><span class="w">
               </span><span class="s2">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"backups"</span><span class="p">,</span><span class="w">
               </span><span class="s2">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"backups-true"</span><span class="w">
            </span><span class="p">},</span><span class="w">
            </span><span class="p">{</span><span class="w">
               </span><span class="s2">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"no backups"</span><span class="p">,</span><span class="w">
               </span><span class="s2">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"backups-false"</span><span class="w">
            </span><span class="p">}</span><span class="w">
         </span><span class="p">]</span><span class="w">
      </span><span class="p">},</span><span class="w">
</span></code></pre></div></div>

<p>Notice that each choice has a unique id associated with it. These will be used as tags associated with each
resource to help with the filtering.</p>

<h4 id="resources">Resources</h4>

<p>A typical resource looks like this:</p>

<div class="language-json highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="w">      </span><span class="p">{</span><span class="w">
         </span><span class="s2">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Nero"</span><span class="p">,</span><span class="w">
         </span><span class="s2">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"nero"</span><span class="p">,</span><span class="w">
         </span><span class="s2">"url"</span><span class="p">:</span><span class="w"> </span><span class="s2">"https://nero-docs.stanford.edu"</span><span class="p">,</span><span class="w">
         </span><span class="s2">"attributes"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
           </span><span class="s2">"q-kind"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"kind-compute"</span><span class="p">,</span><span class="w"> </span><span class="s2">"kind-cloud"</span><span class="p">],</span><span class="w">
           </span><span class="s2">"q-service"</span><span class="p">:</span><span class="w"> </span><span class="p">[],</span><span class="w">
           </span><span class="s2">"q-who"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"who-faculty"</span><span class="p">],</span><span class="w"> </span><span class="err">//</span><span class="w"> </span><span class="err">only</span><span class="w"> </span><span class="err">faculty</span><span class="w"> </span><span class="err">allowed</span><span class="w">
                                     </span><span class="err">//</span><span class="w"> </span><span class="err">domain</span><span class="w"> </span><span class="err">is</span><span class="w"> </span><span class="err">left</span><span class="w"> </span><span class="err">out</span><span class="p">,</span><span class="w"> </span><span class="err">implying</span><span class="w"> </span><span class="err">all</span><span class="w"> </span><span class="err">domains</span><span class="w">
                                     </span><span class="err">//</span><span class="w"> </span><span class="err">size</span><span class="w"> </span><span class="err">is</span><span class="w"> </span><span class="err">left</span><span class="w"> </span><span class="err">out</span><span class="p">,</span><span class="w"> </span><span class="err">implying</span><span class="w"> </span><span class="err">all</span><span class="w"> </span><span class="err">sizes</span><span class="w">
           </span><span class="s2">"q-framework"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"framework-kubernetes"</span><span class="p">,</span><span class="w"> </span><span class="s2">"framework-containers"</span><span class="p">],</span><span class="w">
           </span><span class="s2">"q-backups"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"backups-true"</span><span class="p">]</span><span class="w">
         </span><span class="p">}</span><span class="w">
      </span><span class="p">},</span><span class="w">
</span></code></pre></div></div>

<p>Note that we require a title (a user friendly print-able value), an id (for the object in the DOM), a url to
link to, and then a list of attributes. For each attribute, the key corresponds to an id for a question,
and then the list of values is a list of responses that, if chosen by the user, would make 
it a valid choice. Specifically:</p>

<h5 id="general-logic">General Logic</h5>

<ul>
  <li>if a question key is defined but empty, a selection of any field for the key invalidates the resource.</li>
  <li>if a question key is not defined, making a choice for any field for doesn’t impact the resource (it stays or remains hidden).</li>
  <li>if a question key includes a subset of choices, then the resource is kept only if the user chooses a selection in the list.</li>
</ul>

<h5 id="example-logic">Example Logic</h5>

<p>For the above, this would mean that:</p>

<ul>
  <li>if the user selected any kind of “q-service”, the Nero option would be hidden.</li>
  <li>if the user selected kind-compute and/or kind-cloud, Nero would remain.</li>
  <li>if the user selected any other kind of audience other than faculty (who-faculty) Nero would be hidden.</li>
  <li>Selecting framework as kubernetnes or containers will keep Nero. Selecting slurm (not in the list) will hide it.</li>
  <li>Selecting backups-false will hide Nero.</li>
</ul>

<p><br /></p>

<p>The <a href="https://github.com/vsoch/resource-explorer">repository</a> serves only as an example, and doesn’t reflect the actual state of Stanford resources (but stay tuned, we will make one, as soon as I can get feedback from my colleages!)</p>