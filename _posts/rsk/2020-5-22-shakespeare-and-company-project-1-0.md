---
author: Rebecca Sutton Koeser
blog_subtitle: RSE (Research Software Engineer) posts
blog_title: Rebecca Sutton Koeser
blog_url: https://rlskoeser.github.io/
category: rsk
date: '2020-05-22 19:19:51'
layout: post
original_url: https://rlskoeser.github.io/2020/05/22/shakespeare-and-company-project-1-0/
title: On the release of the Shakespeare and Company Project 1.0
---

<p>Last Thursday, I completed the steps to deploy the 1.0 release of the <a href="https://shakespeareandco.princeton.edu/">Shakespeare and Company Project</a>, updated the site index, and checked that things looked OK. It felt momentous, but I wasn’t up to tweeting about it (other than a joke about <a href="https://twitter.com/suttonkoeser/status/1261017230554382336">how quickly I would need a hotfix</a>). I’m grateful to my colleague Rebecca Munson for <a href="https://twitter.com/Shxperienced/status/1261056009302409221">her twitter thread</a> talking about this moment. I also felt emotional, but in ways that made it difficult for me to want to tweet about it.</p>

<p>I was tired, excited, and anxious. Tired, because I had been working (along with the rest of the project team) very hard, with incredible intensity, for the past few weeks. Excited, because it is a thrill to share something like this with the world and see what people think of your work, and what amazing things they will find. Anxious, because if something goes wrong or is broken, it would likely be my fault (or at least, that’s how it feels at this moment). As technical lead on the project and lead developer at the Center for Digital Humanities, I am ultimately the one responsible for the technical choices and infrastructure. Our work is very collaborative, and there are multiple steps where work is reviewed and tested by a number of different people — unit tests, code review, acceptance testing, etc. This is reassuring, but we all know that bugs can still slip through into production uncaught.</p>

<p>It’s unusual to have project publicity like we’d had (<a href="https://lithub.com/this-new-database-shows-the-reading-habits-of-major-20th-century-authors/">featured on LitHub</a>, an <a href="https://paw.princeton.edu/article/digital-humanities-project-opens-records-famed-french-bookshop">article in Princeton Alumni Weekly</a>) — at <em>any</em> point on a DH project, much less as we were wrapping our 1.0 release! We always knew this project would have a much wider audience than the other projects I’ve been involved at since I joined CDH (<a href="https://prosody.princeton.edu/">Princeton Prosody Archive</a> and <a href="https://derridas-margins.princeton.edu/">Derrida’s Margins</a> are admittedly rather niche). In fact, we were testing and releasing a hotfix (to correct an error in the lending library cards order that was introduced in the 1.0 release) shortly before an <a href="https://www.theguardian.com/books/2020/may/15/legendary-paris-bookshop-reveals-reading-habits-ernest-hemingway-gertrude-stein-shakespeare-and-company">article about the Project was published in The Guardian</a>.</p>

<p>Project data is an additional point of anxiety for me. The data for this project has been in my care for the last few years. I’m well aware of the problems that software can introduce into research data, because <a href="https://www.tandfonline.com/doi/full/10.1080/03080188.2016.1165454">I’ve written about it</a>. I handled the migrations from XML to relational database; I consulted on and helped decide many aspects of how we would track the activity in Beach’s lending library; all the large-scale scripted operations on the data were either implemented or supervised by me. Project Director Joshua Kotin is deeply involved in the project work, very familiar with the data and working with it closely — if we had introduced a problem with the data, he or one of the student researchers would probably have caught it (and they <em>have</em> caught and corrected many problems with the data!). But it still feels possible that there’s something I missed.</p>

<p>Another amazing thing about this moment was that we hit our deadline exactly as planned, which I think is probably a first for us (we’ve been close in the past; deadlines are hard!). This is a testament to the power of project management — Rebecca Munson and I took time in January, in collaboration with Joshua Kotin, to scale back the long list of features we wanted to include into something we would all be satisfied with and that could be done in the time we had. We’re aware from reading about the <a href="https://www.lesswrong.com/posts/CPm5LTwHrvBJCa9h5/planning-fallacy">“planning fallacy”</a> that reality is often worse than the “worst case” we can think of on our own — and <em>nobody</em> had “pandemic” in mind for a possible worst case when we planned this milestone (although we haven’t been as impacted as many).</p>

<p>It may have been hard for me to formulate a tweet about the release because my heart was also full. This is a massive, multiyear project and I’ve had amazing collaborators, past and present. Just check out the list of <a href="https://shakespeareandco.princeton.edu/about/credits/">contributors and supporters on the credits page</a>! After the 1.0 release went out, a colleague commented on how long-running this project has been and mentioned the number of babies born since it started. I joked that we should add a list of “production babies” to the credits list the way movie credits sometimes do, thinking in part of the difficulties of citation for DH projects (<a href="http://adamcrymble.blogspot.com/2012/01/is-old-bailey-online-film-or-science.html">is it a science paper or a film?</a>).</p>

<p>I appreciate my colleague <a href="https://twitter.com/gwijthoff/status/1261333173906165760">Grant Wythoff’s twitter thread</a> celebrating the release and providing some enticing tastes of the <em>Project</em>.  I would love to do something similar, but any tour I can give at this point is going to be idiosyncratic. On the technical side, some of my favorite features are the little things you might not notice, and some that are not even visible. For instance, on the Members and Books pages we have standard navigation for paginated results, but we also have drop-down page navigation that lets you jump to a specific section of the results. This was a feature inspired by and adapted from the <a href="https://findingaids.library.emory.edu/">Emory Finding Aids</a> site, which I developed in collaboration with Elizabeth Russey Roke.</p>

<figure>
    <a href="https://rlskoeser.github.io/images/posts/shakespeareandco/borrowing-activity_desktop_mobile.png">
		<img alt="side by side screenshot of browse interfaces on Shakespeare Company Project and Emory Finding Aids" src="https://rlskoeser.github.io/images/posts/shakespeareandco/borrowing-activity_desktop_mobile.png" />
	</a>
<figcaption>
    <p>Shakespeare and Company Project Members page sorted A-Z with drop down navigation to jump to a specific point in the list (left); Emory Finding Aids browsing collections alphabetically with similar pagination (right).</p>

</figcaption>
</figure>

<p>I also love the way the activity tables you’ll see if you visit the site with a desktop or tablet screen display as tiles on mobile, so the same information is readable and accessible on a smaller screen.</p>

<figure>
    <a href="https://rlskoeser.github.io/images/posts/shakespeareandco/alpha-pagination.png">
		<img alt="side by side screenshot of the same borrowing activity as a table on desktop and tiles on mobile" src="https://rlskoeser.github.io/images/posts/shakespeareandco/alpha-pagination.png" />
	</a>
<figcaption>
    <p>A selection of <a href="https://shakespeareandco.princeton.edu/members/bernheim-antoinette/borrowing/">Antoinette Bernheim’s borrowing activity</a> in desktop and mobile views.</p>

</figcaption>
</figure>

<p>Another thing you’re statistically unlikely to discover on your own: the site should be largely functional without JavaScript. This cost us in development time and additional complexity, and I’ve had second thoughts about it at times, but I feel strongly that things that can work without JavaScript should. It’s also important to me that as much as possible CDH project sites are stable, durable, citable objects with clean, readable, reliable URLs for content, and I think that ensuring that the site works without JavaScript (to the degree possible) helps with these goals. These concerns are at odds with the dynamic interactions that people tend to expect from web interfaces these days. I was reassured by a recent <a href="https://twitter.com/geekygirlsarah/status/1260409688413306882">twitter thread from Sarah Withee</a> on using the NoScript browser extension and the degree to which JavaScript breaks the basic functionality of the web, and I was reminded that this choice is not just a technical one, but a stance about how the web should work.</p>

<p>My tour of the content would be similarly personal. Shortly after I joined the project, Josh shared stories about some of the members related to their cards — and I remember thinking, every card has a story.<sup id="fnref:1"><a class="footnote" href="https://rlskoeser.github.io/2020/05/22/shakespeare-and-company-project-1-0/#fn:1">1</a></sup> Because of my role on the <em>Project</em>, I know the big picture of the data and I know all the weird corners and edge cases (because we had to figure out how to handle them). <a href="https://shakespeareandco.princeton.edu/members/killen/">Alice Killen</a>, the most prolific reader we have documented, was active from 1922 to 1940 with 1500 events, nearly all of them borrows! The amount of data we have for her makes the pages listing her cards and reading activity relatively slow to load, but she was such an outlier that we decided not to add pagination. Or <a href="https://shakespeareandco.princeton.edu/members/raphael-france/">Raphael</a>, who we joked was a time-traveler because of a data error (long since corrected) that made it look like a library member checked a book out and took it back to the Middle Ages.</p>

<figure>
    <a href="https://rlskoeser.github.io/images/posts/shakespeareandco/sorting-bernheims.jpg">
		<img alt="photos of team members looking at printed lending library cards on a board to separate them and sort them by date" src="https://rlskoeser.github.io/images/posts/shakespeareandco/sorting-bernheims.jpg" />
	</a>
<figcaption>
    <p>Sorting out the Bernheim lending library cards. Joshua Kotin gesturing to the cards as Rebecca Koeser and Nick Budak look on (left); detail of the sorted cards (middle); Elspeth Green and Rebecca Koeser looking at project data together (photos by  Jean Bauer and Nick Budak, 2018).</p>

</figcaption>
</figure>

<p>The cards for the <a href="https://shakespeareandco.princeton.edu/members/?query=bernheim+OR+prot">Bernheims</a>, which were mixed up and which Josh, Ellie Green, Nick Budak and I sorted out together (two sisters, a neighbor, and an unrelated Bernheim who was a later member). Or the <a href="https://shakespeareandco.princeton.edu/members/leibowitz/cards/">lending library card for René Leibowitz</a>, which has a collection of events for a number of other members on the reverse (figuring out how to display this card on other members’ pages was how I broke the card order that required the 1.0.1 release). I use Gertrude Stein as a frequent test case, but not for the reasons you might think — her last name occurs <a href="https://shakespeareandco.princeton.edu/members/?query=stein">fully and partially in other member names</a> (Leo Stein, Birstein, Armstein, Goldstein, etc.), and she is one of the members who has <a href="https://shakespeareandco.princeton.edu/members/stein-gertrude/cards/9d839618-9d3e-4e8e-9d6c-28b46d337b11/">events with unknown years</a> — a <a href="https://cdh.princeton.edu/updates/2019/12/05/coding-unknowns/">technical challenge</a> I’ve written about previously.</p>

<p>A week after the release, I’m thrilled, relieved, and reassured. Thrilled by the attention the <em>Shakespeare and Company Project</em> has received (additional articles have appeared in <a href="https://elpais.com/cultura/2020-05-19/que-libros-compraban-simone-de-beauvoir-joyce-hemingway-o-lacan-en-paris.html">El País</a> and <a href="https://www.letemps.ch/culture/lisaient-joyce-hemingway">Le Temps</a>) and amazed at the number of visitors to the site and continuing interest as people discover material and share their finds on Twitter. Relieved, because nothing major has broken; we’ve found minor problems, but we’re already working to correct them. Reassured, because our collaborative work and processes are doing what they should to help us develop high-quality, well-documented code that is easy to maintain. We’re continuing to work on the <em>Project</em> through the end of the month, because we’ve learned from past project launches that there are <em>always</em> unexpected changes and fixes that we’ll discover in this period. As we close this chapter, CDH staff will be taking time to reflect as a group and look back at what we’ve learned from this experience to determine together what we should take forward into new projects, and where we can do better.</p>

<p><em>My thanks to <a href="https://cdh.princeton.edu/people/kathryn-carpenter/">Kate Carpenter</a>, <a href="https://cdh.princeton.edu/people/natasha-ermolaev/">Natalia Ermolaev</a>, and <a href="https://cdh.princeton.edu/people/camey-vansant/">Camey VanSant</a> for invaluable feedback and edits, which helped clarify and improve this post.</em></p>
<div class="footnotes">
  <ol>
    <li id="fn:1">
      <p>This puts me in mind of W. S. Merwin’s poem “The Unwritten” on the potential for words and stories in a simple pencil: “every pencil in the world / is like this.” <a class="reversefootnote" href="https://rlskoeser.github.io/2020/05/22/shakespeare-and-company-project-1-0/#fnref:1">&#8617;</a></p>
    </li>
  </ol>
</div>