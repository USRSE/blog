---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2020-04-26 10:30:00'
layout: post
original_url: https://vsoch.github.io//2020/first-order-model/
title: First Order Models
---

<p>I stumbled on a little bit of magic last night. Specifically, it was the 
<a href="https://github.com/AliaksandrSiarohin/first-order-model">repository</a>
associated with <a href="https://papers.nips.cc/paper/8935-first-order-motion-model-for-image-animation">this paper</a> 
that used a clever algorithm to match a picture (such as a face)
to a video (such as a person talking) to generate a new video. You can then 
pair the audio with the new video, and like magic, you have a talking painting, 
different person, or even an inanimate object with a face. As you might imagine,
I went to town and made a ton of videos, and could barely contain my excitement to
share this with you! Here is one of my favorite:</p>



<h2 id="clone-the-repository">Clone the repository</h2>

<p>If you are planning to run things locally (and install dependencies) you’ll
need to clone the repository, and do that. Note that I’ll show instructions later
for using a container. I found a <a href="https://github.com/AliaksandrSiarohin/first-order-model/pull/123">bug</a> 
with using ffmpeg on my local machine, so I’ll direct you to use the branch from my <a href="https://github.com/researchapps/first-order-model/tree/fix/ffmpeg-frames-bug">fork</a>.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>git clone <span class="nt">-b</span> fix/ffmpeg-frames-bug https://github.com/researchapps/first-order-model
<span class="nb">cd </span>first-order-model
<span class="nb">mkdir</span> <span class="nt">-p</span> driving videos img
</code></pre></div></div>

<p>For each of the folders above, we will put driving videos in driving, output
videos in videos (both those without sound and finished with it) and input
images in img. To install dependencies, after you have your Python environment
of choice active:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>pip <span class="nb">install</span> <span class="nt">-r</span> requirements.txt
</code></pre></div></div>

<p>And then proceed to generate a video for your first shot! After I got this up and
running for the first time, it was pure joy!</p>

<div style="padding: 20px;">
<img src="https://raw.githubusercontent.com/vsoch/vsoch.github.io/master/assets/images/posts/first-order-model/models-run.png" />
</div>

<p>I wound up building a container, and it was actually very unlike me to run something
on my local machine. Either way, the model checkpoints are downlaoded from <a href="https://drive.google.com/drive/folders/1PyQJmkdCsAkOYwUyaj_l-l0as-iLDgeH">here</a>.</p>

<h2 id="generating-a-video">Generating a Video</h2>

<p>The first thing you need to do is to generate videos to match to. Ideally these
should be 256 by 256, and I’d suggest a good starting size is no longer than 40 seconds.
On a CPU this size video will take anywhere between 10 to 20 minutes to be produced.
The repository provides <a href="https://github.com/AliaksandrSiarohin/first-order-model/blob/master/crop-video.py">a script</a> 
to resize a video for you (or actually to generate
an ffmpeg command that you can run) but it didn’t work on my machine because I don’t
have GPU, which is a dependency for the <a href="https://github.com/1adrianb/face-alignment">face alignment</a> library. Instead, 
I found a very easy and free <a href="https://ezgif.com/crop-video">online converter</a> where
you generally want to:</p>

<ol class="custom-counter">
    <li>First reshape the canvas be square, e.g., take the smaller dimension, N, and make the image NXN</li>
    <li>Then resize the video to 256 by 256</li>
    <li>Convert to Mp4 and save</li>
</ol>

<p>I saved these “template videos” to the “driving” folder. If you need to easily
get them off of your phone, Google Photos uploads fairly quickly, and you can
also send a file to yourself in Slack. I’m terrified to admit that I tried sending
a rather embarassing video to myself, and I never found it. I suspect it’s floating
out there somewhere in the internet universe. One additional note is that you shouldn’t
worry too much about how you look, but focus on keeping your head fairly stable
(some movement seems okay but not too much) and thinking about the movement of
your eyes, eyebrows, and mouth.</p>

<h2 id="choosing-a-picture">Choosing a picture</h2>

<p>This is where it gets fun, because you can literally choose any picture that has a mouth
and eyes (and looks like a face). This might include paintings, other people, or stuffed
animals or other inanimate objects. In practice I found that the pictures that worked the
best had some kind of black or dark delineation between the lips, because the model
seems to pick up on these and even add movement and teeth. If your mouth is not
well defined (or if you have other lines in the image that are moving and could be mistaken
for a mouth) it doesn’t come out as well as it would have. For all of these
images that you choose, they should also be 256 by 256. I think you could change
this by updating the config file, but it’s easier to just stick to the defaults.</p>

<p>It’s quite a bit of work editing / cropping all those videos, but it’s worth it!
I used gimp, of course.</p>

<div style="padding: 20px;">
<img src="https://raw.githubusercontent.com/vsoch/vsoch.github.io/master/assets/images/posts/first-order-model/models-gimp.png" />
</div>

<h2 id="running-with-gpu">Running with GPU</h2>

<p>While most of these I ran on my local machine (using a slow CPU) I wanted to test running with GPU,
and so I logged into our cluster and briefly grabbed a GPU node to pull the container 
and try running a model. I wanted to make sure that if others used the container,
that it would work with Singularity.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
singularity pull docker://vanessa/first-order-model
<span class="nb">mkdir</span> <span class="nt">-p</span> videos
<span class="nb">mkdir</span> <span class="nt">-p</span> driving
</code></pre></div></div>

<p>We would then need a bunch of videos and images to use - in my case I had
them on my local machine and used scp to get them onto my scratch space:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
scp <span class="nt">-r</span> driving/ vsochat@login.sherlock.stanford.edu:/scratch/users/vsochat/first-order-model/videos
scp <span class="nt">-r</span> img/ vsochat@login.sherlock.stanford.edu:/scratch/users/vsochat/first-order-model/driving
</code></pre></div></div>

<p>The models and configuration files are provided in the container within the directory /app
so I don’t need to worry about that. Since I’m running these interactively, I decided
to test via an interactive shell.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
singularity shell <span class="nt">--nv</span> first-order-model_latest.sif
<span class="nb">cd</span> /app
</code></pre></div></div>

<p>Your working directory with the videos and images should still be bound to where it
was before (e.g., on my cluster we bind $HOME and $SCRATCH so I can reference files there.
Let’s try running this with GPU! Remember this is relative to /app in the container.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>/usr/bin/python3 demo.py  <span class="nt">--config</span> config/vox-adv-256.yaml <span class="se">\</span>
      <span class="nt">--driving_video</span> <span class="nv">$SCRATCH</span>/first-order-model/driving/lesters-eggs.mp4 <span class="se">\</span>
      <span class="nt">--source_image</span> <span class="nv">$SCRATCH</span>/first-order-model/img/img/image17.png <span class="se">\</span>
      <span class="nt">--checkpoint</span> checkpoints/vox-adv-cpk.pth.tar <span class="se">\</span>
      <span class="nt">--relative</span> <span class="nt">--adapt_scale</span> <span class="se">\</span>
      <span class="nt">--result_video</span> <span class="nv">$SCRATCH</span>/first-order-model/videos/lesters-eggs-17-no-audio.mp4
</code></pre></div></div>

<p><strong>important!</strong> If you have python modules installed to local that might conflict,
 you should either set the python no user site environment variable, or (if you are lazy like me)
just delete them. When you finish up, you can then copy the finished (no audio) files back
to your computer:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nb">mkdir</span> <span class="nt">-p</span> gpu-videos
scp <span class="nt">-r</span> vsochat@login.sherlock.stanford.edu:/scratch/users/vsochat/first-order-model/videos gpu-videos
</code></pre></div></div>

<p>Mind you that it’s up to you to figure out how to do this efficiently.</p>

<h2 id="adding-audio">Adding Audio</h2>

<p>Once you’ve generated the result file, you can open it up with the original file,
and first add the original track to your workspace</p>

<div style="padding: 20px;">
<img src="https://raw.githubusercontent.com/vsoch/vsoch.github.io/master/assets/images/posts/first-order-model/editing1.png" />
</div>

<p>unlink the video and audio</p>

<div style="padding: 20px;">
<img src="https://raw.githubusercontent.com/vsoch/vsoch.github.io/master/assets/images/posts/first-order-model/editing2.png" />
</div>

<p>and then delete the original video. The audio is now paired with your (silent)
video and should match, time wise, 1:1.</p>

<div style="padding: 20px;">
<img src="https://raw.githubusercontent.com/vsoch/vsoch.github.io/master/assets/images/posts/first-order-model/editing3.png" />
</div>

<h2 id="the-final-result">The Final Result!</h2>

<p>Ladies and gentlemen, what I spent an entire day of my weekend doing. Behold! Mind you
these are only the publicly sharable ones - the best ones have real people’s faces
(and I will share with private social networks only).</p>

<h3 id="inanimate-objects">Inanimate Objects</h3>

<p><strong>The Can Opener</strong></p>



<p><strong>Meow Cheese</strong></p>



<p><strong>Meow Flower</strong></p>



<p><strong>Angry Cheese</strong></p>



<p><strong>Angry Pepper</strong></p>



<p><strong>Baby Bumble Bee Octocat</strong></p>



<p><strong>Baby Bumble Bee Avocado</strong></p>



<h3 id="paintings-and-drawings">Paintings and Drawings</h3>

<p><strong>Person Flower</strong></p>



<p><strong>I Pained Myself</strong></p>



<p><strong>You are My Sunshine</strong></p>



<p><strong>Stuffed Flower</strong></p>



<p><strong>Hi Mom and Dad!</strong></p>



<p><strong>Sunburn</strong></p>



<p><strong>You Can’t Get to Heaven</strong></p>



<p><strong>I Painted Myself (Flower)</strong></p>



<p><strong>Lester’s Eggs</strong></p>



<p><strong>Amanda (dark)</strong></p>

<p>This one is a little dark, but it’s meant to be.</p>



<p><strong>Trump Growl</strong></p>



<p><strong>Knock Knock Flower</strong></p>



<p><strong>Knock Knock (2)</strong></p>



<p><strong>Knock Knock (3)</strong></p>



<p><strong>Pancakes</strong></p>



<p>I have more work to do, and more to discuss, but really need to stop today! We didn’t even
scratch the surface because checkpoints are provided for models with different kinds of
videos (e.g., someone walking) and we can also try so many more things. This is definitely
why I shouldn’t be trusted with machine learning! But no matter, I had an amazing day!
If you feel a bit silly and don’t mind recording yourself and putting in a bit of work,
this is a hugely fun and rewarding project. Have fun!</p>