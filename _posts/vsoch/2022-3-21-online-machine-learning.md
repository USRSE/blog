---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2022-03-21 12:30:00'
layout: post
original_url: https://vsoch.github.io//2022/online-machine-learning/
title: Online Machine Learning
---

<p>Where do machine learning projects and web interfaces collide? If you are like me, you are used to
a traditional “batch” style of machine learning - we start with big matrices of data (with some logical
way to split into test and training data) and then we go through batch training and get some metric of accuracy
using our test set. This works fairly well for most applications like static notebooks, research scenarios
where you have access to a big filesystem or amounts of memory, or just on your local laptop using
<a href="https://github.com/tensorflow/tensorflow" target="_blank">tensorflow</a>, <a href="https://scikit-learn.org/stable/" target="_blank">scikit-learn</a>, or another standard ML library we’ve all come to love (or hate, depending on your
view of the whole thing!) So what happens when you hit one of these issues?</p>

<ol class="custom-counter">
  <li>I have streaming data</li>
  <li>I need to deploy a web app and there isn't enough memory or storage for something</li>
  <li>I need my model to easily update over time</li>
</ol>

<p>Oh no… too much data/memory chonk! What can we do?</p>

<h2 id="a-story-of-too-much">A Story of Too Much</h2>

<p>I was recently in this scenario. I was working on a fun personal projet, and had trained a pretty cool model to derive a clustering from software
error messages, and was using <a href="https://radimrehurek.com/gensim/models/doc2vec.html" target="_blank">Doc2Vec</a> from Gensim.
This meant I was:</p>

<ol class="custom-counter">
  <li>Generating a vector (embedding) for each error messages.</li>
  <li>Adding that to a massive data frame.</li>
  <li>Using dimensionality reduction (<a href="https://scikit-learn.org/stable/modules/generated/sklearn.manifold.TSNE.html" target="_blank">TSNE</a> or <a href="https://umap-learn.readthedocs.io/en/latest/" target="_blank">umap</a>) to view it</li>
</ol>

<p><a href="https://buildsi.github.io/spack-monitor-nlp/" target="_blank">Everything was lovely</a> and dandy until I got more data. 
I don’t think it was <strong>that</strong> much when we are talking  about machine learning problems (N=260K), but it was enough that the last step was killed on my local machine. I then took it to our supercomputer, and after waiting forever and a half for a node, it still was killed. Now I could have sought out a bigger node (MOOOOOOAR POWER!) or optimized my computing strategy, 
but I stepped back for a second. You see, I wanted this model to be part of <a href="https://vsoch.github.io/spack-monitor.readthedocs.io/" target="_blank">
spack monitor</a>, which meant that I would need to derive the visualization on a dinky web server. I’d also need to do the initial model
building and update on the server. The solution I was developing wasn’t good enough. I would need to have some
complicated setup to run nightly batch jobs to generate new embeddings and update the model, and then another (once daily perhaps)
updated view of the embeddings. And I couldn’t just use a standard cloud server, I’d need something big. I knew it wouldn’t work.</p>

<h2 id="online-machine-learning">Online Machine Learning</h2>

<p>This led to a search for “streaming” and “machine learning” and I stumbled on the concept of <a href="https://en.wikipedia.org/wiki/Online_machine_learning" target="_blank">online machine learing</a> (online ML):</p>

<blockquote>
  <p>In computer science, online machine learning is a method of machine learning in which data becomes available in a sequential order and is used to update the best predictor for future data at each step, as opposed to batch learning techniques which generate the best predictor by learning on the entire training data set at once. Online learning is a common technique used in areas of machine learning where it is computationally infeasible to train over the entire dataset, requiring the need of out-of-core algorithms. It is also used in situations where it is necessary for the algorithm to dynamically adapt to new patterns in the data, or when the data itself is generated as a function of time, e.g., stock price prediction. Online learning algorithms may be prone to catastrophic interference, a problem that can be addressed by incremental learning approaches.  - Wikipedia Authors</p>
</blockquote>

<p>As you can see, this doesn’t just mean putting machine learning models online. It’s an entirely different way of thinking about ML, where instead of doing things in batches with big matrices, we instead learn <strong>one</strong> data point at a time, and can also make one prediction at a time. We might even add a data point for our model to learn, save an identifier for that addition, and provide a label for it later. A lot of these streaming methods are also neat because they have a temporal aspect, meaning that older samples are discarded as you move through time and have newer data. You can imagine this would work very well for models that are temporally changing. Also note that there are online machine learning techniques for both supervised and unsupervised methods.</p>

<h3 id="river">River</h3>

<p>Very quickly I stumbled on a library called <a href="https://riverml.xyz/latest/" target="_blank">river</a> (and on <a href="https://github.com/online-ml/river" target="_blank">GitHub</a>) along with a Flask server component called <a href="https://github.com/online-ml/chantilly" target="_blank">chantilly</a>, both the work of <a href="https://github.com/maxhalford" target="_blank">@MaxHalford</a>, a data scientist at <a href="https://github.com/carbonfact" target="_blank">carbonfact</a> that (like many a passionate developer) is working on online ML on the side.
I recommend that you check out <a href="https://maxhalford.github.io/" target="_blank">his blog</a> and <a href="https://maxhalford.github.io/blog/predict-fit-switcheroo/" target="_blank">this post</a> and <a href="https://towardsdatascience.com/machine-learning-for-streaming-data-with-creme-dacf5fb469df" target="_blank">this post</a> in particular that I found really helpful for originally learning about online ML. If you’ve gone through an entire PhD and have never heard of it, it’s a hugely refreshing and awesome idea! I am not a machine learning specialist, however Max is, and he describes the design and ideas beautifully. I attended a talk by Max and I’ll also direct you to his <a href="https://maxhalford.github.io/links/#talks" target="_blank">set of slide and talks</a> for your browsing!</p>

<h2 id="django-river-ml">Django River ML</h2>

<p>Alrighty so back to dinosaur land. I was really excited about river. It’s set up to be “the next scikit-learn (or similar) but for online machine learning! Since spack monitor is a Django application, my first thought was to add river to it. But I decided I could
do a lot more for the open source and ML communities if I could instead make a <a href="https://docs.djangoproject.com/en/4.0/intro/reusable-apps/" target="_blank">Django plugin</a>, or “reusable app” that others could plug into their server! So long story short, that’s what I did.
If you are a Django developer and want to easily use river, here are resources for you!</p>

<ol class="custom-counter">
  <li><a href="https://vsoch.github.io/django-river-ml/" target="_blank">Django River ML</a>: the app that you can add to your server.</li>
  <li><a href="https://vsoch.github.io/riverapi/" target="_blank">riverapi</a>: A Python module so you can easily interact with your server.</li>
  <li><a href="https://vsoch.github.io/riverapi/getting_started/spec.html" target="_blank">the API specification</a>: that defines server endpoints, in case you want to create a new tool.</li>
</ol>

<h2 id="how-does-it-work">How does it work?</h2>

<h3 id="installation">Installation</h3>

<p>If you are familiar with Django, this will make sense. If not, this section is basically adding
django-river-ml to a Django server. You will basically pip install it, and then add this to the Django <code class="language-plaintext highlighter-rouge">settings.py</code> file:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">INSTALLED_APPS</span> <span class="o">=</span> <span class="p">(</span>
    <span class="p">...</span>
    <span class="s">'django_river_ml'</span><span class="p">,</span>
    <span class="s">'rest_framework'</span><span class="p">,</span>
    <span class="p">...</span>
<span class="p">)</span>
</code></pre></div></div>

<p>And also to expose URLs:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1"># Add django-river-ml’s URL patterns:
</span>
<span class="kn">from</span> <span class="nn">django_river_ml</span> <span class="kn">import</span> <span class="n">urls</span> <span class="k">as</span> <span class="n">django_river_urls</span>
<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="p">...</span>
    <span class="n">url</span><span class="p">(</span><span class="sa">r</span><span class="s">'^'</span><span class="p">,</span> <span class="n">include</span><span class="p">(</span><span class="n">django_river_urls</span><span class="p">)),</span>
    <span class="p">...</span>
<span class="p">]</span>
</code></pre></div></div>

<p>And of course all this is customizable - everything from the url prefix used to the URL exposed or to add
authentication or not! Check out the <a href="https://vsoch.github.io/django-river-ml/getting_started/user-guide.html#settings" target="_blank">settings</a> to learn more.</p>

<h3 id="client-interaction">Client Interaction</h3>

<p>Okay great! You add these endpoints, which are basically going to allow you to:</p>

<ol class="custom-counter">
  <li>Upload a model you generate locally</li>
  <li>Learn from a trained model</li>
  <li>Get a prediction for a new piece of data</li>
  <li>Apply a label</li>
  <li>Get metrics, stats, or monitor server events.</li>
</ol>

<p>For each of these “external” functions, the same can be accomplished from within the server. E.g., I could
turn off all the externally exposed endpoints, and just use my own endpoints to parse and then present data
to my river model. It’s very flexible for your use case! So now let’s take a look at what these interactions might look like.</p>

<h4 id="authentication">Authentication</h4>

<p>First, if your server requires authentication (e.g., you’ve generated a Django Restful token) you can basically just
export the username and token to the environment for the client to find:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nb">export </span><span class="nv">RIVER_ML_USER</span><span class="o">=</span>dinosaur
<span class="nb">export </span><span class="nv">RIVER_ML_TOKEN</span><span class="o">=</span>xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
</code></pre></div></div>

<p>Authentication can be disabled, and actually this is the default to support the most common use case of you trying it out
on your local machine. Of course for a production server you should be cautious about making this decision!</p>

<h4 id="external-client">External Client</h4>

<p>Next let’s instantiate a client:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kn">from</span> <span class="nn">riverapi.main</span> <span class="kn">import</span> <span class="n">Client</span>

<span class="c1"># This is the default, just to show how to customize
</span><span class="n">cli</span> <span class="o">=</span> <span class="n">Client</span><span class="p">(</span><span class="s">"http://localhost:8000"</span><span class="p">)</span>
</code></pre></div></div>

<p>You might first check out information about the server:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1"># Basic server info (usually no auth required)
</span><span class="n">cli</span><span class="p">.</span><span class="n">info</span><span class="p">()</span>

<span class="p">{</span><span class="s">'baseurl'</span><span class="p">:</span> <span class="s">'https://prod-server'</span><span class="p">,</span>
 <span class="s">'id'</span><span class="p">:</span> <span class="s">'django_river_ml'</span><span class="p">,</span>
 <span class="s">'status'</span><span class="p">:</span> <span class="s">'running'</span><span class="p">,</span>
 <span class="s">'name'</span><span class="p">:</span> <span class="s">'Django River ML Endpoint'</span><span class="p">,</span>
 <span class="s">'description'</span><span class="p">:</span> <span class="s">'This service provides an api for models'</span><span class="p">,</span>
 <span class="s">'documentationUrl'</span><span class="p">:</span> <span class="s">'https://vsoch.github.io/django-river-ml'</span><span class="p">,</span>
 <span class="s">'prefix'</span><span class="p">:</span> <span class="s">'ml'</span><span class="p">,</span>
 <span class="s">'storage'</span><span class="p">:</span> <span class="s">'shelve'</span><span class="p">,</span>
 <span class="s">'river_version'</span><span class="p">:</span> <span class="s">'0.9.0'</span><span class="p">,</span>
 <span class="s">'version'</span><span class="p">:</span> <span class="s">'0.0.18'</span><span class="p">}</span>
</code></pre></div></div>

<p>And then make and upload a model with river:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kn">from</span> <span class="nn">river</span> <span class="kn">import</span> <span class="n">linear_model</span><span class="p">,</span> <span class="n">preprocessing</span>
<span class="kn">from</span> <span class="nn">river</span> <span class="kn">import</span> <span class="n">preprocessing</span>

<span class="c1"># Upload a model
</span><span class="n">model</span> <span class="o">=</span> <span class="n">preprocessing</span><span class="p">.</span><span class="n">StandardScaler</span><span class="p">()</span> <span class="o">|</span> <span class="n">linear_model</span><span class="p">.</span><span class="n">LinearRegression</span><span class="p">()</span>

<span class="c1"># Save the model name for other endpoint interaction
</span><span class="n">model_name</span> <span class="o">=</span> <span class="n">cli</span><span class="p">.</span><span class="n">upload_model</span><span class="p">(</span><span class="n">model</span><span class="p">,</span> <span class="s">"regression"</span><span class="p">)</span>
<span class="k">print</span><span class="p">(</span><span class="s">"Created model %s"</span> <span class="o">%</span> <span class="n">model_name</span><span class="p">)</span>
<span class="c1"># Created model fugly-mango
</span></code></pre></div></div>

<p>And then learn or predict! River provides a bunch of <a href="https://riverml.xyz/latest/api/overview/#datasets" target="_blank">datasets</a> you can play with,
and there is support to load them from sklearn. Here is learning:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kn">from</span> <span class="nn">river</span> <span class="kn">import</span> <span class="n">datasets</span>
<span class="c1"># Learn from some data
</span><span class="k">for</span> <span class="n">x</span><span class="p">,</span> <span class="n">y</span> <span class="ow">in</span> <span class="n">datasets</span><span class="p">.</span><span class="n">TrumpApproval</span><span class="p">().</span><span class="n">take</span><span class="p">(</span><span class="mi">100</span><span class="p">):</span>
    <span class="n">cli</span><span class="p">.</span><span class="n">learn</span><span class="p">(</span><span class="n">model_name</span><span class="p">,</span> <span class="n">x</span><span class="o">=</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="o">=</span><span class="n">y</span><span class="p">)</span>
</code></pre></div></div>

<p>And predicting:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1"># Make predictions
</span><span class="k">for</span> <span class="n">x</span><span class="p">,</span> <span class="n">y</span> <span class="ow">in</span> <span class="n">datasets</span><span class="p">.</span><span class="n">TrumpApproval</span><span class="p">().</span><span class="n">take</span><span class="p">(</span><span class="mi">10</span><span class="p">):</span>
    <span class="k">print</span><span class="p">(</span><span class="n">cli</span><span class="p">.</span><span class="n">predict</span><span class="p">(</span><span class="n">model_name</span><span class="p">,</span> <span class="n">x</span><span class="o">=</span><span class="n">x</span><span class="p">))</span>
</code></pre></div></div>

<p>And note how I’m not calling the first “training.” I don’t know if this is correct, but training
makes me think of the batch approach. A streaming, incremental approach is more learning (over time).
And then you might look at stats or metrics, or just monitor events!</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">cli</span><span class="p">.</span><span class="n">metrics</span><span class="p">(</span><span class="n">model_name</span><span class="p">)</span>
<span class="p">{</span><span class="s">'MAE'</span><span class="p">:</span> <span class="mf">7.640048891289847</span><span class="p">,</span>
 <span class="s">'RMSE'</span><span class="p">:</span> <span class="mf">12.073092099314318</span><span class="p">,</span>
 <span class="s">'SMAPE'</span><span class="p">:</span> <span class="mf">23.47518046795208</span><span class="p">}</span>

<span class="n">cli</span><span class="p">.</span><span class="n">stats</span><span class="p">(</span><span class="n">model_name</span><span class="p">)</span>
<span class="p">{</span><span class="s">'predict'</span><span class="p">:</span> <span class="p">{</span><span class="s">'n_calls'</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span>
  <span class="s">'mean_duration'</span><span class="p">:</span> <span class="mi">2626521</span><span class="p">,</span>
  <span class="s">'mean_duration_human'</span><span class="p">:</span> <span class="s">'2ms626μs521ns'</span><span class="p">,</span>
  <span class="s">'ewm_duration'</span><span class="p">:</span> <span class="mi">2362354</span><span class="p">,</span>
  <span class="s">'ewm_duration_human'</span><span class="p">:</span> <span class="s">'2ms362μs354ns'</span><span class="p">},</span>
 <span class="s">'learn'</span><span class="p">:</span> <span class="p">{</span><span class="s">'n_calls'</span><span class="p">:</span> <span class="mi">100</span><span class="p">,</span>
  <span class="s">'mean_duration'</span><span class="p">:</span> <span class="mi">2684414</span><span class="p">,</span>
  <span class="s">'mean_duration_human'</span><span class="p">:</span> <span class="s">'2ms684μs414ns'</span><span class="p">,</span>
  <span class="s">'ewm_duration'</span><span class="p">:</span> <span class="mi">2653290</span><span class="p">,</span>
  <span class="s">'ewm_duration_human'</span><span class="p">:</span> <span class="s">'2ms653μs290ns'</span><span class="p">}}</span>
    
<span class="c1"># Stream events
</span><span class="k">for</span> <span class="n">event</span> <span class="ow">in</span> <span class="n">cli</span><span class="p">.</span><span class="n">stream_events</span><span class="p">():</span>
    <span class="k">print</span><span class="p">(</span><span class="n">event</span><span class="p">)</span>

<span class="c1"># Stream metrics
</span><span class="k">for</span> <span class="n">event</span> <span class="ow">in</span> <span class="n">cli</span><span class="p">.</span><span class="n">stream_metrics</span><span class="p">():</span>
    <span class="k">print</span><span class="p">(</span><span class="n">event</span><span class="p">)</span>
</code></pre></div></div>

<p>The examples above are for a server running on localhost (how I’ve been using it) but of course
this could be a fully ceritified url. I think (for the time being) my favorite model is <a href="https://riverml.xyz/dev/api/cluster/DBSTREAM/" target="_blank">DBSTREAM</a> because it’s a clustering that doesn’t require me to define a number of clusters ahead of time,
and it’s been working quite well on some of my datasets.</p>

<p>And just for a quick glimpse, here is a visual of the cluster centroids for a subset of data that I’ve been testing with (after dimensionality reduction using TNSE). Indeed we see some structure!</p>

<div style="padding: 20px;">
 <img src="https://vsoch.github.io/assets/images/posts/online-ml/cluster-example.png" />
</div>

<h4 id="internal-client">Internal Client</h4>

<p>If you want to interact with django-river-ml internally, we have a client for that! You’d import this directly from
the module:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kn">from</span> <span class="nn">django_river_ml.client</span> <span class="kn">import</span> <span class="n">DjangoClient</span>
<span class="n">client</span> <span class="o">=</span> <span class="n">DjangoClient</span><span class="p">()</span>
</code></pre></div></div>

<p>That client is going to support the same kinds of functions that you might hit with an API call.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">client</span><span class="p">.</span><span class="n">models</span><span class="p">()</span>
<span class="p">[</span><span class="s">'milky-despacito'</span><span class="p">,</span> <span class="s">'tart-bicycle'</span><span class="p">]</span>

<span class="c1"># A learning example
</span><span class="n">client</span><span class="p">.</span><span class="n">learn</span><span class="p">(</span>
    <span class="n">model_name</span><span class="p">,</span>
    <span class="n">ground_truth</span><span class="o">=</span><span class="n">ground_truth</span><span class="p">,</span>
    <span class="n">prediction</span><span class="o">=</span><span class="n">prediction</span><span class="p">,</span>
    <span class="n">features</span><span class="o">=</span><span class="n">features</span><span class="p">,</span>
    <span class="n">identifier</span><span class="o">=</span><span class="n">identifier</span><span class="p">,</span>
<span class="p">)</span>
</code></pre></div></div>

<p>See <a href="https://vsoch.github.io/django-river-ml/getting_started/user-guide.html#interaction-from-inside-your-application" target="_blank">these pages</a> for more examples.</p>

<h2 id="why-should-i-care">Why should I care?</h2>

<p>Okay, here is the most important part of the post. Online machine learning is a different (and newer) paradigm for machine learning, and if
we can further develop the space we are going to be able to build a lot of cool applications using the tools! As someone who is strongly software engineer and likes to build things, this is a future I want to see. So if you have a problem that would be well supported by this paradigm, give it a shot! And please contribute to both river and django-river-ml - I can tell you from first-hand experience from a handful of issues and pull requests that the river community is welcoming, kind, and they have plans for making the library as awesome as it can be!  <a href="https://github.com/online-ml/river#-contributing" target="_blank">Here is their contribution</a> section. And if you are a Django person and want to contribute (or discuss features or changes for Django River ML), <a href="https://github.com/vsoch/django-river-ml/" target="_blank">I’d love to have you!</a>.</p>