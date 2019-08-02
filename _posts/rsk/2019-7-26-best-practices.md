---
author: Rebecca Sutton Koeser
blog_subtitle: RSE (Research Software Engineer) posts
blog_title: Rebecca Sutton Koeser
blog_url: https://rlskoeser.github.io/
category: rsk
date: '2019-07-26 12:13:39'
layout: post
original_url: https://rlskoeser.github.io/2019/07/26/best-practices/
title: '*Best* practices?'
---

<p><em>Text of a talk given as part of a <a href="https://www.conftool.org/ach2019/index.php?page=browseSessions&amp;form_session=139&amp;presentations=show">roundtable on the state of Digital Humanities Software Development</a> at <a href="http://ach2019.ach.org/">ACH2019</a> with <a href="https://matthewlincoln.net/">Matthew Lincoln</a>, <a href="https://twitter.com/Zoe_LeBlanc">Zoe LeBlanc</a>, and <a href="https://twitter.com/jamiefolsom">Jamie Folsom</a>.</em></p>

<p>I’m going to talk a bit about software development best practices (as they are called), how I’m using them, and some of my questions and concerns about them.</p>

<p>First, a bit of background. I am the Lead Developer at the Center for Digital Humanities at Princeton where I lead our Development &amp; Design Team.  I think of it as a small team, but I know it’s more than many DH centers or even smaller libraries have. Our team is made up of one full-time developer, one half-time developer shared with another unit on campus, an 80% time User Experience (UX) designer, and myself.</p>

<p>I have formal training in both humanities and computer science: I have a PhD in English from Emory University, and in undergrad I double-majored in English Literature and Computer Science. I also have substantial practical experience in software development, most notably by working as a software developer for Emory University Libraries for ten years after I completed my PhD, where I learned software development best practices.</p>

<p>For my team at Princeton, I have put software development best practices in place in order to help us develop <strong>rigorous</strong> and <strong>maintainable</strong> software, in order to build things that <strong>work correctly</strong> and can be <strong>understood</strong>.  However, most, if not all, of our software best practices come from industry where they are used to build commercial products. How well do they actually translate for research software? How can we use or adapt them in practical ways to get things done, and do them well, but still be critical and thoughtful in our process?</p>

<h2 id="agile">Agile</h2>

<p>Many of the processes I’ve put in place come from <strong>Agile</strong> software development. If you’re not familiar, Agile is an umbrella term for several different lightweight methods including Rapid, Scrum, and Extreme Programming. Agile values include collaboration, communication (especially face to face), frequent delivery of working software, and embracing change.  It’s worth taking a look at the <a href="https://agilemanifesto.org/">Agile Manifesto</a>; the values and <a href="https://agilemanifesto.org/principles.html">principles</a> are good, and there’s a lot that is appealing and easy to agree with. The manifesto is also available in over 60 languages, which is impressive.</p>

<p>I’m certainly not the first to point out that the 17 authors of the Agile Manifesto are all white men (for instance, see <a href="https://medium.com/@sarahtamarakaur/agile-is-fragile-32b42e6e54b4">Agile is Fragile</a> by Sarah Kaur in 2018).
But not only that: they all worked in industry, and at least ten of them were consultants (many running their own firms) based on the brief <a href="https://agilemanifesto.org/authors.html">author biographies</a> on <a href="https://agilemanifesto.org/">AgileManifesto.org</a>.</p>

<figure>
    <a href="https://rlskoeser.github.io/images/posts/ach2019/agile-manifesto-founders.png">
		<img alt="Headshots of all Agile Manifesto authors, showing they are all men" src="https://rlskoeser.github.io/images/posts/ach2019/agile-manifesto-founders.png" />
	</a>
<figcaption>
    <p>Agile Manifesto authors. Image used in  <a href="https://medium.com/@sarahtamarakaur/agile-is-fragile-32b42e6e54b4">“Agile is Fragile” by Sarah Kaur (2018)</a> with no credits; earliest use I could find is in a slide from a <a href="https://fr.slideshare.net/smithcdau/40-agile-methods-in-40-minutes-56693089">2016 talk by Craig Smith, “40 Agile Methods in 40 Minutes”</a></p>

</figcaption>
</figure>

<p>The Agile Manifesto website has a list of the nearly 20,000 <a href="https://agilemanifesto.org/display/index.html">independent signatories</a> who signed on as supporters of the manifesto from 2002 to 2016, when they stopped accepting signatures. Roughly half of those signatures include email addresses, but the emails listed on the site are overwhelmingly commercial (.com) addresses. Of course, this includes email providers like GMail and Yahoo, but it seems telling that there are more signatories from several individual countries than there are .edu emails.</p>

<p><img alt="signatory email chart legend" src="https://rlskoeser.github.io/images/posts/ach2019/agile-manifesto-signature-tlds-key.png" /></p>

<figure>
    <a href="https://rlskoeser.github.io/images/posts/ach2019/agile-manifesto-signature-tlds.svg">
		<img alt="Stacked area chart of Agile Manifesto signatory email top-level domains, showing .com overwhelmingly dominant." src="https://rlskoeser.github.io/images/posts/ach2019/agile-manifesto-signature-tlds.svg" />
	</a>
<figcaption>
    <p>Agile Manifesto signatory emails by top-level domain (top ten only) by year</p>

</figcaption>
</figure>

<p>Agile is associated with flexibility. In the commercial software world, that means adapting to the marketplace changing before you release your app. For us, that might mean a collaborator finds a new archival source that’s incredibly relevant but changes a project substantially.</p>

<hr />

<p>I’m going to talk about three software practices we’re using and share some of my experiences with them.</p>

<h2 id="user-stories">User stories</h2>

<p>A user story is a way of documenting software requirements from the end-user perspective, and it’s generally structured like this:</p>

<blockquote>
  <p>As a <em>[who]</em> I can do <em>[what]</em> so that <em>[why]</em>.</p>
</blockquote>

<p>Here are two examples of actual stories from development on <a href="https://prosody.princeton.edu/">Princeton Prosody Archive (PPA)</a>:</p>

<ul>
  <li>As a content editor, I want to add edition notes so that I can document the copy of an item that’s in the archive.</li>
  <li>As a user, I want to see notes on a digitized work’s edition so that I’m aware of the specifics of the copy in PPA.</li>
</ul>

<p>This structure documents the kind of user (as recognized by the system) that is <em>allowed</em> to use a feature and <em>why</em> they need it. It’s a written agreement between technical and non-technical project team members, that both can understand. It needs to be something that can be tested and verified, and it shouldn’t dictate the implementation - the developers decide that, informed by the <em>why</em>. I find this format useful, and we use stories like this to track development and document features in our project change logs (you can see them on our projects on GitHub). But this format is a challenge for our collaborators when they start working with us: they have to learn to read and write in this language.</p>

<p>Not only that; the user-centric approach that is common to Agile has been critiqued as being too consumer-oriented and taking away the subject’s agency. We probably all agree with this claim from Anne Burdick:</p>

<blockquote>
  <p>Software development methods suited to the creation of tools for shoppers or workers are a poor fit for the design of tools that embody the intentional fuzziness, nuanced positionalities, and reflexive activities
of critical interpretation.</p>
</blockquote>

<ul>
  <li>“Meta!Meta!Meta! A Speculative Design Brief for the Digital Humanities” (<a href="http://visiblelanguagejournal.com/issue/172"><em>Visible Language</em>, 2015</a>)</li>
</ul>

<p>Burdick’s suggestions are interesting, but what she proposes is a <em>design</em> approach; I haven’t come across a practical solution for applying human-centered design to describing features for development. We’re fortunate to have a User Experience Designer on our team, who can do usability testing and user research, but I can’t help feeling there’s still some kind of gap here.</p>

<h2 id="story-points">Story points</h2>

<p>Story points are another tool from Agile. They are an arbitrary measure of complexity used to calculate and project team velocity. Estimation is typically done as a team, and it’s a valuable exercise to talk though upcoming features as a group with multiple perspectives.</p>

<p>Here is an actual velocity chart I generated and shared with my team at an iteration retrospective meeting in June. The graph shows story points completed and issues closed for each two-week iteration, and a rolling velocity based on points for the last three iterations.  We use the rolling velocity to help decide how much work to take on in the next two weeks.</p>

<figure>
    <a href="https://rlskoeser.github.io/images/posts/ach2019/devteam-velocity-20190624.png">
		<img alt="Bar chart showing completed story points, issues, and rolling velocity." src="https://rlskoeser.github.io/images/posts/ach2019/devteam-velocity-20190624.png" />
	</a>
<figcaption>
    <p>CDH Princeton development velocity December 2018 - June 2019.</p>

</figcaption>
</figure>

<p>It’s typical to use the Fibonacci sequence for estimates: the bigger something is, the less we know, and it’s not worth quibbling over whether something is an 8 or a 9: it’s just big. For me, an estimate over 5 is a sign that the task is too big and should be broken down into smaller parts.</p>

<figure>
    <a href="https://rlskoeser.github.io/images/posts/ach2019/planning-poker-cards.jpg">
		<img alt="photo of planning poker cards showing numbers from the Fibonacci sequence" src="https://rlskoeser.github.io/images/posts/ach2019/planning-poker-cards.jpg" />
	</a>
<figcaption>
    <p>Planning poker cards. Photo by Nick Budak, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a></p>

</figcaption>
</figure>

<p>A few months ago I read about an alternate estimation scale that ranges from “everything is known” to “complete ignorance”:</p>

<blockquote>
  <ol>
    <li>Just about everyone in the world has done this.</li>
    <li>Lots of people have done this, including someone on our team.</li>
    <li>Someone in our company has done this, or we have access to expertise.</li>
    <li>Someone in the world did this, but not in our organization (and probably at a competitor).</li>
    <li>Nobody in the world has ever done this before.</li>
  </ol>
</blockquote>

<p>— Liz Keogh, <a href="https://lizkeogh.com/2013/07/21/estimating-complexity/">Estimating Complexity</a></p>

<p>I find this approach interesting, because as developers writing research software, we’re living in that 4-5 zone. This means that development is going to be experimental and unpredictable.</p>

<p>To give you a recent example: on the <a href="https://cdh.princeton.edu/projects/shakespeare-and-company-project/">Shakespeare and Company Project</a>, we wanted to reconcile books from Sylvia Beach’s lending library to OCLC records. We hoped to automate the data work to reduce the manual labor required for cleaning up the nearly 8,000 book records. We had a large task (originally estimated as 5 points) that kept rolling over from one iteration to the next; we couldn’t complete it, because we’d framed it as <em>investigating</em> how to do the reconciliation. Eventually we broke it up into discrete steps, starting with a script to generate a report that project team members could review, so we could figure out how well the reconciliation was going to work and what to do next.</p>

<h2 id="minimum-viable-product-mvp">Minimum Viable Product (MVP)</h2>

<blockquote>
  <p>A minimum viable product (MVP) is a product with just enough features to satisfy early customers, and to provide feedback for future product development.</p>
</blockquote>

<p>— Definition from <a href="https://en.wikipedia.org/wiki/Minimum_viable_product">Wikipedia, Minimum Viable Product</a></p>

<p>The notion of an MVP (which comes from Lean software development) can be a useful scoping tool for DH projects: figure out how to start small, scale back from a grand vision into something that can be implemented in a shorter time frame and still be useful.
But are we building a <em>product</em>? Who are our <em>customers</em>? How do we measure the value of new features when, unlike industry software firms, our standard is not how much money we make?</p>

<p>I’m becoming less certain that this actually works for scholarship.</p>

<ol>
  <li>It’s hard to be minimal.</li>
  <li>How do we identify early adopters and then get them to actually use the project?</li>
</ol>

<p>We have used MVPs or baseline feature lists in our CDH project charters, but I’m starting to question how useful it is, or if we’re doing it wrong. In some projects, it meant we got into specifics too soon and started quibbling over whether a particular search filter or field is essential. I’ve also learned that text-based lists of features, which make complete sense to me, don’t work for everyone; mockups are much more compelling and powerful. In one case, a design mockup actually lead our collaborators to think we’d implemented a feature that wasn’t even intended to be in the MVP.</p>

<p>How can you be minimal in your scope and still adapt to change? For instance, the Shakespeare and Company Project started out looking at lending library cards from the Sylvia Beach archive at Princeton, but then the Project Director  discovered logbooks in the archive which include member subscription information for nearly the whole duration of the library. He knew the lending library cards were only extant for a subset of members, but without incorporating data from the logbooks he didn’t know what portion it represented. Including that new source was important, but was also a substantial expansion of scope.</p>

<p>We’ve tried an informal “soft launch” for some of our projects; for example, the <a href="https://derridas-margins.princeton.edu/">Derrida’s Margins</a> site was made public but we didn’t officially tell anyone about it. I’m still not sure how effective that was; people discovered it and discussed it on social media, and maybe we lost momentum by not taking advantage of that early interest. For <a href="https://prosody.princeton.edu/">Princeton Prosody Archive</a> we shared an early version with PPA board members, which meant we had a roomful of experts looking at the site and talking about the project for a day. Some of them continued to use it for their scholarship, but we didn’t have channels for feedback after that meeting, and our project director didn’t want to share the preliminary site when one of the outcomes of that day was a decision to revisit the visual design.</p>

<p>A proper soft launch might be more of a closed beta with access restricted only to early adopters working with the project, but handling that access is something extra to implement and also raises concerns about being exclusionary.</p>

<p>Other kinds of scholarship have drafts and revisions, deadlines. Are there other, more scholarly ways of scoping projects and imposing limits that we could adapt for software?</p>

<h2 id="etc">etc….</h2>

<p>There are plenty of other software development “best” practices I haven’t touched on that are worth discussing and thinking more about.</p>

<p>One the process side:</p>
<ul>
  <li>Code review</li>
  <li>Time-boxed development (iterations or sprints)</li>
  <li>Retrospective meetings</li>
  <li>Acceptance testing</li>
</ul>

<p>On the technical side:</p>
<ul>
  <li>Unit tests</li>
  <li>Documentation</li>
  <li>Continuous integration</li>
  <li>Automated deploy</li>
  <li>Reusable code</li>
</ul>

<h2 id="community--career-paths">Community &amp; career paths</h2>

<p>Software development best practices are difficult, if not impossible, to implement without a team or community. One possibility we might be able to leverage is the <a href="https://us-rse.org/">US Research Software Engineer Community (US-RSE)</a>. There are established RSE groups elsewhere that are much farther along, and I know the DH community is well-represented in the <a href="https://rse.ac.uk/">UK equivalent</a>. The US organization is still getting started, which means DH developers have a chance to get involved, help shape the group, and ensure that it includes us.</p>

<hr />

<p>See also Matthew Lincoln’s remarks from the same session:  <a href="https://matthewlincoln.net/2019/07/27/whats-in-a-name.html">‘What’s in a name?’ Transitioning from implicit to explicit software dev</a> and the Twitter hashtag for the a wonderful audience <a href="https://twitter.com/search?q=%23ACH2019%20%23SI5">live-tweeting the panel, including Q&amp;A</a>.</p>