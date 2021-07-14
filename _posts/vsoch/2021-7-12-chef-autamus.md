---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2021-07-12 19:30:00'
layout: post
original_url: https://vsoch.github.io//2021/chef-autamus/
title: Chef Autamus
---

<blockquote>
  <p>Chef Autamus, at your service!</p>
</blockquote>

<div style="margin: 20px;">
 <a href="https://github.com/autamus/chef-wasm/" target="_blank"><img src="https://raw.githubusercontent.com/autamus/chef-wasm/main/docs/img/chef-stash.png" /></a>
</div>

<p>Wouldn’t it be fun to build a custom software container with any number of packages available on <a href="https://autamus.io" target="_blank">autamus.io</a>?
<a href="https://github.com/alecbcs">@alecbcs</a> and I thought so too! Let’s walk through this fun project by asking and answering some questions.</p>

<h2 id="what-is-autamus">What is autamus?</h2>

<p>Autamus is:</p>

<blockquote>
  <p>A Semi-Autonomous Build System for Scientific Containers</p>
</blockquote>

<p>This means that we have an entire <a href="https://autamus.io/registry/" target="_blank">library of Docker containers</a> that build directly from spack packages. That’s right - you can pull a container with your favorite spack package without waiting for it to build from source! These same containers are also provided on <a href="https://singularityhub.github.io/singularity-hpc/" target="_blank">Singularity Registry HPC</a> to install on your local cluster with environment modules.</p>

<p>Autamus is much cooler than just a registry - it automatically detects changed package versions (not depending on spack, which does not do this and requires manual updating) and then opens a pull request with the package updates.  Here <a href="https://github.com/autamus/registry/pull/514" target="_blank">is an example update for LMOD</a>. The pull request with the update will also rebuild all package containers that depend on the library of interest. <a href="https://github.com/autamus/registry/actions/runs/987559728" target="_blank">Here</a> is an example doing that for Python.</p>

<h2 id="how-is-autamus-related-to-spack">How is Autamus related to spack?</h2>

<p>Autamus builds containers from spack packages, and maintains it’s own updated folder in the <a href="https://github.com/autamus/registry" target="_blank">registry</a>. 
The driver behind this is a bot named <a href="https://github.com/autamus/binoc" target="_blank">binoc</a> (short for binoculars) that can read any spack package file and then look for updates at whatever source it is derived from. Binoc then knows how to open a pull request with these updates, and the autamus registry uses
a <a href="https://github.com/autamus/builder" target="_blank">few</a> <a href="https://github.com/autamus/buildconfig" target="_blank">other</a> helper libraries
to build a new container along with containers for all dependencies. This functionality could easily be added to spack one day, but it comes at the cost of many more pull requests for maintainers to review, so it wouldn’t be a good decision at this point in time. However, all of the updated package files live in the registry, and could
be extended to spack some day.</p>

<p>Autamus serves a different use case than spack as it provides container bases (and tools around that related to containers) and shoud not be considered a competitor. Likely
you would use one or the other based on your use case.</p>

<h2 id="how-does-it-work">How does it work?</h2>

<p>Autamus Chef (wasm) is a web assembly application, meaning we’ve actually compiled a small <a href="https://github.com/autamus/chef-wasm" target="_blank">Go library</a>
that you can see in the repository linked there. The real driver of this is the <a href="https://github.com/autamus/chef" target="_blank">autamus/chef</a> library,
which is a command line client and library for generating these multi-stage build Dockerfiles from autamus packages. For the command line client we:</p>

<ol class="custom-counter">
<li>Read in a list of packages and versions from a chef.yaml file</li>
<li>Validate that they exist by looking for their manifests</li>
<li>Generate the Dockerfile, with each spack install being copied to a final install folder</li>
</ol>

<p>We then use these same functions in the chef wasm interface, except instead of requiring validation with an http request, we instead load
the known packages from the Autamus <a href="https://autamus.io/registry/library.json" target="_blank">library API</a> and allow you to select from
only those. We can quickly generate the Dockerfile any time you select or remove a new package. We generate the wasm interface with
a slightly different compile command to generate chef.wasm:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">GOOS</span><span class="o">=</span>js <span class="nv">GOARCH</span><span class="o">=</span>wasm go build <span class="nt">-o</span> chef.wasm<span class="p">;</span> <span class="nb">mv </span>chef.wasm docs/
</code></pre></div></div>

<p>Which as you can see above, we move into the docs folder where the static files for the site are. <a href="https://github.com/autamus/chef-wasm/tree/main/docs" target="_blank">Take a look</a> if you are interested in how it works! It’s just basic Javascript and styling. Then when you build and then load the container for the first time, since we’ve removed any previous spack database it rebuilds based on what is discovered in the spack install location. And that’s it!</p>

<h2 id="how-do-i-use-it">How do I use it?</h2>

<p>You start in the web interface:</p>

<div style="margin: 20px;">
 <a href="https://autamus.io/chef-wasm/" target="_blank"><img src="https://raw.githubusercontent.com/autamus/chef-wasm/main/docs/img/chef-wasm.png" /></a>
</div>

<p>And whatever packages are available in autamus (and their versions!) will appear as options in the dropdown. This
is possible because the autamus library serves a little JSON <a href="https://autamus.io/registry/library.json" target="_blank">library API</a> for you to use.
You can search and select packages in the search box, and as you add or remove packages a Dockerfile recipe is updated in the right of the screen.
You can then download the recipe, optionally add to version control, and build and interact with your container! Instructions are provided in the
<a href="https://github.com/autamus/chef-wasm#usage" target="_blank">README</a> of the repository and re-iterated here.</p>

<h3 id="1-build-the-container">1. Build the container</h3>

<p>You want to do the build in the same directory as your Dockerfile.
I like using <code class="language-plaintext highlighter-rouge">DOCKER_BUILDKIT</code> for more efficient builds:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ DOCKER_BUILDKIT</span><span class="o">=</span>1 docker build <span class="nt">-t</span> mystack <span class="nb">.</span>
</code></pre></div></div>

<h3 id="2-interact">2. Interact</h3>

<p>When you shell into your container, you will be in the spack root at <code class="language-plaintext highlighter-rouge">/opt/spack</code></p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>docker run <span class="nt">-it</span> mystack
</code></pre></div></div>

<p>And spack is available to you!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="c"># which spack</span>
/opt/spack/bin/spack

</code></pre></div></div>

<p>To see your installed software, use <code class="language-plaintext highlighter-rouge">spack find</code></p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>spack find
root@62b0820b3524:/opt/spack# spack find
<span class="o">==&gt;</span> 61 installed packages
<span class="nt">--</span> linux-ubuntu18.04-skylake / gcc@7.5.0 <span class="nt">------------------------</span>
abi-dumper@1.1       gdbm@1.19       ncurses@6.2             vtable-dumper@1.2
berkeley-db@18.1.40  gettext@0.21    perl@5.32.1             xz@5.2.5
binutils@2.36.1      libelf@0.8.13   readline@8.1            zlib@1.2.11
bzip2@1.0.8          libiconv@1.16   <span class="nb">tar</span>@1.34
elfutils@0.182       libxml2@2.9.10  universal-ctags@master

<span class="nt">--</span> linux-ubuntu18.04-x86_64 / gcc@7.5.0 <span class="nt">-------------------------</span>
bcftools@1.13        libmd@1.0.3          py-python-dateutil@2.8.1
berkeley-db@18.1.40  libpng@1.6.37        py-setuptools@50.3.2
bzip2@1.0.8          libunistring@0.9.10  py-six@1.15.0
curl@7.76.1          libxml2@2.9.10       python@3.9.5
expat@2.3.0          ncurses@6.2          python@3.9.6
freetype@2.10.4      openblas@0.3.15      qhull@2020.1
gdbm@1.19            openssl@1.1.1k       readline@8.1
gettext@0.21         patchelf@0.12        samtools@1.12
htslib@1.12          perl@5.35.0          sqlite@3.35.5
htslib@1.13          py-cycler@0.10.0     <span class="nb">tar</span>@1.34
libbsd@0.11.3        py-kiwisolver@1.1.0  util-linux-uuid@2.36.2
libffi@3.3           py-matplotlib@3.4.2  xz@5.2.5
libiconv@1.16        py-numpy@1.21.0      zlib@1.2.11
libidn2@2.3.0        py-pillow@8.0.0
libjpeg-turbo@2.0.6  py-pyparsing@2.4.7

</code></pre></div></div>

<p>And then to load any particular piece of software (meaning it gets added to your path) use <code class="language-plaintext highlighter-rouge">spack load</code>. Let’s load Python!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="c"># setup your environment</span>
<span class="nv">$ </span><span class="nb">.</span> /opt/spack/share/spack/setup-env.sh

<span class="c"># load Python</span>
root@62b0820b3524:/opt/spack# spack load python@3.9.5

<span class="c"># Did we load it?</span>
root@62b0820b3524:/opt/spack# which python
/opt/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/python-3.9.5-kykqwyent2svlspsvehqpu4xdgcp54z5/bin/python

/opt/spack# python <span class="nt">--version</span>
Python 3.9.5

</code></pre></div></div>
<p>And that’s it! I jumped on this fun little project because I love programming in Go,
and I’ll take every opportunity to try doing something in Go or wasm.
This is pretty new and experimental, so please <a href="https://github.com/autamus/chef-wasm/issues" target="_blank">open an issue</a>
if you run into any trouble. Right now we make available all autamus packages, and it
could be that we need to limit to a subset (e.g., with just one architecture) for this app.</p>