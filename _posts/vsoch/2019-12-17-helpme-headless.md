---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-12-17 12:37:00'
layout: post
original_url: https://vsoch.github.io//2019/helpme-headless/
title: Automated GitHub Support for Python Packages
---

<p>Hey research software engineers! How often might a user trigger a bug
in your Python software and not tell you about it? Or how often is a bug triggered
but not enough information is provided about it?</p>

<h3 id="who-is-responsible">Who is Responsible?</h3>

<p>We shouldn’t place the burden on collecting and submission of information on the user -
they already have their own research and domain of knowledge to worry about. 
Instead, we should make it really easy for them. We should make it so that, when they
are using our software and trigger a bug, we do the heavy lifting to report it.</p>

<h2 id="remember-helpme">Remember HelpMe?</h2>

<p>A few years back I created the <a href="https://vsoch.github.io/helpme" target="_blank">HelpMe</a>
software, which is a command line tool that would help a user to interactively submit a bug report.
It might look like this:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">helpme</span> <span class="n">github</span> <span class="n">rseng</span><span class="o">/</span><span class="n">github</span><span class="o">-</span><span class="n">support</span>
</code></pre></div></div>

<p>The above line would collect a title, description, and optionally environment and asciinema
screen recording to post an issue at <a href="https://github.com/rseng/github-support" target="_blank">rseng/github-support</a>.
HelpMe also supports discourse, uservoice, and any other client that you might want that has an API to post to.
But it stopped there - the issue would be opened and that’s it. But what if we could automate it, 
and (even better) create a database of issues? A discussion was picked up on 
the <a href="https://github.com/datalad/datalad/issues/3649" target="_blank">datalad</a>
issue board around this exact idea.</p>

<h2 id="what-do-we-need">What do We Need?</h2>

<p>We need a <strong>version controlled</strong> and <strong>automated</strong> issue reporting system that can either
create the entire issue on behalf of the user (given a GitHub token), or a more interative version
that opens up a pre-populated issue report and allows the user to provide additional context.
Specifically, I would want the following:</p>

<ol class="custom-counter">
<li>the user triggers a bug</li>
<li>the traceback of the error is automatically collected, and a GitHub issue opened with context</li>
<li>the user can preview and add additional detail to the issue</li>
<li>the issue is assigned an identifier based on a hash of some metadata you've provided</li>
<li>a GitHub workflow will direct the user to another issue (if was previously reported) and close the current issue</li>
<li>the metadata about the error is pushed to a support repository that serves as a flat file database.</li>
</ol>

<p>And then the errors database can be used to derive insights about your software.</p>

<h2 id="an-example">An Example</h2>

<p>To drive this functionality, I’ve published version 0.0.40 of helpme that provides 
a headless submission workflow, and created an example repository <a href="https://github.com/rseng/github-support" target="_blank">rseng/github-support</a> that uses it. A side-note: this new organization
 <a href="https://github.com/rseng" target="_blank">rseng</a> is intended to be an open community of research
software engineers that want to collaborate on fun projects (such as this one!) so if you’d like to join,
please let me know and I’ll send an invite!</p>

<h3 id="rsenggithub-support">rseng/github-support</h3>

<p>This post introduces the idea of a</p>

<blockquote>
  <p>GitHub support repository</p>
</blockquote>

<p>A GitHub support repository is one that solely exists for providing user support for some software.
The content of the repository is a database of error reports, organized and updated by identifiers that
can uniquely identify the issues. 
Within the repository, an <a href="https://github.com/rseng/github-support/blob/master/example.py" target="_blank">example.py</a> 
script shows how the issue is generated and submit, along with a <a href="https://github.com/rseng/github-support/blob/master/.github/workflows/report_issue.yml" target="_blank">workflow</a> that will trigger on newly opened issues, parse the issue body,
and then determine based on the identifier and current issues database if the issue has already been opened. In the case
that the issue was previously opened, the body is still saved, but a comment is added to the newly opened issue to alert
the user, link to the previous issue, and then the issue is closed.</p>

<div style="padding: 20px;">
    <img src="https://vsoch.github.io/assets/images/posts/helpme/closed-issue.png" />
</div>

<p>If you look at the repository itself, issues are organized under an “issues” folder,
and then an identifier subfolder, which is derived from the exception thrown and message.</p>

<hr />

<h3 id="how-does-it-work">How does it work?</h3>

<p>It comes down to importing the helper, and then running the function when you want
the issue to be reported, and including a meaningful identifier, title, and body.
In the example below, we don’t require a GitHub token, so the user will be
given a url to open to (or if on a desktop, the window will be opened). 
We turn off confirmation of the whitelisted environment variables since
we know the defaults don’t include anything secret.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="kn">from</span> <span class="nn">helpme.main</span> <span class="kn">import</span> <span class="n">get_helper</span>
<span class="n">helper</span> <span class="o">=</span> <span class="n">get_helper</span><span class="p">(</span><span class="s">"github"</span><span class="p">,</span> <span class="n">confirm</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
<span class="n">helper</span><span class="o">.</span><span class="n">run_headless</span><span class="p">(</span>
    <span class="n">identifier</span><span class="o">=</span><span class="n">identifier</span><span class="p">,</span>
    <span class="n">title</span><span class="o">=</span><span class="n">message</span><span class="p">,</span>
    <span class="n">body</span><span class="o">=</span><span class="n">body</span><span class="p">,</span>
    <span class="n">repo</span><span class="o">=</span><span class="s">"rseng/github-support"</span><span class="p">,</span>
<span class="p">)</span>
</code></pre></div></div>

<p>Not shown is placing this in the context of an error, or deriving a meaningful
identifier, title, or body. Specifically, if you look at the more substantial
<a href="https://github.com/rseng/github-support/blob/master/example.py" target="_blank">example.py</a>,
you’ll see that:</p>

<ol class="custom-counter">
<li>we triggers an import error, and create a <a href="https://vsoch.github.io/helpme/helper-github#headless" target="_blank">helpme GitHub helper</a></li>
<li>the error message and type is used for the identifier (converted to an md5 hash by helpme)</li>
<li>a browser window opens for the user to add details to the error in a GitHub issue.</li>
<li>a GitHub <a href="https://github.com/rseng/github-support/blob/master/.github/workflows/report_issue.yml" target="_blank">workflow</a> parses the new issue to add it's metadata to the database</li>
<li>the user is alerted and the issue closed if the issue has already been opened</li>
<li>the database is updated</li>
</ol>

<p>If you intend to use this helper throughout your library, it’s recommended to write a function 
as we’ve done in the example. The structure is of course up to you, and in fact, all of the steps from the metadata
included, how the identifier is derived, and how the GitHub issue is parsed, are totally
up to you. Let’s talk about some of these decisions next.</p>

<h2 id="questions-to-think-about">Questions to Think About</h2>

<h3 id="what-should-i-use-for-an-identifier">What should I use for an identifier?</h3>

<p>The level of detail that you want to include with your string identifier is up to you,
but you should be consistent as changes would change the database structure. For example,
you might be happy including the calling module relative path and the error message,
but not want to include the software version so that an error that reports the same
message and location (across versions) is considered the same. On the other hand, you
might want to add a level of nesting to folder organization of your issues database
to take versions into  account. For example:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>issues/
   0.0.1/
     md5.5fddcd18252717a7a11c494d24b16d4e/
        8.md
   0.0.2/
     md5.5fddcd18252717a7a11c494d24b16d4e
        10.md
</code></pre></div></div>

<p>The above structure would suggest that the same issue (based on the identifier) was
found across two versions of software. On the other hand, if version is not important
to you, you might have this instead:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>issues/
     md5.5fddcd18252717a7a11c494d24b16d4e
        8.md
        10.md
</code></pre></div></div>

<p>You would want to provide whatever metadata is needed by the issue parser into the helpme body,
and then have the GitHub workflow parse it to derive the desired organization.
This structure isn’t implemented or enforced by helpme, but rather provided as
a starting example.</p>

<h3 id="how-should-you-structure-the-data">How should you structure the data?</h3>

<p>A GitHub repository isn’t a real database, but we can sort of think of it as a flat file
database, and in this dummy example I chose to use markdown so that it would render
the issue content easily. In that even the largest repositories have a few thousand
issues, having a few thousand small files is very reasonable for keeping in GitHub.
You could dump into markdown anticipating some future processing
as I’ve done here, but likely you’d do better to provide structured metadata, and then parse into a data format like json
or yaml. If you do choose one of these formats, make sure to validate your generated files for
correctness in the workflow.</p>

<h3 id="how-should-you-organize-the-data">How should you organize the data?</h3>
<p>The example takes a simple approach of creating an “issues” folder at the root of the
repository, and then naming subfolders based on identifier hashes. The issues are named
according to numbers within, meaning that you can quickly parse over files (and sort)
to find the first opened issue for any particular identifier, or you can quickly 
count the files to get the number opened for the identifier. There is no reason
you need to organize your data in this fashion  - I took an approach to limit
each issue to one file to not have any “GitHub race” conditions, or two opened issues
trying to edit the same file at once that might hold some summary information.</p>

<h3 id="should-you-use-your-software-repository">Should you use your software repository?</h3>

<p>While your issues might be directed to your software repository, the issues database
doesn’t necessarily need to be, and I would recommend keeping the two separate.
This example repository uses the same repository for the content (examples, etc.) 
and database so the entire package can be cloned and easily understood. You
would basically want to update the <a href="https://vsoch.github.io/.github/parse_issue.py">parse_issue.py</a> and 
<a href="https://vsoch.github.io/.github/workflows/report_issue.yml">report_issue</a> workflow to clone and update
another repository with issues.</p>

<h3 id="how-automated-do-you-want-it">How automated do you want it?</h3>

<p>Depending on how you instantiate the helper, you can require the user to export a HELPME_GITHUB_TOKEN</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">helper</span> <span class="o">=</span> <span class="n">get_helper</span><span class="p">(</span><span class="s">'github'</span><span class="p">,</span> <span class="n">require_token</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
</code></pre></div></div>

<p>and ensure that the issue posts are completely programmatic. If you don’t require a token,
the URL will be printed to the terminal, and a window opened (in the case of having a desktop).
For either approach, you can also skip the verification of the whitelisted environment variables:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">helper</span> <span class="o">=</span> <span class="n">get_helper</span><span class="p">(</span><span class="s">'github'</span><span class="p">,</span> <span class="n">confirm</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
</code></pre></div></div>

<p>and in fact it would make sense to do this for a totally automated submission:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">helper</span> <span class="o">=</span> <span class="n">get_helper</span><span class="p">(</span><span class="s">'github'</span><span class="p">,</span> <span class="n">require_token</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span> <span class="n">confirm</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
</code></pre></div></div>

<h2 id="questions">Questions?</h2>

<p>I hope that this small library can be helpful to Python maintainers that want a little more
automated error reporting. The only moduled required for this base GitHub interaction
is <a href="https://requests.readthedocs.io/en/master/">requests</a>, and additional support for
asciinema, uservoice, or discourse can be installed if desired, e.g.,:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>pip <span class="nb">install </span>helpme[discourse]
pip <span class="nb">install </span>helpme[asciinema]
pip <span class="nb">install </span>helpme[uservoice]
</code></pre></div></div>

<p>Would you like help to write a custom workflow? Please <a href="https://github.com/rseng/github-support/issues">open an issue</a> and I’d be happy to help. We are going to be writing an integration with datalad, and I’m excited! If you want more examples of arguments and usage of helpme, see <a href="https://vsoch.github.io/helpme/helper-github#headless">the GitHub headless docs</a>. And if you are a research software engineer that wants to join the rseng community to collaborate on projects, please also send me a note.</p>