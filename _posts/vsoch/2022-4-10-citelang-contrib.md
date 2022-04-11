---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2022-04-10 12:30:00'
layout: post
original_url: https://vsoch.github.io//2022/citelang-contrib/
title: Open Source Contributions with CiteLang
---

<p>I recently contributed to a project, and a maintainer pinged me apologizing that it didn’t
show up in the Twitter preview for the changeset. The problem was that it showed him
as the committer because it was cherry picked.</p>

<p>Now reader, I had already been graciously thanked and felt appreciated for the contribution.
But he brought up a good point - sometimes with large git histories, or if the final product is
squashed and merged or cherry picked, we can lose sight of our contributors. This I think
is a bad thing, because as a maintainer I want to champion my contributors. We should have an automated
way to quickly say:</p>

<blockquote>
  <p>Show me the contribution summary for this repository between the last release and this one!</p>
</blockquote>

<p>So this was (one of) my fun projects to tackle this weekend!</p>

<h2 id="citelang">CiteLang</h2>

<p>To sidetrack for a moment, one of my personal projects, a library called 
<a href="https://vsoch.github.io/citelang/getting_started/user-guide.html" target="_blank">CiteLang</a> is
aiming to make an effort to redefine credit for software. Now, that’s another post in and of itself, but I’ll briefly summarize
for those interested. If you aren’t interested, jump down to <a href="https://vsoch.github.io/usrse-feed.xml#citelang-contrib">the contrib section</a>.
CiteLang takes the approach that trying to generate some digital object identifier (DOI) is
too much extra work to reasonably do. I mean seriously, do we really need to follow a dated academic practice that
champions a unit of worth as the publication when we know that process is broken? We need to move
away from this traditional “be valued like an academic” model. I am troubled that I don’t see anyone
really going for this. All I see is our community trying to write more papers about citation? That is very ironic.
So although I can’t change it overnight, I can at least try and introduce new ideas, and put my money
where my mouth is by implementing them.</p>

<p>This is why I created CiteLang. CiteLang takes the approach that the unit of “publication” for a piece of software is 
when it’s added to a package manager, meaning that anyone else can use it within that manager’s domain, and “publishing”
or getting credit means doing what you would or even should be doing for software anyway. So what CiteLang basically
does is provide a client and Python functions for taking some package from a manager, and generating credit summaries
in markdown, or even badges for a repository. Here is the badge for CiteLang itself. We give some credit split (50%) to the
library itself, and the other half goes to the dependencies, and this pattern continues to their dependencies up to some
minimum credit threshold. It makes a nice sunburst:</p>

<div style="padding: 20px;">
  <img src="https://raw.githubusercontent.com/vsoch/citelang/main/docs/assets/img/pypi-citelang.png" />
</div>

<p>I haven’t written it up formally yet because I’m still working on it quite a bit, but I suspect I will at some point!</p>

<h3 id="citelang-contrib">CiteLang Contrib</h3>



<p>A “contrib” command for CiteLang made sense because it’s a different level of credit attribution. Instead of looking
at the package manager metadata, we instead want to look at the git history. However, we want to look into the deepest level 
of contribution - the actual contributions for commits on a line to line bases! This means that we can use git blame.
What a git blame can show us, for some point and time (commit) and file, is the last person that contributed each line.
This means that if you look at a file over time (commits) you’ll see duplication if the line has not changed,
and we need to be careful of this if we are counting contributions. So first, let’s run citelang on itself! 
Here we can see detailed contributions, by author, between two tagged releases:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>citelang contrib <span class="nt">--start</span> 0.0.18 <span class="nt">--end</span> 0.0.19 <span class="nt">--detail</span>
</code></pre></div></div>

<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/citelang/run1.png" />
</div>

<p>What you see above is a (truncated) listing of files, authors, and lines changed. It’s truncated
for display in the table. If you wanted to save this data to an output file, you can get the complete story:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>citelang contrib <span class="nt">--start</span> 0.0.18 <span class="nt">--end</span> 0.0.19 <span class="nt">--detail</span> <span class="nt">--outfile</span> citelang-18-to-19.json
</code></pre></div></div>
<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/citelang/run2.png" />
</div>

<p>That also shows that sometimes I commit as vsoch, and sometimes Vanessasaurus. :_)
If I make an alias lookup file, I can tell CiteLang that “Vanessasaurus” is in fact “vsoch”:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>citelang contrib <span class="nt">--start</span> 0.0.18 <span class="nt">--end</span> 0.0.19 <span class="nt">--detail</span> <span class="nt">--filters</span> citelang-authors.yaml
</code></pre></div></div>

<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/citelang/run11.png" />
</div>

<p>And this author duplication is fixed!</p>

<h4 id="contrib-for-singularity">Contrib for Singularity</h4>

<p>It’s more interesting for a larger project like Singularity! Here is the same thing, but 
for an old version of Singularity, between 2.3 and 2.3.1. Note that I’m sitting in the
root of the repository when I run this:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>citelang contrib <span class="nt">--start</span> 2.2.1 <span class="nt">--end</span> 2.3
</code></pre></div></div>
<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/citelang/run3.png" />
</div>

<p>or with detail:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>citelang contrib <span class="nt">--start</span> 2.2.1 <span class="nt">--end</span> 2.3 <span class="nt">--detail</span>
</code></pre></div></div>
<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/citelang/run4.png" />
</div>

<p>Again, the detailed view truncates fairly arbitrarily, so it’s just a shapshot. We
could change this at some point but the table could get VERY large. 
Here is an example of a range that had cherry picking where (if you look superficially at the release) you miss
the contributors:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>citelang contrib <span class="nt">--start</span> v3.9.7 <span class="nt">--end</span> v3.9.8
</code></pre></div></div>
<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/citelang/run5.png" />
</div>

<h4 id="include-all-blames">Include All Blames</h4>

<p>We also might say “Hey, show me that range of commits but include credit for all time, meaning blames (commits) that
are still present in the time we are interested in, but perhaps were created before it.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>citelang contrib <span class="nt">--start</span> v3.9.7 <span class="nt">--end</span> v3.9.8 <span class="nt">--all-time</span> <span class="nt">--filters</span> authors.yaml
</code></pre></div></div>
<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/citelang/run6.png" />
</div>

<p>This barely fit on my screen, but it did! And actually, I again used that <code class="language-plaintext highlighter-rouge">--filters</code> flag
so we can associate a specific name with one author alias. It was when I was looking at Singularity
that I realized that I needed this filtering functionality, because I wanted to remove bots,
match aliases, and ignore some random commit by “root” (although I think I know who that was!)
Here is what that filters file looks like:</p>

<div class="language-yaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="na">authors</span><span class="pi">:</span>
   <span class="na">Vanessa Sochat</span><span class="pi">:</span> <span class="s">vsoch</span>
   <span class="na">Cedric Clerget</span><span class="pi">:</span> <span class="s">Cédric Clerget</span>
   <span class="na">cclerget</span><span class="pi">:</span> <span class="s">Cédric Clerget</span>
   <span class="na">ArangoGutierrez</span><span class="pi">:</span> <span class="s">Eduardo Arango</span>
   <span class="s">Marcelo E. Magallon</span><span class="pi">:</span> <span class="s">Marcelo Magallon</span>
   <span class="na">Ian</span><span class="pi">:</span> <span class="s">Ian Kaneshiro</span>
   <span class="na">sashayakovtseva</span><span class="pi">:</span> <span class="s">Sasha Yakovtseva</span>
   <span class="na">jscook2345</span><span class="pi">:</span> <span class="s">Justin Cook</span>
   <span class="na">GodloveD</span><span class="pi">:</span> <span class="s">David Godlove</span>
   <span class="na">Dave Trudgian</span><span class="pi">:</span> <span class="s">David Trudgian</span>
   <span class="na">Gregory</span><span class="pi">:</span> <span class="s">Gregory M. Kurtzer</span>
 <span class="na">ignore_bots</span><span class="pi">:</span> <span class="no">true</span>
 <span class="na">ignore_users</span><span class="pi">:</span>
   <span class="pi">-</span> <span class="s">root</span>
</code></pre></div></div>

<p><strong>Update</strong> citelang also now has support for <code class="language-plaintext highlighter-rouge">ignore_basename</code> and <code class="language-plaintext highlighter-rouge">ignore_files</code> to not parse
specific files, such as go.mod and go.sum that are automatically derived and should not count toward credit!
With this change we also tweaked the parsing to not count empty lines, since no code is actually written there!
Note that you could also use this alias functionality to assign people to groups (companies,
organizations, labs, etc.) as a different way to summarize. And special shoutout to the main group 
of guys at the top (yes it is all men, I can say guys) before my GitHub username that have been the 
powerhouse behind Singularity.</p>

<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/citelang/run61.png" />
</div>

<p>I see many names from Sylabs, and even before. It’s surprising that I 
have that many lines of blame still somewhere given that I largely stopped working on the project at 
the end of 2017. Cedric and Dave Trudgian are superheros!</p>

<h4 id="summarize-by-path">Summarize by Path</h4>

<p>We can also decide we want to summarize by the file instead of the author, for example
if you want to see the most changes files for the range based on lines changed.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>citelang contrib <span class="nt">--start</span> v3.9.7 <span class="nt">--end</span> v3.9.8 <span class="nt">--by</span> path
</code></pre></div></div>
<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/citelang/run7.png" />
</div>

<p>…and add details (in this case the author and not the path, and again truncated for display).</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>citelang contrib <span class="nt">--start</span> v3.9.7 <span class="nt">--end</span> v3.9.8 <span class="nt">--by</span> path <span class="nt">--detail</span>
</code></pre></div></div>
<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/citelang/run8.png" />
</div>

<p>And that’s the basic functionality! I’ve created both this interactive “quickly see”
and export data, along with a GitHub action that you can use to generate the data.
I’m planning (in a future evening or weekend) to make a more structured GitHub action
paired with an interface to automatically display contributions. Mind you I just threw this
together today so likely I’ll work more on it.</p>

<h3 id="how-does-it-work">How does it work?</h3>

<p>I decided early on I wanted to use git blame - the reason being that we want to get
the original, fine-grained contributions. We also want to take account for the amount
of work that a person puts into a contribution. It’s not to say that more code is always better,
but if the practice is always to squash and merge, a single commit isn’t the right granularity to measure
because it could be one or 1000 lines.</p>

<p>This means that deriving credit is a bit slower because we have to run git blame across
files and commits, but with multiprocessing I was able to get ranges of commits to run in a reasonable
amount of time. What you’d likely want to do for a huge repository that you want to study over time
is save the cache of blame results, and then incrementally grow it with each release. I will
likely provide this when I provide some kind of action or UI in the future.</p>

<h4 id="git-blame">git blame</h4>

<p>But I digress! So first let’s talk about git blame. When we run git blame on a file we get the last commit and timestamp that every line
was modified. Here is an example, again with Singularity:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>git blame <span class="nt">-w</span> <span class="nt">-M</span> <span class="nt">-C</span> <span class="nt">--line-porcelain</span> e214d4ebf0a1274b1c63b095fd55ae61c7e92947 <span class="nt">--</span> libexec/cli/inspect.exec
</code></pre></div></div>
<div style="padding: 20px;">
  <img src="https://vsoch.github.io/assets/images/posts/citelang/run9.png" />
</div>

<p>There you see two lines under the same commit that were changed, and we count each one.
Note that the above commit can be derived from a tag name as follows (and this is what the library does):</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>git rev-list <span class="nt">-n</span> 1 2.3.1
</code></pre></div></div>

<p>From the above entry we save:</p>

<ol class="custom-counter">
<li>The author name</li>
<li>The path</li>
<li>The text</li>
<li>The commit</li>
<li>The timestamp</li>
</ol>

<p>The timestamp is especially important because by default we want to honor the request to see
a particular range, and don’t include git blame commits that happened before or after it.
You can disable this with the <code class="language-plaintext highlighter-rouge">--all-time</code> flag. The primary use case, as you might imagine,
is that you are maintainer releasing a new version of your software and you want to give a manual
or programmatic shout-out to the contributors between the last release and this new one.
So the way that contrib works is to:</p>

<ol class="custom-counter">
<li>Take a start and end tag or commit to parse</li>
<li>Find the range of commits relevant to that range</li>
<li>For each commit, find all relevant changed files </li>
<li>Run git blame for the commit and file, and save a cached result (the fields above)</li>
<li>Each multiprocessing worker returns the complete blame result</li>
<li>We combine results across workers</li>
<li>We then filter down to timestamps within the range (if not all time)</li>
<li>We organized based on author or path, and with or without detail</li>
<li>The result is printed or saved to the filesystem as json</li>
</ol>

<p>For the fourth step, we run jobs with multiprocessing, and we store the result in the <code class="language-plaintext highlighter-rouge">.contrib</code> 
cache, which is organized equivalently to the repository. This means that future workers can
immediately find and load the file instead of running git blame (and parsing that result) again.
For example, here is Singularity after a run, and we can see there are files from both the 
golang history and the previous bash/C++ and python history:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span><span class="nb">ls</span> .contrib/cache/
alpinelinux  bin                 configure.ac     debian  examples  INSTALL.md  ...
AUTHORS      CHANGELOG.md        CONTRIBUTING.md  docs    go.mod    internal    ...
AUTHORS.md   cmd                 COPYING          e2e     go.sum    libexec     ...
autogen.sh   CODE_OF_CONDUCT.md  COPYRIGHT.md     etc     INSTALL   LICENSE     ...
</code></pre></div></div>

<p>This happened because I ran contrib for an older version range and a newer version range.
If we look in any particular file “directory” we find json files named by commit that had blame run.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span><span class="nb">ls</span> .contrib/cache/libexec/cli/inspect.help/
045edd5c94822ae4f46c918399249f0dda38b6fc.json  83f6e979c533d09e0b5474fb40508238bb673a11.json
082ef81f751267b475bb918e7d1ca415567ebfe2.json  8f62c342009686990da5f0925d7f815edd110801.json
</code></pre></div></div>

<p>We can look in each json file and see the parsed blame:</p>

<div class="language-json highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">{</span><span class="w">
    </span><span class="nl">"Vanessa Sochat"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"libexec/cli/inspect.exec"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="nl">"be6c7a85391e79e1273acc659410fbd17ced6d5f"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
                </span><span class="nl">"1495390269"</span><span class="p">:</span><span class="w"> </span><span class="mi">28</span><span class="w">
            </span><span class="p">},</span><span class="w">
            </span><span class="nl">"cf8bbc36e1a97be04e779c5d0ab2f9a4a407cfdb"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
                </span><span class="nl">"1495597759"</span><span class="p">:</span><span class="w"> </span><span class="mi">5</span><span class="w">
            </span><span class="p">}</span><span class="w">
        </span><span class="p">}</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"Gregory M. Kurtzer"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"libexec/cli/inspect.exec"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="nl">"3a9f9cf1e6369080c4676a8919e2cb09d8116e6e"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
                </span><span class="nl">"1496258562"</span><span class="p">:</span><span class="w"> </span><span class="mi">2</span><span class="w">
            </span><span class="p">},</span><span class="w">
            </span><span class="nl">"2ad9d5862b7d1cea937044a23997bf66cd06695c"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
                </span><span class="nl">"1483468664"</span><span class="p">:</span><span class="w"> </span><span class="mi">2</span><span class="w">
            </span><span class="p">},</span><span class="w">
</span><span class="err">...</span><span class="w">
</span></code></pre></div></div>

<p>What we see above are authors, file paths, and commits, timestamps, and then
changed lines for each timestamp. To hammer down the point, the reason we have different commits for one
blame (run for a commit) is because this represents the git blame for every line
of the file at that timepoint. A change from a previous commit, even if it’s outside the commit,
if it’s still present is going to show up! This is also why we keep the timestamps, because we can
then filter this set down to include only commits that were within a desired range.
But it also gives us the ability to see “all time credit” at any particular timestamp,
which is pretty cool!</p>

<p>Once we parse a blame for a specific commit and file, we won’t need to parse
it again as it will be cached in (default) <code class="language-plaintext highlighter-rouge">.contrib</code> in the present working
directory. We can load that file and filter it for different contexts.
In the root of that same directory we also have a record of previously run ranges, which
are cached so we don’t have to load the individual files and filter them again. We also keep caches of other
information to help the multiprocessing workers, but we don’t need the details.</p>

<p>That’s it! This was a super fun little weekend project, and I suspect I’ll have more
in the coming weeks related to it.</p>