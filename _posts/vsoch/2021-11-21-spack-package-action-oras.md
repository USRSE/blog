---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2021-11-21 10:30:00'
layout: post
original_url: https://vsoch.github.io//2021/spack-package-action-oras/
title: Spack Package Action (with oras)
---

<p>This last week was <a href="https://sc21.supercomputing.org/" target="_blank">Supercomputing 21</a>, 
and I participated in the <a href="https://sc21.supercomputing.org/presentation/?id=bof170&amp;sess=sess410" target="_blank">Spack BOF</a>,
where “BOF” means “Birds of a Feather.” I guess dinosaurs are birds, or bird-ish? Feathers? Anyway, if you are looking for things that I talked about,
they were:</p>

<ol class="custom-counter">
  <li><a href="https://spack.github.io/packages" target="_blank">Spack Packages Site/API</a></li>
  <li><a href="https://github.com/spack/label-schema" target="_blank">Spack Label Schema</a></li>
  <li><a href="https://github.com/spack/spack-monitor" target="_blank">Spack Monitor</a></li>
  <li><a href="https://autamus.io" target="_blank">Autamus Container Builds</a></li>
  <li><a href="https://spack.readthedocs.io/en/latest/analyze.html" target="_blank">Spack Analyze</a></li>
</ol>

<p>But that’s not what I want to talk about today! There was some discussion about an idea (and I cannot quote because
I don’t remember the exact words) but it was something like this:</p>

<blockquote>
  <p>Something something… install spack packages from GitHub?</p>
</blockquote>

<p>I can’t remember the details, but it prompted me to (twice!) in the chat say “You mean, like Go?” because
I absolutely love that build system. Anyway, I went on my merry business that week. Until today, when this little
thought bubbled into my consciousness.</p>

<blockquote>
  <p>Can we build and install spack packages from GitHub?</p>
</blockquote>

<p>Hmm, I don’t know. Can we? With the talented <a href="https://github.com/alecbcs" target="_blank">@alecbcs</a> we were
able to put together to build spack containers, but in this case we would want not just a single GitHub action to build a container
deployed <em>alongside</em> a repository, but also an action to build a spack package and upload to GitHub packages as a binary cache.</p>

<h2 id="too-long-didnt-read">Too Long, Didn’t Read</h2>

<p>The short story is yes, these things are possible! I put together a set of GitHub actions called
<a href="https://github.com/vsoch/spack-package-action" target="_blank">spack-package-action</a> that 
handles the following:</p>

<ol class="custom-counter">
  <li><a href="https://github.com/vsoch/spack-package-action#install-spack" target="_blank">Installs Spack</a> with customization of branch/release, root, and even depth.</li>
  <li><a href="https://github.com/vsoch/spack-package-action#package-binary-build" target="_blank">Release binaries</a> build and (optionally) release spack binaries to GitHub packages</li>
  <li><a href="https://github.com/vsoch/spack-package-action#package-binary-build" target="_blank">Release containers</a> build and (optionally) deploy a container with spack packages to GitHub packages</li>
</ol>

<h2 id="how-does-it-work">How does it work?</h2>

<h3 id="github-workflows">GitHub Workflows</h3>

<p>Belold the power - of GitHub workflows! 🎉️ We can use GitHub <a href="https://docs.github.com/en/actions/creating-actions/creating-a-composite-action" target="_blank">composite actions</a> to assemble a structure of folders, each with an action.yml to define one of more steps, and a folder of scripts!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># This one builds a container, uses: vsoch/spack-package-action/container@main</span>
├── container
│   ├── action.yml
│   ├── default.yaml
│   └── scripts
│       ├── build.sh
│       ├── release.sh
│       └── set_root.sh

<span class="c"># This one installs spack, uses: vsoch/spack-package-action/install@main</span>
├── <span class="nb">install</span>
│   ├── action.yml
│   └── scripts
│       └── install.sh

<span class="c"># This one builds package binaries, uses: vsoch/spack-package-action/package@main</span>
└── package
    ├── action.yml
    └── scripts
        ├── build.sh
        ├── release.sh
        └── set_root.sh

</code></pre></div></div>

<p>Each folder above shows a hodge-podge of bash scripts and yaml that make this workflow work!
Then, you might have a workflow that include any of these jobs:</p>

<div class="language-yaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="na">name</span><span class="pi">:</span> <span class="s">Spack Package Building</span>
<span class="na">on</span><span class="pi">:</span>
  <span class="na">pull_request</span><span class="pi">:</span> <span class="pi">[]</span>
  <span class="na">push</span><span class="pi">:</span>
    <span class="na">branches</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">main</span> 
 
<span class="na">jobs</span><span class="pi">:</span>  
  <span class="na">install-spack</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Install Spack</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v2</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Spack</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">vsoch/spack-package-action/install@main</span>

  <span class="c1"># This builds spack binaries for the build cache for a package of choice</span>
  <span class="na">build-binaries</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">permissions</span><span class="pi">:</span>
      <span class="na">packages</span><span class="pi">:</span> <span class="s">write</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Build Package Binaries</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v2</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Build Spack Package</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">vsoch/spack-package-action/package@main</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">package</span><span class="pi">:</span> <span class="s">zlib</span>
          <span class="na">token</span><span class="pi">:</span> <span class="s">${{ secrets.GITHUB_TOKEN }}</span>
          <span class="na">deploy</span><span class="pi">:</span> <span class="s">${{ github.event_name != 'pull_request' }}</span>
          
  <span class="c1"># This builds a spack container given a spack.yaml</span>
  <span class="na">build-container-spack-yaml</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">permissions</span><span class="pi">:</span>
      <span class="na">packages</span><span class="pi">:</span> <span class="s">write</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Build Package Container spack.yaml</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v2</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Build Spack Container</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">vsoch/spack-package-action/container@main</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">spack_yaml</span><span class="pi">:</span> <span class="s">spack/spack.yaml</span>
          <span class="na">token</span><span class="pi">:</span> <span class="s">${{ secrets.GITHUB_TOKEN }}</span>
          <span class="na">deploy</span><span class="pi">:</span> <span class="s">${{ github.event_name != 'pull_request' }}</span>

  <span class="c1"># This builds a spack container for a package of choice</span>
  <span class="na">build-container</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">permissions</span><span class="pi">:</span>
      <span class="na">packages</span><span class="pi">:</span> <span class="s">write</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Build Package Container</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v2</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Build Spack Container</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">vsoch/spack-package-action/container@main</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">package</span><span class="pi">:</span> <span class="s">zlib</span>
          <span class="na">token</span><span class="pi">:</span> <span class="s">${{ secrets.GITHUB_TOKEN }}</span>
          <span class="na">deploy</span><span class="pi">:</span> <span class="s">${{ github.event_name != 'pull_request' }}</span>
</code></pre></div></div>

<p>Here are some notes for the different actions:</p>

<h3 id="install">Install</h3>

<p>This is an easy way to install spack! Along with a list of comma separated “repos” to add extra repos (GitHub urls) you can
specify a branch/release, a custom root (defaults to /opt/spack) and a boolean “full_clone.” By default we clone with a depth of 1
for a faster clone, but if you need the full git history you’d want to set this to true. And the really cool thing about this step,
and composite actions, is that I wound up using it in the <em>other</em> composite actions. I wasn’t sure if GitHub would limit me to the scope
of the action root, but what I could actually do is just get the directory name (the root of the repository) for a context one level up,
and then specify to use install. For example, this is within the package action.yml:</p>

<div class="language-yaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nn">...</span>
    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Set Root Directory</span>
      <span class="na">env</span><span class="pi">:</span>
        <span class="na">ACTION_PATH</span><span class="pi">:</span> <span class="s">${{ github.action_path }}</span>
      <span class="na">run</span><span class="pi">:</span> <span class="s">${{ github.action_path }}/scripts/set_root.sh</span>
      <span class="na">shell</span><span class="pi">:</span> <span class="s">bash</span>
</code></pre></div></div>

<p>The script “set_root.sh” is very silly, it sets the root as the parent of our current directory (shortened for brevity):</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="c">#!/bin/bash</span>

<span class="nv">ACTION_ROOT</span><span class="o">=</span><span class="si">$(</span><span class="nb">dirname</span> <span class="nv">$ACTION_PATH</span><span class="si">)</span>
<span class="nb">echo</span> <span class="s2">"ACTION_ROOT=</span><span class="k">${</span><span class="nv">ACTION_ROOT</span><span class="k">}</span><span class="s2">"</span> <span class="o">&gt;&gt;</span> <span class="nv">$GITHUB_ENV</span>
</code></pre></div></div>

<p>And then in the next step, we run the install script from that context, and with our variables of choice.</p>

<div class="language-yaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nn">...</span>
    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Spack and Dependencies</span>
      <span class="na">env</span><span class="pi">:</span>
        <span class="na">INPUT_BRANCH</span><span class="pi">:</span> <span class="s">${{ inputs.branch }}</span>
        <span class="na">INPUT_RELEASE</span><span class="pi">:</span> <span class="s">${{ inputs.release }}</span>
        <span class="na">INPUT_REPOS</span><span class="pi">:</span> <span class="s">${{ inputs.repos }}</span>
        <span class="na">INPUT_ROOT</span><span class="pi">:</span> <span class="s">/opt/spack</span>
      <span class="na">run</span><span class="pi">:</span> <span class="s">${{ env.ACTION_ROOT }}/install/scripts/install.sh</span>
      <span class="na">shell</span><span class="pi">:</span> <span class="s">bash</span>
</code></pre></div></div>

<p>Pretty neat, yeah? With this approach I can use this action in the install folder in the other two folders, container and package respectively.</p>

<h4 id="package">Package</h4>

<p>The package builder is essentially going to build you the same set of files you’d get by adding something to the spack build cache.
You can either give it a package name, or a local package.py file for an existing (or not existing) package.
The resulting uploaded package is going to have a name that corresponds to what we’d keep in the build cache, my idea being we
will eventually be able to query a packages endpoint to find files of this format.
If this file is discovered you’ll essentially use <code class="language-plaintext highlighter-rouge">spack develop</code> to install from that root. Note that I haven’t tested
this fully and should make some time soon, but please open an issue if you try it out and run into trouble!</p>

<p>The interestingIt’s going to have a build hash that depends on the GitHub runner to some extent, and you’ll want to add <code class="language-plaintext highlighter-rouge">flags</code> as an action argument to get
better control of your package target, compiler, etc (interestingly, the runners come with quite a few ready to go!).</p>

<h3 id="container">Container</h3>

<p>The container builder is similar to the package action, but you can also provide a “spack_yaml” instead of a package to build
a container from it with “spack containerize.” I intended to add a bunch of spack package and compiler labels, but realized
this would be better done upstream (within the file, before it’s built to avoid the double build). You can also specify the different
branch/release of spack to use for containerize, and a tag that will default to latest if not set.</p>

<h3 id="packages-generated">Packages Generated</h3>

<p>What does this result in? On your deploy (release) triggers (when deploy is true), you’ll build and deploy packages
not only for a container that has spack, but also for a binary for a spack build cache!
You can take a look at the example packages <a href="https://github.com/vsoch?tab=packages&amp;repo_name=spack-package-action" target="_blank">here</a>.
This is done by way of the magic of <a href="https://oras.land" target="_blank">oras</a>. Let’s take a look at how we might pull an oras artifact:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>oras pull ghcr.io/vsoch/spack-package-action/linux-ubuntu20.04-broadwell-gcc-10.3.0-zlib-1.2.11-5vlodp7yawk5elx4dfhnpzmpg743fwv3.spack:d115bcc4
Downloaded 5ee1f2ed8b80 spack-package.tar.gz
Pulled ghcr.io/vsoch/spack-package-action/linux-ubuntu20.04-broadwell-gcc-10.3.0-zlib-1.2.11-5vlodp7yawk5elx4dfhnpzmpg743fwv3.spack:d115bcc4
Digest: sha256:102901abeb89676e466184df1a87a23916febb465f688e5f4c12174263b98f9b
</code></pre></div></div>

<p>What did we pull?</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span><span class="nb">ls
</span>container  <span class="nb">install  </span>opt  package  README.md  spack  spack-package.tar.gz
</code></pre></div></div>

<p>Let’s look inside!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span><span class="nb">tar</span> <span class="nt">-xzvf</span> spack-package.tar.gz 
build_cache/
build_cache/linux-ubuntu20.04-broadwell/
build_cache/linux-ubuntu20.04-broadwell/gcc-10.3.0/
build_cache/linux-ubuntu20.04-broadwell/gcc-10.3.0/zlib-1.2.11/
build_cache/linux-ubuntu20.04-broadwell/gcc-10.3.0/zlib-1.2.11/linux-ubuntu20.04-broadwell-gcc-10.3.0-zlib-1.2.11-5vlodp7yawk5elx4dfhnpzmpg743fwv3.spack
build_cache/linux-ubuntu20.04-broadwell-gcc-10.3.0-zlib-1.2.11-5vlodp7yawk5elx4dfhnpzmpg743fwv3.spec.json
build_cache/_pgp/
build_cache/_pgp/03335A5FDBD232812567D91E07AA94F305E9B077.pub
</code></pre></div></div>

<p>Wow! So oras pull resulted in the “spack-package.tar.gz” to be in the present working directory, which has the contents
of a build cache with one package. This is exciting. Let’s chat about why next.</p>

<h2 id="why-is-this-exciting">Why is this exciting?</h2>

<h3 id="build-cache-from-github">Build Cache from GitHub</h3>

<p>The spack build cache is a powerful thing. When you go through the steps to create your own or connect to an existing one,
it speeds up builds a lot! But historically I’ve just found this whole set up steps hard to do. If I wanted to go crazy and build a
bunch of stuff and populate a cache, I’d have to give AWS some of my moolas, or use someone else’s moolahs. I’d rather not do that if I don’t need
to. So this thought occurred to me earlier today:</p>

<blockquote>
  <p>Why not GitHub packages?</p>
</blockquote>

<p>And yes, why not! As we’ve seen from the above, we are able to generate a programatically accessible build cache (artifacts) for a spack package,
and store in GitHub packages. If this is an interesting or desired direction, the next step would be for us to put our heads together and
decide how we might want this to work. Should we have different automated caches with different families of packages? Should we store some more
permanent key alongside the repository instead of generating to sign on the fly? There are lots of cool questions to ask, so if you are interested
in discussion please <a href="https://github.com/vsoch/spack-package-action/issues" target="_blank">open an issue</a> to discuss or ping me on Spack slack.</p>

<h3 id="spack-containers">Spack Containers</h3>

<p>I’ve shared many times before that I really like <a href="https://autamus.io" target="_blank">autamus</a>. How cool would it be to
empower people to build spack packages, via their own repos, to produce containers for others to use? The idea that one repository == one spack package is relly neat, especially if the repository wants to have tighter control over releasing some kind of artifact with spack. One thing I wish we could
do better is always testing new versions of packages, and for packages provided alongside GitHub repos, and alongside the code they provide,
we could always have this testing.</p>

<h3 id="questions-for-discussion">Questions for Discussion</h3>

<p>So is this the only way? Of course not! I literally came up with this during an afternoon, and I’d say everything is subject to change.
Here are some questions I’d like to propose to you, dear reader.</p>

<ol>
  <li>Should we add an ability to install a spack binary from GitHub packages (akin to an on the fly build cache?)</li>
  <li>What should the namespace of the package be in GitHub packages? Since it’s technically one package in a build cache, we could name based on the build hash, but arguably there could be more than one.</li>
  <li>Should we preserve the entire thing .tar.gz-ed or just the .spack archive?</li>
  <li>Should we have a way to keep a persistent gpg key to sign packages?</li>
  <li>What about <a href="https://github.com/spack/label-schema">spack container labels</a>? How should we include here or extent?</li>
  <li>Should we add these labels to spack containerize instead (I think so)</li>
</ol>