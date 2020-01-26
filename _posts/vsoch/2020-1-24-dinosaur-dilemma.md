---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2020-01-24 04:30:00'
layout: post
original_url: https://vsoch.github.io//2020/dinosaur-dilemma/
title: The Dinosaur Dilemma
---

<p>A few weeks ago I interviewed an engineer for <a href="https://us-rse.org/rse-stories" target="_blank">RSE Stories</a> (note
the episode isn’t released yet so I’m keeping the identity of said engineer a secret!) and she
expressed that one of the first learning experiences she had in graduate school was doing
simulations. I did a few simple simulations with neuroimaging data during graduate school,
but largely these felt a little boring, and I wanted to do something a little more fun.
I also wanted to be able to visualize the simulation - because honestly, that seemed more powerful than just
generating texty outputs that a human would have a hard time parsing over. This was the birth of the idea for the “dinosaur dilemma.”
In this post, I’ll talk about my thought and development process, and my goal is that if someone
else is interested in tackling a problem like this, you might have some ideas for how to approach it.
Here is a sneak peak of what the end of a simulation looks like, where the last dinosaur (purple)
is wandering around for food (avocado trees, green) but he eventually dies due to some combination
of weather and lacking food to make up for his size.</p>

<div style="padding: 20px;">
<img src="https://raw.githubusercontent.com/vsoch/dinosaur-dilemma/master/img/dinosaur-dilemma.gif" />
</div>

<p>It’s called the Dinosaur Dilemma because, ultimately, it seems like the dinos are in trouble
for the most part, based on my decisions for modeling the world. More on this later.
If you want to skip over the verbosity and jump to the dinosaur dilemma music videos, see <a href="https://vsoch.github.io/usrse-feed.xml#running-simulations">the running simulations</a>.</p>

<p><br /></p>

<h1 id="the-dinosaur-dilemma">The Dinosaur Dilemma</h1>

<p>When you are trying something new, it’s sometimes tempting to jump into the latest and greatest, but
I think it’s usually best to start with simple, and make more complex as your understanding grows.
With this logic, I decided that my first attempt at a simulation should be a simple setup that warranted
interaction between two characters. The characters would interact in a basic world and at the end, we would be
interested to know how they turned out (evolved) given the parameters of the simulation. 
It resulted in a GitHub repository <a href="https://github.com/vsoch/dinosaur-dilemma" target="_blank">vsoch/dinosaur-dilemma</a>
along with a package on pypi, <a href="https://pypi.org/project/dinolemma/" target="_blank">dinolemma</a>.
See the repository for install and usage instructions.</p>

<p><br /></p>

<h1 id="development">Development</h1>

<p>Since I had no clue what I was doing, the only way that I could think about this was in stages.</p>

<h3 id="stage-one-stateful">Stage One: Stateful</h3>

<h4 id="1-environment">1. Environment</h4>

<p>The first thing to design was the environment, meaning a stateful base that had a set of variables (e.g., temperature, humidity) that would vary
on some regular increment and then influence the entities that live in it downstream. For example, a base environment might 
be defined by a season and day that leads to a particular temperature that has downstream influences on the organisms that live
in it. If my environment has a function to cycle through a unit (e.g., a day) then I can update it’s state, and
then update the entities in it depending on the new state. I wound up first doing this cycling through days totally
automated (with a time delay) but when I later added the graphical interface, I added a function handle to do the same but to
allow some external entity to control progression. Basically, instead of the simulation progressing on its own, 
some user could click on the graphical interface to progress to the next day.</p>

<h4 id="2-entities">2. Entities</h4>

<p>Once the environment is defined, the next level of stateful objects needed to be defined - the entities that live within
the environment. The entities needed to first update themselves based on the changed environment, and then interact.
Interaction would come down to each entity changing location on some grid, and if the location is in the vicinity
of another entity, then the interaction would occur. I thought about whether I wanted all entities to move (and then 
interact) versus allowing them to interact as they move, and I chose the latter. The reason is because
we would allow for multiple interactions for any given entity in one turn, and that’s more interesting.
I also realized that I needed to ensure that the order of movement was randomized. When I thought about this interaction
step, this is also when I realized that avocado trees cannot move.</p>

<h4 id="3-interactions">3. Interactions</h4>

<p>Every entity would need to have defined rules for interaction with other entities. When all entities
in the simulation change location, those that are within some vicinity of one another
are allowed to interact. Interaction can further influence the state of the entity,
or even lead to creation or destruction of said entity. This is where I could get creative
for how an interaction occurred. For example, when two dinosaurs interact, since they each
have a gender (male, female, and hybrid), aggression and size, interactions could range from
everything to mating, mating and fighting, or doing nothing at all. Given a fight, the size
of the dinosaur is a large factor in winning, and if a dinosaur loses a fight, there is some
chance that it will die based on it’s other vitality signs (size, hunger, etc.) A hungry dinosaur
is obviously not as strong as a well-fed one!</p>

<p>At the end of the design of stage 1, I decided that I would have developed essentially a text based, stateful simulation.
I would be able to run it with some set of starting conditions, and then observe the interactions
over a particular number of time steps (days) and some final outcome. This is largely what I made, here
is an example of the asciinema running:</p>



<h3 id="stage-two-graphical">Stage Two: Graphical</h3>

<p>Once the stateful simulation was designed, I wanted to visualize it. The text printing to the screen
is fine, but it doesn’t do justice to show what is going on spatially. While I realized that I might
want to re-implement it in a browser (e.g., using JavaScript, something like d3.js or even
Web Assembly) I ultimately realized that the simpler solution was to try out <a href="https://www.pygame.org/news" target="_blank">pygame</a>
since it would already be implemented in Python. I’ve seen pygame for years but never had
given it a try, so this seemed like a good time for that. In retrospect, since the grid 
was already generated and controlled by the initial library, porting it into 
graphics turned out to be much easier than I anticipated. If you’ve never tried
pygame, I highly recommend it.</p>

<h4 id="stage-three-live">Stage Three: Live</h4>

<p>To be fair, I haven’t implemented this yet, because it would warrant taking a lot more time
than 20 minutes here and there, and I’d need a really good (likely work related) reason to do it.
But to be comprehensive in discussion of stages, this third “live” stage would be different
from the stateful approach described in stages 1 and 2. What we would essentially want is
a bunch of entities that are co-existing in an environment, and then reacting to one another.
To achieve this I would have an implementation that has entities as indepedently running
things (either with containers or processes) and each would be able to emit and subscribe
to one another’s events. Likely there would be a shared environment that can be the
common fabric for entities to discover one another, and after that, it would be a
free for all! I could even add in a layer of probabilities for interaction, and create
a larger grid that might allow for dinosaurs to move more than one space per turn, or
to have a higher probability of returning to a location where avocado trees were 
previous found. This is akin to giving the dinosaurs memory. There are likely even implementation
ideas that could be based entirely on probabilities and distances, and do away with
an actual grid (and instead use a space). Anyway, I didn’t work on this.</p>

<h2 id="characters">Characters</h2>

<p>It was fun to think about my characters, and in terms of the code, thinking about
how to create a common “Entity” class that could be used to subclass each of “Dinosaur”
and “AvocadoTree.” I won’t get into details here, but I wound up designing a general group
of entities, and then using general methods to reproduce, interact, and iterate.
<a href="https://github.com/vsoch/dinosaur-dilemma/tree/master/dinolemma" target="_blank">Take a look at the code</a> if you
are interested in this, and please open an issue if you want to talk about any of the design.
The game.py largely includes the DinosaurDilemma class that controls the grid
and interacts with the entities defined in entity.py and interactions in interactions.py.
I wanted avocado and dinosaur classes to be easily found, so they are found in dinosaurs.py
and avocados.py, respectively.</p>

<h3 id="dinosaurs">Dinosaurs</h3>

<p>Dinosaurs are the main character in this world, and we initialize the world
with some number.</p>

<h4 id="attributes">Attributes</h4>

<p>Specifically, a dinosaur wanders around and has the following
attributes:</p>

<ul>
  <li><strong>hunger</strong>: each dinosaur is hungry, and gets more hungry as the simulation progresses. If the dinosaur encounters a ripe avocado, he will eat it and the hunger will subside. Each dinosaur has a slightly different threshold for deciding to eat another dinosaur.</li>
  <li><strong>size</strong>: each dinosaur has a randomly set size. A larger dinosaur is obviously requiring more food than a smaller one, and a larger one is also advantaged to be able to eat a smaller dinosaur, if desperate.</li>
  <li><strong>disease</strong>: if a dinosaur is hungry and eats an avocado or another dinosaur with a disease, he can get sick. A sick dinosaur moves less, and thus has a greater chance of dying due to hunger or even being eaten by another dinosaur.</li>
  <li><strong>gender</strong>: A dinosaur has a 45% change of being male or female, and a 10% chance of being a hybrid, which can reproduce without a mate. Only mature dinosaurs (greater than or equal to 80% of their full adult size) can reproduce, and with every interaction, there is only some small percentage of it.</li>
</ul>

<h4 id="actions">Actions</h4>

<p>The dinosaur has the following actions:</p>

<ul>
  <li><strong>move</strong>: for each turn of the game, the dinosaur moves, and then interacts with whatever he finds in his new spot.</li>
  <li><strong>eat</strong>: a dinosaur can choose to eat an avocado, or even another dinosaur, depending on the size and level of hunger.</li>
  <li><strong>sleep</strong> a dinosaur can choose to sleep (with some probability) if he is sick to increase the chance of getting better.</li>
  <li><strong>reproduce</strong> a dinosaur that encounters another dinosaur (mature of the opposite gender) has some percent change of reproduction.</li>
</ul>

<h3 id="avocados">Avocados</h3>

<p>Avocados are grown on trees that are scattered in the environment. For any given tree, it must be a certain age to produce avocados, and once it’s old enough, it can only generate a certain number of avocados over a period of time. This gives us the following attributes:</p>

<ul>
  <li><strong>mature</strong>: a mature tree cannot be eaten by a dinosaur, and can produce avocados. An immature tree can be eaten entirely and removed from the game.</li>
  <li><strong>avocados</strong>: once a tree is mature, it holds a certain number of avocados</li>
  <li><strong>disease</strong>: any tree can get a disease with a small probability. Getting a disease puts the tree at risk for dying, or getting a dinosaur sick.</li>
</ul>

<p><br /></p>

<p>Since avocados cannot move, this make their interactions far less interesting. Largely, they wait around to be interacted with by way of a dinosaur,
and on each turn, have some chance of reproducing, getting a disease, or dying from the weather conditions.</p>

<h2 id="variables">Variables</h2>

<p>For each of the scenarios above, there must be probabilities generated within some range (set when the game starts) and then allocated to randomly generated entities, which are also randomly placed on a game board of some size. The interesting thing for both entities is that they can also interact with the environment. For example, each avocado tree has a custom and randomly set threshold of temperatures that are tolerable. This is why when we shift into the winter season,
a small subset of the trees will die off.</p>
<h2 id="running-simulations">Running Simulations</h2>

<p>Now that you have a general sense of how the various probabilities work, while I can’t
set up every simulation that we could think of, I’ll show you examples of a few simple cases.</p>

<ol class="custom-counter">
   <li>50 dinosaurs, 50 trees</li>
   <li>100 dinosaurs, 30 trees</li>
   <li>30 dinosaurs, 100 trees</li>
</ol>

<p>I stream music as I’m working - yes, all day, every day - so I decided to make this a little more fun
(or funny given the topic of dinosaurs eating avocados and one another) and add some
dramatic music to eat of these simulations. For each video, the panel on the left is
what you see in the graphical interface, and the panel on the right is a (cropped)
version of the metadata printed to the terminal. I didn’t pay too much attention
to the cropping because you can see, for the most part, how the populations change
and interact in the panel on the left.</p>

<h2 id="100-dinosaurs-and-30-trees">100 dinosaurs and 30 trees</h2>



<p>In this simulation I wanted to give the dinosaurs (purple) the advantage by making more of them. Or, given that they eat one another, am I really doing that?
The interesting part is that they start out relatively clustered together, eat one another, and then at the end we are left with 
avocado trees (green). If I continued the simulation I suspect the avocado trees would have grown over the entire board! I chose
this particular song because of chosen terms like “safe and sound” and “you’ll be all right” in the context of the poor little dinos
eating one another! If you notice toward the end, when it switches to winter we do see a few of the avocado trees die because
they can’t survive the cold temperatures.</p>

<h3 id="50-dinosaurs-and-50-trees">50 dinosaurs and 50 trees</h3>



<p>This is an example of running the Dinosaur Dilemma simulation with an even number of trees and dinosaurs. We see that the simulation starts in warmer seasons, and dinosaurs quickly eat one another (or starve). The avocado trees reproduce successfully, and spread their distribution. The interesting part comes toward the end because we have just one dinosaur that is barely surviving on random encounters with trees, and then said dinosaur finally starves. Interesting, when we see the season switch from fall to winter, a large number of trees die because the temperature has dropped below their particular threshold.</p>

<p>I want to give a shout out to the music for these videos, someone I’ve been listening to for over 10 years now, <a href="https://www.youtube.com/channel/UCWrtsravWX0ANhHiJXNlyXw" target="_blank">Sam Tsui</a> and of course, Queen! I’ve played some of Sam’s songs on repeats for years. I’m not kidding.</p>

<h2 id="30-dinosaurs-and-100-trees">30 dinosaurs and 100 trees</h2>



<p>This simulation is interesting because we see the dinosaurs quickly die away, and the avocado trees actually start to get a bit overgrown.
I stop the simulation before going further, but this might be a good example for how multiple species are needed to maintain some balance.</p>

<h2 id="what-comes-next">What comes next?</h2>

<p>This small demo is only the beginning to show how simply it can be to think through a simulation you want to create,
and then first create a text-based version and then an interactive visual one. Specifically, there are many things
that I’d want to improve upon or otherwise continue working on.</p>

<h3 id="modeling-balance">Modeling Balance</h3>

<p>From these simple tests, the simulation is clearly biased to favor survival of the trees and eventual (and sometimes quick)
death of the dinosaurs. Arguably, there is some number of dinosaurs and trees, and then attributes of those things, that would produce two
species that can co-exist in some kind of balance. We would want the tree growth to offset the dinosaur hunger, and to have
enough trees to prevent the dinosaurs from eating one another.</p>

<h3 id="realistic-scenario">Realistic Scenario</h3>

<p>Dinosaurs and avocado trees are fun, but it would be much more useful to have a simulation that is actually attempting
to model real life phenomena. I suspect I’d realize quickly how many things I cannot model and how bad my simulation is,
but it would still be fun and a good learning experience.</p>

<h3 id="interface-design">Interface Design</h3>

<p>My initial instinct was to make the simulation really graphical, meaning having little dinosaurs and a snazzy logo.
But when I thought about this more, I realized that akin to creating other kinds of scientific visualizations,
by adding more extra stuff I would be taking away from the core thing I wanted to show - the dinosaurs moving, and
the numbers changing. For this reason, I opted to just represent dinosaurs and trees as different colored
boxes on a grid. And I’m very happy with how this came out. The action I found myself wanting to do was
reset the simulation given some starting state that I didn’t like, so I figured out how to add a reset button.
If I were to add or tweak the interface, I think I’d want to have a small set of summary statistics (total deaths, and reasons
for deaths, and interactions) somewhere on the screen. This seems like a useful thing to have.</p>

<h2 id="summary">Summary</h2>

<p>That’s all folks! I hope this is a simple example that if you want to try something you’ve never done before,
you largely can! Just start simple, make a plan, and go for it.</p>