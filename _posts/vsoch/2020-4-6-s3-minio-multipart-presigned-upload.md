---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2020-04-06 16:01:00'
layout: post
original_url: https://vsoch.github.io//2020/s3-minio-multipart-presigned-upload/
title: Pre-signed MultiPart Uploads with Minio
---

<p>Have you ever stumbled into working on something challenging, not in a “solve this proof”
sort of way, but “figure out the interaction of these systems in the context of this API
and tool” and found it so intoxicatingly wonderful to work on that you can’t stop?
This happened to me this previous weekend, actually starting on Friday. I was working
on adding a <a href="https://github.com/singularityhub/sregistry/pull/297" target="_blank">Minio backend for storage</a>
for <a href="https://singularityhub.github.io/sregistry/" target="_blank">Singularity Registry Server</a>
because many users were, despite efforts to use the <a href="https://vsoch.github.io/2018/django-nginx-upload/" target="_blank">nginx-upload</a> module, still running into issues with large uploads. I had recently noticed that the Singularity
<a href="https://github.com/sylabs/scs-library-client/blob/30f9b6086f9764e0132935bcdb363cc872ac639d/client/push.go" target="_blank">scs-library-client</a> 
was pinging a “_multipoint” endpoint that my server was 
<a href="https://github.com/singularityhub/sregistry/pull/283" target="_blank">not prepared for</a>, and the quick
fix there was to add the endpoint and have it return 404 as a message “Sorry, we don’t support multipart uploads!”
But as time passed on I wondered - why can’t I support them? This led me back to trying to
integrate some kind of <a href="https://github.com/smartfile/django-transfer/issues/9#issuecomment-607976246" target="_blank">nginx helper</a>
to handle multipart uploads, but it proved to be too different than the <a href="https://docs.aws.amazon.com/AmazonS3/latest/dev/mpuoverview.html" target="_blank">Amazon Multipart Upload</a> protocol, which was further challenging because I needed to wrap those functions
in something else to generate the pre-signed urls.</p>

<h2 id="minio-to-the-rescue">Minio to the Rescue!</h2>

<p>I remembered when I was implementing Singularity Registry Client to have <a href="https://singularityhub.github.io/sregistry-cli/client-s3" target="_blank">s3 support</a> I had used a storage server called <a href="https://min.io" target="_blank">Minio</a>,
and other than having an elegant bird paper clip logo</p>

<div style="padding: 20px;">
   <img src="https://vsoch.github.io/assets/images/posts/minio/browser.png" />
</div>

<p>(who now is much more badass by the way)</p>

<div style="padding: 20px;">
   <img src="https://vsoch.github.io/assets/images/posts/minio/badass-bird.png" />
</div>

<p>and it provided a really elegant, open source solution to host your own “S3-like” storage (this is my
understanding at least).</p>

<h3 id="1-adding-the-minio-container">1. Adding the Minio Container</h3>

<p>I was very easily able to (in just an hour or two) 
<a href="https://github.com/singularityhub/sregistry/pull/297" target="blank"> add a Minio backend for storage</a>,
meaning that the minio Docker container was added to the “docker-compose.yml”</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>minio:
  image: minio/minio
  volumes:
    - ./minio-images:/images
  env_file:
   - ./.minio-env
  ports:
   - "9000:9000"  
  command: ["server", "images"]
</code></pre></div></div>

<p>and notice that it binds a .minio-env file that provides the key and secret, and the
images are bound to our host so if the minio container goes away we don’t lose them.
The minio environment secrets are also mapped to the main uwsgi container with the Django application
since we will be instantiating clients from in there.</p>

<h3 id="2-installing-mc-for-management">2. Installing mc for management</h3>

<p>I quickly found that <code class="language-plaintext highlighter-rouge">docker-compose logs minio</code> didn’t show many meaningful logs.
I found it very useful to shell into the minio container, and install “mc,” which
is a command line client for minio:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>docker <span class="nb">exec</span> <span class="nt">-it</span> sregistry_minio_1 bash
wget https://dl.min.io/client/mc/release/linux-amd64/mc
<span class="nb">chmod</span> +x mc
./mc <span class="nt">--help</span>
</code></pre></div></div>

<p>I was able to add my server as host to reference it as “myminio”</p>
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>./mc config host add myminio http://127.0.0.1:9000 <span class="nv">$MINIO_ACCESS_KEY</span> <span class="nv">$MINIO_SECRET_KEY</span>
Added <span class="sb">`</span>myminio<span class="sb">`</span> successfully.
</code></pre></div></div>

<p>Verify that it was added:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>./mc config host <span class="nb">ls</span>
</code></pre></div></div>
<p>And then leave this command hanging in a window to show output logs.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>./mc admin trace <span class="nt">-v</span> myminio
</code></pre></div></div>

<p>This is hugely important for development, you wouldn’t know what is going on
with Minio otherwise.</p>

<h2 id="singularity-registry-server-views">Singularity Registry Server Views</h2>

<p>On the backend, in the configuration for Singularity Registry Server I give the
(developer) user full control over the time to allow signed URLs, using
SSL, and of course the internal and external servers.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="n">MINIO_SERVER</span> <span class="o">=</span> <span class="s">"minio:9000"</span>  <span class="c1"># Internal to sregistry
</span><span class="n">MINIO_EXTERNAL_SERVER</span> <span class="o">=</span> <span class="p">(</span>
    <span class="s">"127.0.0.1:9000"</span>  <span class="c1"># minio server for Singularity to interact with
</span><span class="p">)</span>
<span class="n">MINIO_BUCKET</span> <span class="o">=</span> <span class="s">"sregistry"</span>
<span class="n">MINIO_SSL</span> <span class="o">=</span> <span class="bp">False</span>  <span class="c1"># use SSL for minio
</span><span class="n">MINIO_SIGNED_URL_EXPIRE_MINUTES</span> <span class="o">=</span> <span class="mi">5</span>
<span class="n">MINIO_REGION</span> <span class="o">=</span> <span class="s">"us-east-1"</span>
</code></pre></div></div>

<blockquote>
  <p>What the heck, internal and external server?</p>
</blockquote>

<p>Yes! Remember that the Singularity client sees the minio container as <code class="language-plaintext highlighter-rouge">127.0.0.1:9000</code> 
(on localhost) but from inside the uwsgi container, we see it as <code class="language-plaintext highlighter-rouge">minio:9000</code>. You <em>could</em>
adopt a solution that adds minio as a hostname to “/etc/hosts” but my preference was not
to require that. For creating the clients, this was actually really quick to do!</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="kn">from</span> <span class="nn">shub.settings</span> <span class="kn">import</span> <span class="p">(</span>
    <span class="n">MINIO_SERVER</span><span class="p">,</span>
    <span class="n">MINIO_EXTERNAL_SERVER</span><span class="p">,</span>
    <span class="n">MINIO_BUCKET</span><span class="p">,</span>
    <span class="n">MINIO_REGION</span><span class="p">,</span>
    <span class="n">MINIO_SSL</span><span class="p">,</span>
<span class="p">)</span>
<span class="kn">from</span> <span class="nn">minio</span> <span class="kn">import</span> <span class="n">Minio</span>

<span class="kn">import</span> <span class="nn">os</span>

<span class="n">minioClient</span> <span class="o">=</span> <span class="n">Minio</span><span class="p">(</span>
    <span class="n">MINIO_SERVER</span><span class="p">,</span>
    <span class="n">region</span><span class="o">=</span><span class="n">MINIO_REGION</span><span class="p">,</span>
    <span class="n">access_key</span><span class="o">=</span><span class="n">os</span><span class="o">.</span><span class="n">environ</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">"MINIO_ACCESS_KEY"</span><span class="p">),</span>
    <span class="n">secret_key</span><span class="o">=</span><span class="n">os</span><span class="o">.</span><span class="n">environ</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">"MINIO_SECRET_KEY"</span><span class="p">),</span>
    <span class="n">secure</span><span class="o">=</span><span class="n">MINIO_SSL</span><span class="p">,</span>
<span class="p">)</span>

<span class="n">minioExternalClient</span> <span class="o">=</span> <span class="n">Minio</span><span class="p">(</span>
    <span class="n">MINIO_EXTERNAL_SERVER</span><span class="p">,</span>
    <span class="n">region</span><span class="o">=</span><span class="n">MINIO_REGION</span><span class="p">,</span>
    <span class="n">access_key</span><span class="o">=</span><span class="n">os</span><span class="o">.</span><span class="n">environ</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">"MINIO_ACCESS_KEY"</span><span class="p">),</span>
    <span class="n">secret_key</span><span class="o">=</span><span class="n">os</span><span class="o">.</span><span class="n">environ</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">"MINIO_SECRET_KEY"</span><span class="p">),</span>
    <span class="n">secure</span><span class="o">=</span><span class="n">MINIO_SSL</span><span class="p">,</span>
<span class="p">)</span>

<span class="k">if</span> <span class="ow">not</span> <span class="n">minioClient</span><span class="o">.</span><span class="n">bucket_exists</span><span class="p">(</span><span class="n">MINIO_BUCKET</span><span class="p">):</span>
    <span class="n">minioClient</span><span class="o">.</span><span class="n">make_bucket</span><span class="p">(</span><span class="n">MINIO_BUCKET</span><span class="p">)</span>
</code></pre></div></div>

<p>When the server starts, we create one for each of internal and external endpoints, and
also create the bucket if it doesn’t exist. This is done entirely with
<a href="https://github.com/minio/minio-py/" target="_blank">minio-py</a>.
And then we can use the external client to generate a presigned PUT url
to return to Singularity:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="n">storage</span> <span class="o">=</span> <span class="n">container</span><span class="o">.</span><span class="n">get_storage</span><span class="p">()</span>
<span class="n">url</span> <span class="o">=</span> <span class="n">minioExternalClient</span><span class="o">.</span><span class="n">presigned_put_object</span><span class="p">(</span>
    <span class="n">MINIO_BUCKET</span><span class="p">,</span>
    <span class="n">storage</span><span class="p">,</span>
    <span class="n">expires</span><span class="o">=</span><span class="n">timedelta</span><span class="p">(</span><span class="n">minutes</span><span class="o">=</span><span class="n">MINIO_SIGNED_URL_EXPIRE_MINUTES</span><span class="p">),</span>
<span class="p">)</span>
</code></pre></div></div>

<p>and do the same thing to retrieve a pre-signed url to GET (or download) it:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="n">storage</span> <span class="o">=</span> <span class="n">container</span><span class="o">.</span><span class="n">get_storage</span><span class="p">()</span>
<span class="n">url</span> <span class="o">=</span> <span class="n">minioExternalClient</span><span class="o">.</span><span class="n">presigned_get_object</span><span class="p">(</span>
    <span class="n">MINIO_BUCKET</span><span class="p">,</span>
    <span class="n">storage</span><span class="p">,</span>
    <span class="n">expires</span><span class="o">=</span><span class="n">timedelta</span><span class="p">(</span><span class="n">minutes</span><span class="o">=</span><span class="n">MINIO_SIGNED_URL_EXPIRE_MINUTES</span><span class="p">),</span>
<span class="p">)</span>
<span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
</code></pre></div></div>

<p>I’m not showing these calls in the context of their functions - you can look
at the full code in the <a href="https://github.com/singularityhub/sregistry/pull/297" target="blank"> pull request </a> to get this. The main thing to understand is that (a very basic flow) for the legacy upload endpoint is:</p>

<ol class="custom-counter">
   <li>Singularity looks for _multipart endpoint, 404 defaults to <a href="https://github.com/sylabs/scs-library-client/blob/master/client/push.go#L287" target="_blank">the legacy endpoint</a></li>
   <li>In our case, the legacy endpoint now provides a presigned URL to PUT an image file</li>
   <li>The PUT request is done with Minio storage now instead of the nginx upload module</li>
</ol>

<p>And now with Minio, we’ve greatly improved this workflow by adding an extra layer
of having signed URLs, and having the uploads and GETs go directly to the minio container.
Before we were using the nginx upload module to upload directly via nginx, and then
have a callback that then pings the uwsgi container to validate the upload and finish things up.
We now have a huge improvement, I think, because the main uwsgi django server isn’t taking the brunt of load
to upload and download containers, and validation happens before any files are transferred. 
A cluster could much more easily deploy some kind of (separate) and scaled Minio cluster and then still use Singularity Registry
Server. I haven’t done this yet, but I suspect that we are allowing for better scaling, hooray!</p>

<h2 id="filesystem">FileSystem</h2>

<p>On the user filesystem, if we bind the minio data folder (which by the way can 
be more than one) we can see the images that we pushed</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>$ tree minio-images/
minio-images/
└── sregistry
    └── test
        ├── big:sha256.92278b7c046c0acf0952b3e1663b8abb819c260e8a96705bad90833d87ca0874
        └── container:sha256.c8dea5eb23dec3c53130fabea451703609478d7d75a8aec0fec238770fd5be6e
</code></pre></div></div>

<h2 id="multipart-upload">Multipart Upload</h2>

<p>The above was quick, but it didn’t add multipart uploads. Hey, it was only Friday afternoon!
This is where it got fun and challenging, and took me most of Saturday, all of Sunday, and half of Monday.
A multipart upload would look something like this:</p>

<h3 id="1-singularity-asks-for-multipart">1. Singularity Asks for Multipart</h3>
<p>Singularity first looks for the <code class="language-plaintext highlighter-rouge">_multipart</code> endpoint, specifically making a POST
to a URL matching the pattern “^v2/imagefile/(?P.+?)/_multipart?$"
and provides an upload_id (actually this is the id associated with a container
object that is first generated, but we don't need to talk about this detail! The
upload_id is passed around between these various endpoints to always be able
to retrieve the correct container instance. Anyway, here is pseudocode for the steps
that are taken to start an upload. Note that this is a POST to the _multipart endpoint.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">def</span> <span class="nf">post</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">request</span><span class="p">,</span> <span class="n">upload_id</span><span class="p">):</span>
    <span class="s">"""In a post, the upload_id will be the container id.
    """</span>
    <span class="k">print</span><span class="p">(</span><span class="s">"POST RequestMultiPartPushImageFileView"</span><span class="p">)</span>

    <span class="c1"># 1. Handle authorization, if the request doesn't have a token, return 403
</span>    <span class="c1"># 2. If the config setting MINIO_MULTIPART_UPLOAD is False, return 404 to default to legacy
</span>    <span class="c1"># 3. Get the container instance, return 404 if not found
</span>    <span class="c1"># 4. get the filesize from the body request, calculate the number of chunks and max upload size
</span>    <span class="c1"># 5. Create the multipart upload!
</span></code></pre></div></div>

<p>For that last step (5), this is the first time we need to interact with another API
for minio. However, minio-py doesn’t support generating anything for pre-signed
multipart, so in this case we need to interact with s3 via <a href="https://github.com/boto/boto3" target="_blank">boto3</a>. 
Again, we need to create an internal (minio:9000) and external (127.0.0.1:9000) client:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="kn">from</span> <span class="nn">boto3</span> <span class="kn">import</span> <span class="n">Session</span>
<span class="kn">from</span> <span class="nn">botocore.client</span> <span class="kn">import</span> <span class="n">Config</span>

<span class="kn">from</span> <span class="nn">shub.settings</span> <span class="kn">import</span> <span class="p">(</span>
    <span class="n">MINIO_SERVER</span><span class="p">,</span>
    <span class="n">MINIO_EXTERNAL_SERVER</span><span class="p">,</span>
    <span class="n">MINIO_BUCKET</span><span class="p">,</span>
    <span class="n">MINIO_REGION</span><span class="p">,</span>
    <span class="n">MINIO_SSL</span><span class="p">,</span>
<span class="p">)</span>
<span class="kn">import</span> <span class="nn">os</span>

<span class="n">MINIO_HTTP_PREFIX</span> <span class="o">=</span> <span class="s">"https://"</span> <span class="k">if</span> <span class="n">MINIO_SSL</span> <span class="k">else</span> <span class="s">"http://"</span>

<span class="o">...</span>

<span class="n">session</span> <span class="o">=</span> <span class="n">Session</span><span class="p">(</span>
    <span class="n">aws_access_key_id</span><span class="o">=</span><span class="n">os</span><span class="o">.</span><span class="n">environ</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">"MINIO_ACCESS_KEY"</span><span class="p">),</span>
    <span class="n">aws_secret_access_key</span><span class="o">=</span><span class="n">os</span><span class="o">.</span><span class="n">environ</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">"MINIO_SECRET_KEY"</span><span class="p">),</span>
    <span class="n">region_name</span><span class="o">=</span><span class="n">MINIO_REGION</span><span class="p">,</span>
<span class="p">)</span>


<span class="c1"># https://github.com/boto/boto3/blob/develop/boto3/session.py#L185
</span><span class="n">s3</span> <span class="o">=</span> <span class="n">session</span><span class="o">.</span><span class="n">client</span><span class="p">(</span>
    <span class="s">"s3"</span><span class="p">,</span>
    <span class="n">verify</span><span class="o">=</span><span class="bp">False</span><span class="p">,</span>
    <span class="n">use_ssl</span><span class="o">=</span><span class="n">MINIO_SSL</span><span class="p">,</span>
    <span class="n">endpoint_url</span><span class="o">=</span><span class="n">MINIO_HTTP_PREFIX</span> <span class="o">+</span> <span class="n">MINIO_SERVER</span><span class="p">,</span>
    <span class="n">region_name</span><span class="o">=</span><span class="n">MINIO_REGION</span><span class="p">,</span>
    <span class="n">config</span><span class="o">=</span><span class="n">Config</span><span class="p">(</span><span class="n">signature_version</span><span class="o">=</span><span class="s">"s3v4"</span><span class="p">,</span> <span class="n">s3</span><span class="o">=</span><span class="p">{</span><span class="s">"addressing_style"</span><span class="p">:</span> <span class="s">"path"</span><span class="p">}),</span>
<span class="p">)</span>

<span class="c1"># signature_versions
# https://github.com/boto/botocore/blob/master/botocore/auth.py#L846
</span><span class="n">s3_external</span> <span class="o">=</span> <span class="n">session</span><span class="o">.</span><span class="n">client</span><span class="p">(</span>
    <span class="s">"s3"</span><span class="p">,</span>
    <span class="n">use_ssl</span><span class="o">=</span><span class="n">MINIO_SSL</span><span class="p">,</span>
    <span class="n">region_name</span><span class="o">=</span><span class="n">MINIO_REGION</span><span class="p">,</span>
    <span class="n">endpoint_url</span><span class="o">=</span><span class="n">MINIO_HTTP_PREFIX</span> <span class="o">+</span> <span class="n">MINIO_EXTERNAL_SERVER</span><span class="p">,</span>
    <span class="n">verify</span><span class="o">=</span><span class="bp">False</span><span class="p">,</span>
    <span class="n">config</span><span class="o">=</span><span class="n">Config</span><span class="p">(</span><span class="n">signature_version</span><span class="o">=</span><span class="s">"s3v4"</span><span class="p">,</span> <span class="n">s3</span><span class="o">=</span><span class="p">{</span><span class="s">"addressing_style"</span><span class="p">:</span> <span class="s">"path"</span><span class="p">}),</span>
<span class="p">)</span>
</code></pre></div></div>

<p>That might look trivial or straight forward in practice, but it was tricky
figuring out that I needed to provide a custom configuration to the clients,
specify the signature version to match minio, and also use “path” (actually
I think it might have worked without this, but I didn’t remove it). I am sure,
however, that “addressing_style” as “virtual” didn’t work, and “auto” I’m not
sure if it would have defaulted to path. Once we have these clients,
in our view to start the upload, we can generate the request!</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1"># Create the multipart upload
</span><span class="n">res</span> <span class="o">=</span> <span class="n">s3</span><span class="o">.</span><span class="n">create_multipart_upload</span><span class="p">(</span><span class="n">Bucket</span><span class="o">=</span><span class="n">MINIO_BUCKET</span><span class="p">,</span> <span class="n">Key</span><span class="o">=</span><span class="n">storage</span><span class="p">)</span>
<span class="n">upload_id</span> <span class="o">=</span> <span class="n">res</span><span class="p">[</span><span class="s">"UploadId"</span><span class="p">]</span>
<span class="k">print</span><span class="p">(</span><span class="s">"Start multipart upload </span><span class="si">%</span><span class="s">s"</span> <span class="o">%</span> <span class="n">upload_id</span><span class="p">)</span>
</code></pre></div></div>

<p>All we really need from there is the uploadID, which we then return to
the calling Singularity client that is looking for the uploadID,
total parts, and size for each part.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">data</span> <span class="o">=</span> <span class="p">{</span>
    <span class="s">"uploadID"</span><span class="p">:</span> <span class="n">upload_id</span><span class="p">,</span>
    <span class="s">"totalParts"</span><span class="p">:</span> <span class="n">total_parts</span><span class="p">,</span>
    <span class="s">"partSize"</span><span class="p">:</span> <span class="n">max_size</span><span class="p">,</span>
<span class="p">}</span>
<span class="k">return</span> <span class="n">Response</span><span class="p">(</span><span class="n">data</span><span class="o">=</span><span class="p">{</span><span class="s">"data"</span><span class="p">:</span> <span class="n">data</span><span class="p">},</span> <span class="n">status</span><span class="o">=</span><span class="mi">200</span><span class="p">)</span>
</code></pre></div></div>

<p>Singularity then gets a 200 response with this data, and hooray! This means that
the server supports multipart upload, continue!”</p>

<h2 id="2-singularity-asks-for-signed-urls">2. Singularity Asks for Signed URLS</h2>

<p>After the 200 response, Singularity dutifully iterates through chunks of the file,
and for each part uses the <a href="https://github.com/sylabs/scs-library-client/blob/5d0614e4bddff1b2231efe79609f9f177bec8014/client/push.go#L473" target="_blank">multipart upload part</a> function to calculate a sha256 sum, and send the part number,
part size, sha256sum, and upload id back to Singularity Registry Server. It’s actually the
same endpoint as before (ending in _multipart) but this time with a PUT request.
The reason is because Singularity is saying “Hey, here is information about the part, 
can you give me a signed URL?” And this is where things got tricky.
I first tried generating a signed URL with the same s3 client as I was supposed to do:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="c1"># Generate pre-signed URLS for external client (lookup by part number)
</span><span class="n">signed_url</span> <span class="o">=</span> <span class="n">s3_external</span><span class="o">.</span><span class="n">generate_presigned_url</span><span class="p">(</span>
    <span class="n">ClientMethod</span><span class="o">=</span><span class="s">"upload_part"</span><span class="p">,</span>
    <span class="n">HttpMethod</span><span class="o">=</span><span class="s">"PUT"</span><span class="p">,</span>
    <span class="n">Params</span><span class="o">=</span><span class="p">{</span>
        <span class="s">"Bucket"</span><span class="p">:</span> <span class="n">MINIO_BUCKET</span><span class="p">,</span>
        <span class="s">"Key"</span><span class="p">:</span> <span class="n">storage</span><span class="p">,</span>
        <span class="s">"UploadId"</span><span class="p">:</span> <span class="n">upload_id</span><span class="p">,</span>
        <span class="s">"PartNumber"</span><span class="p">:</span> <span class="n">part_number</span><span class="p">,</span>
        <span class="s">"ContentLength"</span><span class="p">:</span> <span class="n">content_size</span><span class="p">,</span>
        <span class="p">},</span>
    <span class="n">ExpiresIn</span><span class="o">=</span><span class="n">timedelta</span><span class="p">(</span><span class="n">minutes</span><span class="o">=</span><span class="n">MINIO_SIGNED_URL_EXPIRE_MINUTES</span><span class="p">)</span><span class="o">.</span><span class="n">seconds</span><span class="p">,</span>
<span class="p">)</span>
</code></pre></div></div>

<p>And that approach, along with maybe 15-20 derivations and tweaks of it, always resulted in
a Signature mismatch error message:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>127.0.0.1 <span class="o">[</span>REQUEST s3.PutObjectPart] 22:59:38.025
127.0.0.1 PUT /sregistry/test/big:sha256.92278b7c046c0acf0952b3e1663b8abb...
127.0.0.1 Host: 127.0.0.1:9000
127.0.0.1 Content-Length: 928
127.0.0.1 User-Agent: Go-http-client/1.1
127.0.0.1 X-Amz-Content-Sha256: 2fc597b42f249400d24a12904033454931eb3624e8c048fe825c360d9c1e61bf
127.0.0.1 Accept-Encoding: <span class="nb">gzip
</span>127.0.0.1 &lt;BODY&gt;
127.0.0.1 <span class="o">[</span>RESPONSE] <span class="o">[</span>22:59:38.025] <span class="o">[</span> Duration 670µs  ↑ 68 B  ↓ 806 B <span class="o">]</span>
127.0.0.1 403 Forbidden
127.0.0.1 X-Xss-Protection: 1<span class="p">;</span> <span class="nv">mode</span><span class="o">=</span>block
127.0.0.1 Accept-Ranges: bytes
127.0.0.1 Content-Length: 549
127.0.0.1 Content-Security-Policy: block-all-mixed-content
127.0.0.1 Content-Type: application/xml
127.0.0.1 Server: MinIO/RELEASE.2020-04-02T21-34-49Z
127.0.0.1 Vary: Origin
127.0.0.1 X-Amz-Request-Id: 1602C00C5749AF1F
127.0.0.1 &lt;?xml <span class="nv">version</span><span class="o">=</span><span class="s2">"1.0"</span> <span class="nv">encoding</span><span class="o">=</span><span class="s2">"UTF-8"</span>?&gt;
&lt;Error&gt;&lt;Code&gt;SignatureDoesNotMatch&lt;/Code&gt;&lt;Message&gt;The request signature we...
127.0.0.1 
</code></pre></div></div>

<p>Oh no!! You can imagine I dove into figuring out if the signature algorithm was correct,
if the client was okay, and what headers to use. Early on I figured out that Minio
uses a s3v4 signature, for which the host is included, so this was huge reason that
the external client needed to generate the signed urls and not the internal ones.
You can see my post from the end of the <a href="https://github.com/singularityhub/sregistry/pull/298#issuecomment-609490746" target="_blank">Sunday</a> that links to only some of the resources that I was using
to try and debug.</p>

<h2 id="3-reproducing-minio-py">3. Reproducing minio-py</h2>

<p>I realized that if the single PUT request detailed above works to generate a signed URL
from inside my Docker container for the outside calling client, perhaps if I replicated
this approach to generate the signature, my multipart pre-signed URLs would work too?
I figured out that I could import “presign_v4” from minio.signers and then use
it to generate the signature:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">signed_url</span> <span class="o">=</span> <span class="n">presign_v4</span><span class="p">(</span>
    <span class="s">"PUT"</span><span class="p">,</span>
    <span class="n">url</span><span class="p">,</span>
    <span class="n">region</span><span class="o">=</span><span class="n">MINIO_REGION</span><span class="p">,</span>
    <span class="n">credentials</span><span class="o">=</span><span class="n">minioExternalClient</span><span class="o">.</span><span class="n">_credentials</span><span class="p">,</span>
    <span class="n">expires</span><span class="o">=</span><span class="nb">str</span><span class="p">(</span><span class="n">timedelta</span><span class="p">(</span><span class="n">minutes</span><span class="o">=</span><span class="n">MINIO_SIGNED_URL_EXPIRE_MINUTES</span><span class="p">)</span><span class="o">.</span><span class="n">seconds</span><span class="p">),</span>
    <span class="n">headers</span><span class="o">=</span><span class="p">{</span><span class="s">"X-Amz-Content-Sha256"</span><span class="p">:</span> <span class="n">sha256</span><span class="p">},</span>
    <span class="n">response_headers</span><span class="o">=</span><span class="n">params</span><span class="p">,</span>
<span class="p">)</span>
</code></pre></div></div>

<p>In the above, the url consisted of the minio (external) base, followed by
the path in storage, something like:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>http://127.0.0.1:9000/sregistry/test/chonker%3Asha256.92278b7c046c0acf0952b3...
</code></pre></div></div>

<p>The credential I provided from the minioExternalClient to be absolutey sure it
was the same; specifying the region is hugely important because in previous
attempts leaving it out led to another error, and then of course the expires
should be in seconds (and it appears that the function defaults to using a string).
For response headers, this is where I included “uploadID” and “partNumber”
that are needed. You’ll also notice that in the example above, I tried
adding the “X-Amz-Content-Sha256” header that was provided by the client (it
still didn’t work). Actually, all of my derivations of this call (adding or removing
headers, tweaking the signature) didn’t work. I didn’t save a record of absolutely
everything that I tried, but I can guarantee you that it was most of Sunday
and a good chunk of Monday. I spent a lot of time reading every post or GitHub
issue on either minio, S3, or Multipart uploads, and posted on several
slacks which turned out to be wonderful rubber duck equivalents!</p>

<p>I decided to look more closely at the parse_v4 function. I noticed something interesting -
that by default, it created an internal variable called <a href="https://github.com/minio/minio-py/blob/f4b20f90d408215dcd51eb102f069b8355c13dc4/minio/signer.py#L96" target="_blank">
content_hash_hex</a> that by default was using a sha256sum for an empty payload.
This seemed off to me, and especially because Singularity was going out of it’s way to
provide the sha256sum for each part. What if I added it there? I wrote almost an
equivalent function, but this time exposed that sha256sum as an input variable:</p>

<pre><code class="language-pythonf">signed_url = sregistry_presign_v4(
    "PUT",
    new_url,
    region=MINIO_REGION,
    content_hash_hex=sha256,
    credentials=minioExternalClient._credentials,
    expires=str(timedelta(minutes=MINIO_SIGNED_URL_EXPIRE_MINUTES).seconds),
    headers={"X-Amz-Content-Sha256": sha256},
    response_headers=params,
)
</code></pre>

<p>And then returned this signed URL</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1"># Return the presigned url
</span><span class="n">data</span> <span class="o">=</span> <span class="p">{</span><span class="s">"presignedURL"</span><span class="p">:</span> <span class="n">signed_url</span><span class="p">}</span>
<span class="k">return</span> <span class="n">Response</span><span class="p">(</span><span class="n">data</span><span class="o">=</span><span class="p">{</span><span class="s">"data"</span><span class="p">:</span> <span class="n">data</span><span class="p">},</span> <span class="n">status</span><span class="o">=</span><span class="mi">200</span><span class="p">)</span>
</code></pre></div></div>

<p>And in a beautiful stream of data, all of a sudden all of the Multipart requests
were going through! This image shows the uwsgi logs for my server (left) and the
minio logs generated by the mc command line client (right).</p>

<div style="padding: 20px;">
   <img src="https://vsoch.github.io/assets/images/posts/minio/stream.png" />
</div>

<p>This was a great moment, because when you’ve tried getting something to work for days,
and have had infinite patience and attention to detali, when it works you almost
fly out of your shoes and take a trip around your apartment building.</p>

<h2 id="4-completing-the-request">4. Completing the Request</h2>

<p>The last step was
just completing the multipart upload, which is done with another 
<a href="https://github.com/sylabs/scs-library-client/blob/5d0614e4bddff1b2231efe79609f9f177bec8014/client/push.go#L537" target="_blank">scs-library-client</a> view that provides an uploadID and list of parts, each part
providing a partNumber and token, but the token is actually referring to what
is called an “ETag.” The server can receive this request, parse these
from the body, and then use the internal s3 client to finish the request.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">def</span> <span class="nf">put</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">request</span><span class="p">,</span> <span class="n">upload_id</span><span class="p">):</span>
    <span class="s">"""A put is done to complete the upload, providing the image id and number parts
       https://github.com/sylabs/scs-library-client/blob/master/client/push.go#L537
    """</span>
    <span class="k">print</span><span class="p">(</span><span class="s">"PUT RequestMultiPartCompleteView"</span><span class="p">)</span>

    <span class="c1"># 1. Again handle authorization of the request
</span>    <span class="c1"># 2. Parse uploadID and completedParts list from the body
</span>
    <span class="n">body</span> <span class="o">=</span> <span class="n">json</span><span class="o">.</span><span class="n">loads</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">body</span><span class="o">.</span><span class="n">decode</span><span class="p">(</span><span class="s">"utf-8"</span><span class="p">))</span>

    <span class="c1"># Assemble list of parts as they are expected for Python client
</span>    <span class="n">parts</span> <span class="o">=</span> <span class="p">[</span>
        <span class="p">{</span><span class="s">"ETag"</span><span class="p">:</span> <span class="n">x</span><span class="p">[</span><span class="s">"token"</span><span class="p">]</span><span class="o">.</span><span class="n">strip</span><span class="p">(</span><span class="s">'"'</span><span class="p">),</span> <span class="s">"PartNumber"</span><span class="p">:</span> <span class="n">x</span><span class="p">[</span><span class="s">"partNumber"</span><span class="p">]}</span>
        <span class="k">for</span> <span class="n">x</span> <span class="ow">in</span> <span class="n">body</span><span class="p">[</span><span class="s">"completedParts"</span><span class="p">]</span>
    <span class="p">]</span>

    <span class="c1"># 3. Get the container, return 404 if not found
</span>
    <span class="c1"># 4. Complete the multipart upload
</span>    <span class="n">res</span> <span class="o">=</span> <span class="n">s3</span><span class="o">.</span><span class="n">complete_multipart_upload</span><span class="p">(</span>
        <span class="n">Bucket</span><span class="o">=</span><span class="n">MINIO_BUCKET</span><span class="p">,</span>
        <span class="n">Key</span><span class="o">=</span><span class="n">container</span><span class="o">.</span><span class="n">get_storage</span><span class="p">(),</span>
        <span class="n">MultipartUpload</span><span class="o">=</span><span class="p">{</span><span class="s">"Parts"</span><span class="p">:</span> <span class="n">parts</span><span class="p">},</span>
        <span class="n">UploadId</span><span class="o">=</span><span class="n">body</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">"uploadID"</span><span class="p">),</span>
        <span class="c1"># RequestPayer='requester'
</span>    <span class="p">)</span>

    <span class="c1"># Currently this response data is empty
</span>    <span class="c1"># https://github.com/sylabs/scs-library-client/blob/master/client/response.go#L97
</span>     <span class="k">return</span> <span class="n">Response</span><span class="p">(</span><span class="n">status</span><span class="o">=</span><span class="mi">200</span><span class="p">,</span> <span class="n">data</span><span class="o">=</span><span class="p">{})</span>
</code></pre></div></div>

<p>The above has a lot of pseudo code, but generally you can see that we use the s3
client we created (the internal one) to issue a “complete_multipart_upload” with
the provided list of parts. We then have the containers in our collection:</p>

<div style="padding: 20px;">
   <img src="https://vsoch.github.io/assets/images/posts/minio/collection.png" />
</div>

<p>and these can be viewed in the Minio browser, which was shown at the top of this post.</p>

<h2 id="next-steps">Next Steps</h2>

<p>I’m not sure if the pull request will be accepted, but I did <a href="https://github.com/minio/minio-py/pull/870" target="_blank">open one</a>
to expose the variable to the presign_v4 function so if others run into my issue, they don’t need to rewrite the function.
And the integration is not complete! If you want to test the current pull request, you can find everything that you need
<a href="https://github.com/singularityhub/sregistry/pull/298" target="_blank">here</a>.
Note that we will need to test both setting up SSL (meaning generating certificates for Minio,
something I’ll need help with since I don’t have a need to deploy a Registry myself)
and also customizing the various environment variables for Minio and the container.
I’ve done the bulk of hard work here I think, and hopefully interested users can help
me to test and finish up the final documentation that is needed to properly deploy a registry.
You can reference <a href="https://github.com/singularityhub/sregistry/blob/d7420186b03711723e146bf3de1a06040facd722/docs/_docs/install/server.md#storage" target="_blank">this README</a> for documentation that will
be rendered into the <a href="https://singularityhub.github.io/sregistry/docs/install/server" target="_blank">web server and storage</a>
pages.</p>

<p>And that’s it! What a fun few days. I hope that users of Singularity Registry Server
can help to test out the endpoints, and adding support for SSL. Happy Monday everyone!</p>