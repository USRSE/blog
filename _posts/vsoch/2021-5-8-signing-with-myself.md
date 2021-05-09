---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2021-05-08 08:30:00'
layout: post
original_url: https://vsoch.github.io//2021/signing-with-myself/
title: Signing with Myself
---

<p>Have you ever found yourself… signing with yourself? 🤔️</p>

<div style="padding: 20px;">
   <img src="https://vsoch.github.io/assets/images/posts/github/signing-with-myself.png" />
</div>

<blockquote>
  <p>Signing with myself, oh-oh <br />
Signing with myself <br />
When some coding ensues and I’m in a groove <br />
Well I’m signing with myself <br />
Ah oh oh-oh <br /></p>
</blockquote>

<p>This happened unexpectedly. I had recently changed git gpg signing email, and opened a 
pull request (note there is one author, me, for the commit) and then when
it was time to squash and merge, I saw that I was signing with myself 🎶️. It turns
out this is an interesting little bug (or feature?) that I worked with GitHub
support to unravel, and it’s based on GitHub’s history with this no-reply
email address. After some back and forth to get to the bottom of it, I have 
their blessing to share with you today (because it’s kind of neat).</p>

<h2 id="how-to-sign-wth-yourself">How to sign wth yourself</h2>

<h4 id="the-github-no-reply-address">The GitHub no-reply address</h4>

<p>GitHub accounts created before July 18, 2017 (older fogies like myself)
have a no-reply email address formatted with the username as follows:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>username@users.noreply.github.com.
</code></pre></div></div>

<p>But if you created your account after this date, for reasons that I don’t know
but would probably guess it might be related to the re-use of usernames, the no-reply
address also included a seven digit number plus the username, e.g:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>ID+username@users.noreply.github.com
</code></pre></div></div>

<p>In practice I’m not sure that all IDs are seven (I’ve seen six) but we can
confidently state that the number of some length was added. This means that
my account had the older email. That is, until (likely a few years ago) I decided
to select “Keep my email address private” in my profile. Once this happens,
Github updates your no-reply email from the older format without the number
to the newer one. There is no going back to the older format. This is the
address that is used for web-based git operations (for example, like squashing
and merging as we see in the picture at the top).</p>

<h4 id="sign-away-merrill">Sign away, Merrill</h4>

<p>So why didn’t I see this until recently? The reason is because I was signing
with a (non-GitHub) email that was associated with my account. I had a change
in my job this year, so I updated my signing email to be my no-reply address.
It just had to be associated with a gpg key, so I figured it would be okay to use
my no-reply address. That’s when this “signing with myself” phenomena started to show up. So to reproduce
this issue, the GitHub support staff was able to do the following. Note that
for the first step, I only created two keys with equivalent addresses to debug:</p>

<ol class="custom-counter">
  <li>Create a GPG key for each of ID+username@users.noreply.github.com and username@users.noreply.github.com</li>
  <li>Set your commit email address in Git to username@users.noreply.github.com</li>
  <li>Create a signed commit in a new branch using git commit -a -s -m "my signed commit" README.md</li>
  <li>You can use git log to verify that you see the signature in the git commit.</li>
  <li>Push your branch to GitHub</li>
  <li>Open a pull request</li>
  <li>Select Squash and merge and note the newly added "Co-authored-by" line.</li>
</ol>

<p>For the fourth step when you’ve signed and are checking the log, you should
only see your signature. There is no “Co-authored-by” line – only a “Signed-off-by” line.</p>

<h4 id="squash-and-merge">Squash and merge</h4>

<p>When you try to Squash and merge you will see both:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>Signed-off-by: username username@users.noreply.github.com
Co-authored-by: username username@users.noreply.github.com
</code></pre></div></div>

<p>This default message is generated (that you can edit before squashing) because
GitHub looks at the author metadata inside commits, and then links
that to user accounts. In this dummy example, since squashing and merging
is a web-based git operation, the committer is the address with the number,
and the author is still the email we signed with, the one without.
GitHub interprets these different emails as two different people, but I think
they are both still linked to my account. Strangely in the GitHub interface the
numbers don’t seem to show up, so it looks like it’s authored and co-authored by the same email.
At this point, if this were a real pull request, you would click “Squash and merge”
and the (possibly multiple) commits would be squashed into one.</p>

<p>There you go! For some follow up, I removed the extra gpg key, and am signing with the address 
with the number so it matches my web-based git operation email
and looks like the same person. Will this bug/feature/weirdness be fixed? I’m not sure. But given
that this had my scratching my head for quite a few months, I wanted to share
in case you ran into the same issue.</p>

<p>Happy GitHubbing! And if got the reference from the title of the post,
maybe it’s time to take a dance break. :)</p>