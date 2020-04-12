---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2020-04-11 14:15:00'
layout: post
original_url: https://vsoch.github.io//2020/remote-bingo/
title: Remote Bingo
---

<p>I’ve been trying to think of creative ways to have some asynchronous fun, and
when I was planning a research software engineering day (now postponed, for obvious reasons)
for Stanford, one of the activities I thought would be fun is some kind of Bingo.
If it were in person, it would have been easy enough to hand out paper sheets, and have
participants either fill out items while listening to talks, or to take the boards
“home” and complete with their lab over some specified amount of time.  When this was 
no longer possible but the <a href="https://us-rse.org/2020-04-09-virtual-workshop/" target="_blank">US-RSE Virtual Workshop</a>
was being discussed, I jumped at the idea of (still) injecting a little fun with playing Bingo.
I quickly realized it would be much easier for participants to have a virtual (web-based)
game, and I could imagine people playing along while checking off items like:</p>

<ol class="custom-counter">
  <li>Talks about their research cluster</li>
  <li>Tweet shown on slide</li>
  <li>Asks the audience a question</li>
  <li>Slide with all pictures</li>
  <li>Cat walks across the screen</li>
</ol>

<p>It would be so fun! The issue of course, which came up in a conversation with one
of the coordinators for the virtual conference, was that it would be distracting.
And I have to agree, it definitely could be. So while I was disappointed that my 
Remote Bingo idea couldn’t be officially done, I realized that it was still a 
great idea! I could create a simple remote bingo game that any remote person 
could play on their own, or in a more multiplayer sort of setting
(conference calls? virtual happy hour? during a remote conference? watching a movie?). In that we can
participate in conferences without wearing pants, or with our pets, what would
stop someone that wants to have fun from playing Bingo? Exactly. I wanted to build
a very general tool that would have many use cases, be easy to customize, and
hopefully afford some fun in these strange times. If you want to skip over
the description and just play, hop over to 
<a href="https://rseng.github.io/remote-bingo/" target="_blank">rseng/remote-bingo</a>.</p>

<h2 id="remote-bingo">Remote Bingo</h2>

<p>I had some fairly simple goals for this application. I wanted:</p>

<ol class="custom-counter">
  <li>A simple board interface with items to click</li>
  <li>A scoring mechanism</li>
  <li>An ability to clear or reset a board with new items</li>
  <li>Choosing to use a new list of items</li>
</ol>

<p>And for all of the above, I wanted it to be easy for the user to fork
the repository, change some text files to have their own updated items,
and deploy on GitHub pages to deploy their own game. Actually, given that you
have a list of items:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
Item,tags
Seeing where your coworkers live, "remote"
Sorry I've been having networking issues, "remote,conference-call"
Cat walks across the screen,"remote,conference-call"
Rethinking your response to how you are doing, "remote, covid-19"
Oh sorry go ahead, no you go ahead, "remote,conference-call"
Piano starts playing in the background, "remote,conference-call"
Docker automated build fails, "rse"
Ask for help on Slack and you're ignored, "rse"
GitHub goes down!, "rse"
Confuse a child with a pet, "covid-19"
Talk to a pet cat or dog like they are human, "covid-19"
Can you hear me now?, "remote,conference-call"
Outdoor noises during a call, "remote,conference-call"
Child or animal noises during a call, "remote,conference-call"
A Disney themed breathing mask, "covid-19"
Drinks before 3pm, "covid-19"
Attend a virtual happy hour, "covid-19,remote"
A Disney themed breathing mask, "covid-19"
Toilet paper sold out at the store, "covid-19"
Woke up later than 10am, "covid-19,remote"
Netflix asks if you are still watching, "remote,covid-19"
Start a new project, "remote"
Feel overwhelmed with video calls, "covid-19,remote"
Give alcoholism a try, "covid-19,remote"
Start a puzzle, "covid-19,remote"
Bake bread, "covid-19"
Think about toilet paper left, "covid-19"
Do it anyway, "covid-19"
Feel nervous around a package, "covid-19"
Take a nap, hope that dinner is soon, "covid-19,remote"
Overwater your plants, "covid-19"
Develop a super weird routine around getting groceries, "covid-19"

</code></pre></div></div>

<p>The above is the contents of the default “remote-bingo.csv” that the application 
serves on start, you can then create the Bingo instance, add the default list,
and then add other lists for the user to choose from:</p>

<div class="language-js highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nx">$</span><span class="p">(</span><span class="nb">document</span><span class="p">).</span><span class="nx">ready</span><span class="p">(</span><span class="kd">function</span><span class="p">()</span> <span class="p">{</span>

    <span class="kd">var</span> <span class="nx">bingo</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Bingo</span><span class="p">()</span>
    <span class="nx">bingo</span><span class="p">.</span><span class="nx">load_csv</span><span class="p">(</span><span class="dl">"</span><span class="s2">remote-bingo.csv</span><span class="dl">"</span><span class="p">)</span>
    <span class="nx">bingo</span><span class="p">.</span><span class="nx">add_bingo_list</span><span class="p">(</span><span class="dl">"</span><span class="s2">bingo-lists/new-programmer.csv</span><span class="dl">"</span><span class="p">)</span>
    <span class="nx">bingo</span><span class="p">.</span><span class="nx">add_bingo_list</span><span class="p">(</span><span class="dl">"</span><span class="s2">bingo-lists/quarantine-cooking.csv</span><span class="dl">"</span><span class="p">)</span>
    <span class="nx">bingo</span><span class="p">.</span><span class="nx">add_bingo_list</span><span class="p">(</span><span class="dl">"</span><span class="s2">bingo-lists/indoor-activities.csv</span><span class="dl">"</span><span class="p">)</span>
    <span class="nx">bingo</span><span class="p">.</span><span class="nx">add_bingo_list</span><span class="p">(</span><span class="dl">"</span><span class="s2">bingo-lists/virtual-conference.csv</span><span class="dl">"</span><span class="p">)</span>

<span class="p">});</span>
</code></pre></div></div>

<p>Here is the active board!</p>

<div style="padding: 20px;">
  <img src="https://raw.githubusercontent.com/rseng/remote-bingo/master/img/remote-bingo.png" />
</div>

<p>And then the user can change the board to another in the list at the bottom of the page:</p>

<div style="padding: 20px;">
  <img src="https://raw.githubusercontent.com/rseng/remote-bingo/master/img/choose-list.png" />
</div>

<p>The above should give you a hint that I created additional boards for programming
items, quarantine cooking, indoor activities, and (in spirit of the US-RSE virtual conference)
a virtual conference. If you are interested in the implementation, you can see my 
terrible JavaScript skills at the repository, <a href="https://github.com/rseng/remote-bingo" target="_blank">rseng/remote-bingo</a>.
The file with the Bingo instance is <a href="https://github.com/rseng/remote-bingo/blob/master/assets/js/remote-bingo.js" target="_blank">remote-bingo.js</a>. If you want to play bingo, then go to the statically deployed GitHub pages interface at
<a href="https://rseng.github.io/remote-bingo/" target="_blank">https://rseng.github.io/remote-bingo/</a>.</p>

<h2 id="how-do-i-play">How do I play?</h2>

<p>Generally, you get a bingo notification when you fill in a row, column, diagonal, or
fill the entire board. It’s up to you (and your family, friends, or colleagues) how
you want to decide on what constitutes a win, and the context to play in, period.
See the <a href="https://vsoch.github.io/usrse-feed.xml#other-ideas-for-lists">other ideas for lists</a> to get some inspiration.
A game of bingo could be played over a night of watching movies, during the duration
of a conference call, or over a longer period of time. Given this choice, you can
decide how is best to declare bingo (for shorter events, saying Bingo! is probably
sufficient, but longer events might need an email with a screen shot of your board).
If I were planning an event, I’d make sure to use a much longer list of items to
increase the variety of boards generated, and to give good prizes to winners.
If you want to play, go to <a href="https://rseng.github.io/remote-bingo/" target="_blank">https://rseng.github.io/remote-bingo/</a>
and again, please contribute to the repository to improve the lists, or add a new list.</p>

<h2 id="bingo-lists">Bingo Lists</h2>

<p>The following lists are provided with the current Remote Bingo interface.</p>

<h3 id="remote-bingo-1">Remote Bingo</h3>

<p>The default “remote bingo” includes items that are related to working remotely,
COVID-19, and general conference calls.</p>

<div style="padding: 20px;">
  <img src="https://raw.githubusercontent.com/rseng/remote-bingo/master/img/remote-bingo.png" />
</div>

<h3 id="quarantine-cooking">Quarantine Cooking</h3>

<p>The idea here is that you would play this board long term with friends, and try
to get bingo with funny cooking ideas.</p>

<div style="padding: 20px;">
  <img src="https://raw.githubusercontent.com/rseng/remote-bingo/master/img/quarantine-cooking.png" />
</div>

<h3 id="virtual-conference">Virtual Conference</h3>

<p>This board would be fun to play on your own or with a small group while watching
a virtual conference. You might not want to tell the conference that you are playing. :)</p>

<div style="padding: 20px;">
  <img src="https://raw.githubusercontent.com/rseng/remote-bingo/master/img/virtual-conference.png" />
</div>

<h3 id="indoor-activities">Indoor Activities</h3>

<p>This is another quarantine related board, but specific to activities. There is a lot
of room for improvement here!</p>

<div style="padding: 20px;">
  <img src="https://raw.githubusercontent.com/rseng/remote-bingo/master/img/indoor-activities.png" />
</div>

<h3 id="cookies">Cookies</h3>

<p>This is a fun idea if you like to bake, and want to have a longer term competition with
friends. Who can get bingo first baking cookies?</p>

<div style="padding: 20px;">
  <img src="https://raw.githubusercontent.com/rseng/remote-bingo/master/img/cookies.png" />
</div>

<h3 id="new-programmer">New Programmer</h3>

<p>This list is intended for someone learning to program. There could be a lot of 
improvement here as well, I can only do so much on a Saturday!</p>

<div style="padding: 20px;">
  <img src="https://raw.githubusercontent.com/rseng/remote-bingo/master/img/new-programmer.png" />
</div>

<h3 id="other-ideas-for-lists">Other ideas for lists</h3>

<p>Here are some other ideas that I had for lists - if you’d like to contribute a csv file
with items I’d be happy to add to the interface for you to use with your family and friends!</p>

<h4 id="moving-watching-bingo">Moving Watching Bingo</h4>

<p>If you (remotely) watch a shared TV show or film, you could have a board specific to that.
It could even be scoped to include themed items (e.g., scary movies, romantic comedies, TV series)
and I’d imagine it should also include people’s reactions to watching.</p>

<h4 id="stuffed-animals-in-windows">Stuffed Animals in Windows</h4>

<p>I’ve seen that it’s a “thing” to walk around your neighborhood with kids and look
for stuffed animals in windows, or to find chalk messages on the driveway. It would
be fun to have a list of items related to this search, which you could possibly
print out from the interface and bring with you.</p>

<h4 id="quarantine-goals">Quarantine Goals</h4>

<p>Another quarantine related list (and similar to those already shared) would be
a list of funny goals to achieve during the quarantine. E.g., “gain the quarantine 15”
or “attend a meeting without pants.” This would likely be a list that you do on your
own.</p>

<h4 id="acts-of-kindness">Acts of Kindness</h4>

<p>There are a lot of lovely acts of kindess going on, this would be a very
socially appropriate list, in contrast to the previous! You could create a list
of acts of kindness you’ve seen other people do, and then see if you
can do them yourself.</p>

<h4 id="lets-make-cake">Let’s Make Cake!</h4>

<p>If you really love cake (or some other niche item) you can join with a group
of friends and have a battle to see who can make the most different kinds of a
baked good to get a Bingo! For this board, I’d say that you would be required to
take pictures of the result for proof. You could easily do this with any kind
of baked good, pizza, bread, pasta, potato, or food category that has many derivations.
See the cookies example provided already with the repository.</p>

<h2 id="contributing">Contributing</h2>

<p>If you’d like to contribute, here are a few ideas:</p>

<h3 id="1-create-a-custom-list">1. Create a custom list</h3>

<p>I’ve created a set of good starting lists (see <a href="https://vsoch.github.io/usrse-feed.xml#bingo-lists">bingo lists</a> but of course
there is great room for improvement, or even making new lists!</p>

<h3 id="2-add-a-color-picker">2. Add a color picker</h3>

<p>While not essential, it would be fun to allow a user to choose the color to highlight
the squares. It would require adding a color picker (perhaps to the bottom right)
and then storing the chosen color with the Bingo instance. To do this best,
it would be nice to not add additional javascript dependencies.</p>

<h3 id="3-support-for-cache">3. Support for Cache</h3>

<p>Currently, if the user closes the browser window, the current game and its state
are lost. It would be nice to have an ability to cache a state, and then
refreshing the browser would still keep the state.</p>

<h3 id="4-tags">4. Tags</h3>

<p>Currently if you look in the bingo list csv files, there is a second column
for tags that aren’t used. I was thinking that it would be nice to be able
to load a very large file of items, and then filter down to some subset of
categories.</p>

<h3 id="5-explosive-congratulations">5. Explosive Congratulations</h3>

<p>I think that when the user fills the entire board, there should be a more
explosive congratulations (confetti or small animals dancing across the screen?)
I think this could be done with some kind of simple css animation, again,
with preference to not add additional dependencies.</p>

<h3 id="6-saving-board">6. Saving Board</h3>

<p>For games that aren’t done live, it would be good to allow for saving to file
of the screen. I’ve done this many times, but typically with svg, so I’m open
to ideas for how to best implement this for standard html/css.</p>

<h3 id="7-support-for-mobile">7. Support for Mobile</h3>

<p>Mobile development is usually an afterthought for me. Nuff’ said!</p>

<h2 id="thank-you">Thank you!</h2>

<p>I want to say a quick thank you to <a href="https://codepen.io/nbrombal/pen/JAedG">A Game of Dabs</a> on Codepen
that I was able to refactor into a Vue.js application to serve a custom board.
I’m terrible at JavaScript and this example was exactly what I needed to get started.
That’s all folks! Have fun <a href="https://rseng.github.io/remote-bingo/">playing!</a>,
and if you are an #rseng please consider attending the <a href="https://us-rse.org/2020-04-09-virtual-workshop/" target="_blank">US-RSE Virtual Workshop</a>!</p>