---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2020-03-12 14:47:00'
layout: post
original_url: https://vsoch.github.io//2020/crc32c-validation-google-storage/
title: crc32c Validation for Google Storage Objects
---

<p>I recently was working on <a href="https://github.com/snakemake/snakemake" target="_blank">snakemake</a> and we ran into
the issue where a rule reported an unexpected end of file. Since the files were
obtained with the <a href="https://snakemake.readthedocs.io/en/stable/snakefiles/remote_files.html#google-cloud-storage-gs" target="_blank">Google Storage (GS) remote</a>, this meant that the download had not produced the full
file. We needed to verify the completeness of the file using
a <a href="https://en.wikipedia.org/wiki/Checksum" target="_blank">checksum</a>.
Since this was fun (and likely useful for others) I want to share what I learned
and wound up <a href="https://github.com/snakemake/snakemake/pull/273" target="_blank">implementing</a> here.</p>

<h2 id="google-storage-checksums">Google Storage Checksums</h2>

<p>If you look at the <a href="https://cloud.google.com/storage/docs/hashes-etags#_CRC32C" target="_blank">Google Cloud Storage</a>
page, you’ll notice that storage objects are provided with both an md5 and a crc32c checksum. You can 
read a little bit about both at the link provided. Since we are using Python, and the
<a href="https://pypi.org/project/google-cloud-storage/" target="_blank">google storage client</a> 
 exposes this crc32c checksum as an <a href="https://googleapis.dev/python/storage/latest/blobs.html?highlight=checksum#google.cloud.storage.blob.Blob.crc32c" target="_blank">attribute</a>, I figured this
would be fairly easy to do.</p>

<h2 id="google-storage-streaming-to-file">Google Storage Streaming to File</h2>

<p>Okay, so I need to generate a hasher, and then update it with chunks from the file being
streamed to download to the host from Google Storage. We would usually use Python’s hashlib,
and that might look something like this (md5 is used in this example, and it’s pseudocode):</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kn">import</span> <span class="nn">hashlib</span>

<span class="n">hasher</span> <span class="o">=</span> <span class="n">hashlib</span><span class="o">.</span><span class="n">md5</span><span class="p">()</span>
<span class="k">for</span> <span class="n">chunk</span> <span class="ow">in</span> <span class="n">some_method_to_stream_download</span><span class="p">():</span>
    <span class="c1"># write chunk to file
</span>    <span class="n">hasher</span><span class="o">.</span><span class="n">update</span><span class="p">(</span><span class="n">chunk</span><span class="p">)</span>

<span class="c1"># Get the final digest for comparison with something provided
</span><span class="k">assert</span> <span class="n">hasher</span><span class="o">.</span><span class="n">hexdigest</span><span class="p">()</span> <span class="o">==</span> <span class="n">some_provided_checksum</span>
</code></pre></div></div>

<p>But the problem is that the Google Storage client does not provide a streaming
function. They provide functions to write to a filename, or to write to a file
object. And lots of people <a href="https://github.com/googleapis/python-storage/issues/29" target="_blank">seem
to want this feature</a>. Here are the functions that we do have:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kn">from</span> <span class="nn">google.cloud.storage</span> <span class="kn">import</span> <span class="n">Client</span>

<span class="n">client</span> <span class="o">=</span> <span class="n">Client</span><span class="p">()</span>
<span class="n">bucket</span> <span class="o">=</span> <span class="n">client</span><span class="o">.</span><span class="n">get_bucket</span><span class="p">(</span><span class="s">'bucket_name'</span><span class="p">)</span>
<span class="n">blob</span> <span class="o">=</span> <span class="n">bucket</span><span class="o">.</span><span class="n">blob</span><span class="p">(</span><span class="s">'the_awesome_blob.txt'</span><span class="p">)</span>

<span class="c1"># Download to a file object
</span><span class="n">blob</span><span class="o">.</span><span class="n">download_to_file</span><span class="p">(</span><span class="n">file_obj</span><span class="p">)</span>

<span class="c1"># Download to filename
</span><span class="n">blob</span><span class="o">.</span><span class="n">download_to_filename</span><span class="p">(</span><span class="s">"name_of_file.txt"</span><span class="p">)</span>
</code></pre></div></div>

<p>So how do we stream to file? Well, we could download it using one of the above
methods, and then read it in again (streaming) to update a digest. But doesn’t that mean
we need to iterate through the chunks twice? That might work okay for tiny files,
but some of these files are chonkers. What if we tried to wrap the file object and expose
the same needed functions (namely write)…</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">class</span> <span class="nc">Crc32cCalculator</span><span class="p">(</span><span class="nb">object</span><span class="p">):</span>
    <span class="s">"""The Google Python client doesn't provide a way to stream a file being
       written, so we can wrap the file object in an additional class to
       do custom handling. This is so we don't need to download the file
       and then stream read it again to calculate the hash.
   """</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">fileobj</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">_fileobj</span> <span class="o">=</span> <span class="n">fileobj</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">digest</span> <span class="o">=</span> <span class="n">hasher</span><span class="o">.</span><span class="n">md5</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">write</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">chunk</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">_fileobj</span><span class="o">.</span><span class="n">write</span><span class="p">(</span><span class="n">chunk</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">_update</span><span class="p">(</span><span class="n">chunk</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">_update</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">chunk</span><span class="p">):</span>
        <span class="s">"""Given a chunk from the read in file, update the hexdigest
        """</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">digest</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">digest</span><span class="o">.</span><span class="n">update</span><span class="p">(</span><span class="n">chunk</span><span class="p">)</span>
</code></pre></div></div>

<p>But then added our own little special parsing of the chunk after the standard
write? This is what is shown in the example above - calling write()
writes the chunk to the file object, but then uses the _update()
function we’ve written with the class to update the digest created on init
with the chunk. But the example above uses md5. We don’t want to use md5. 
We want crc32c! Let’s figure that out next.</p>

<h2 id="choosing-a-crc32c-library">Choosing a crc32c Library</h2>

<p>The <a href="https://github.com/ICRAR/crc32c" target="_blank">crc32c module</a>
was my choice to calculate the hash, and although there wasn’t documentation I was able to see
how it’s used for binary data in <a href="https://github.com/ICRAR/crc32c/blob/master/test/test_crc32c.py" target="_blank">this test</a>.
The maintainer then was able to give <a href="https://github.com/ICRAR/crc32c/issues/14" target="_blank">speedy and lovely feedback</a> about the library and usage. For example, when we use it we need to import <code class="language-plaintext highlighter-rouge">crc32</code> from <code class="language-plaintext highlighter-rouge">crc32c</code> which is confusing,
because <code class="language-plaintext highlighter-rouge">crc32</code> is technically a <a href="https://stackoverflow.com/questions/26429360/crc32-vs-crc32c" target="_blank">different algorithm</a>. But he clarified the reason for this:</p>

<blockquote>
  <p>The package in PyPI is called crc32c, and the module is called crc32c, but it exposes a function called crc32. To be honest this is probably historical baggage – when this package was first implemented it tried to mimic binascii.crc32 to an extreme, including the function name… Maybe in a later release we could adjust the name</p>
</blockquote>

<p>I love learning about history behind some of these choices! Anyway, it turns out
that you can create an equvalent digest, and then update it (providing the previous digest too)
when you are reading some chunk.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">digest</span> <span class="o">=</span> <span class="mi">0</span>

<span class="c1"># reading in chunks, keep doing this
</span><span class="n">digest</span> <span class="o">=</span> <span class="n">crc32</span><span class="p">(</span><span class="n">chunk</span><span class="p">,</span> <span class="n">digest</span><span class="p">)</span>
</code></pre></div></div>

<p>At the end of that process (when the file is finished reading, either from a read
or write) you’ll have a digest. But there is still one more step! If we look
at an actual digest from Google Storage vs. what we calculated, the formats are different:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">blob</span><span class="o">.</span><span class="n">crc32c</span>
<span class="c1"># 'nuFtcw=='
</span><span class="n">digest</span>
<span class="c1"># 2665573747
</span></code></pre></div></div>

<p>Ah, crap! Well, it turns out, the last key to figuring this out was shared in
the <a href="https://cloud.google.com/storage/docs/hashes-etags" target="_blank">original documentation</a>:</p>

<blockquote>
  <p>The Base64 encoded CRC32c is in big-endian byte order</p>
</blockquote>

<p>Woot! So we needed one more final step to use Python’s <a href="https://docs.python.org/3/library/struct.html" target="_blank">struct.pack</a>.
I first used this when I was developing <a href="https://github.com/singularityhub/sif/blob/d4d915a219b9f69d2f95260bd559a061dd837ea9/sif/main/header.py" target="_blank">Singularity SIF in Python</a>, although I was using unpack. The final step
was to do the conversion, and I also converted from bytes, since the Google Storage API was returning
a string. The thing I love about this line is that you can almost read it from left to
right to understand what is going on, based on the description above.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1"># "Base 64 encode this thing in big endian byte order"
</span><span class="n">base64</span><span class="o">.</span><span class="n">b64encode</span><span class="p">(</span><span class="n">struct</span><span class="o">.</span><span class="n">pack</span><span class="p">(</span><span class="s">"&gt;I"</span><span class="p">,</span> <span class="bp">self</span><span class="o">.</span><span class="n">digest</span><span class="p">))</span>

<span class="c1"># Oh, but also decode it so we return a string
</span><span class="n">base64</span><span class="o">.</span><span class="n">b64encode</span><span class="p">(</span><span class="n">struct</span><span class="o">.</span><span class="n">pack</span><span class="p">(</span><span class="s">"&gt;I"</span><span class="p">,</span> <span class="bp">self</span><span class="o">.</span><span class="n">digest</span><span class="p">))</span><span class="o">.</span><span class="n">decode</span><span class="p">(</span><span class="s">"utf-8"</span><span class="p">)</span>
</code></pre></div></div>

<h2 id="all-together-now">All Together Now!</h2>

<p>Okay, so we’ve figured out the library to use, and how to add it to the file object
class to provide to Google Storage, let’s put all the pieces together! Here
are the first set of imports that we need:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kn">import</span> <span class="nn">os</span>
<span class="kn">import</span> <span class="nn">base64</span>
<span class="kn">import</span> <span class="nn">struct</span>
<span class="kn">from</span> <span class="nn">crc32c</span> <span class="kn">import</span> <span class="n">crc32</span>
</code></pre></div></div>

<p>And here is our special calculator class to wrap the file object, and expose
a “write” function that then does an update of our digest with the
same chunk. Notice that when we initialize, we also set the digest to 0,
as we saw in all the examples for using crc32c.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">class</span> <span class="nc">Crc32cCalculator</span><span class="p">(</span><span class="nb">object</span><span class="p">):</span>
    <span class="s">"""The Google Python client doesn't provide a way to stream a file being
       written, so we can wrap the file object in an additional class to
       do custom handling. This is so we don't need to download the file
       and then stream read it again to calculate the hash.
   """</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">fileobj</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">_fileobj</span> <span class="o">=</span> <span class="n">fileobj</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">digest</span> <span class="o">=</span> <span class="mi">0</span>

    <span class="k">def</span> <span class="nf">write</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">chunk</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">_fileobj</span><span class="o">.</span><span class="n">write</span><span class="p">(</span><span class="n">chunk</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">_update</span><span class="p">(</span><span class="n">chunk</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">_update</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">chunk</span><span class="p">):</span>
        <span class="s">"""Given a chunk from the read in file, update the hexdigest
        """</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">digest</span> <span class="o">=</span> <span class="n">crc32</span><span class="p">(</span><span class="n">chunk</span><span class="p">,</span> <span class="bp">self</span><span class="o">.</span><span class="n">digest</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">hexdigest</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">"""Return the hexdigest of the hasher.
           The Base64 encoded CRC32c is in big-endian byte order.
           See https://cloud.google.com/storage/docs/hashes-etags
        """</span>
        <span class="k">return</span> <span class="n">base64</span><span class="o">.</span><span class="n">b64encode</span><span class="p">(</span><span class="n">struct</span><span class="o">.</span><span class="n">pack</span><span class="p">(</span><span class="s">"&gt;I"</span><span class="p">,</span> <span class="bp">self</span><span class="o">.</span><span class="n">digest</span><span class="p">))</span><span class="o">.</span><span class="n">decode</span><span class="p">(</span><span class="s">"utf-8"</span><span class="p">)</span>
</code></pre></div></div>

<p>Finally, notice that we expose a hexdigest() function so the class has similar functionality
to one provided by hashlib. Now let’s use this class with the blob.download_to_file
in a download function to get a final solution. You can
look at the <a href="https://github.com/snakemake/snakemake/pull/273" target="_blank">Snakemake PR</a>
to get a more real world context. The function below uses the class above to download the file
and ensure that the crc32c checksums match.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">def</span> <span class="nf">download</span><span class="p">(</span><span class="n">blob</span><span class="p">,</span> <span class="n">file_name</span><span class="p">):</span>
    <span class="s">"""download a blob object to some file_name.
       We assume the download directory and blob both exist for 
       simplicity of this snippet
    """</span>
    <span class="c1"># Continue trying to download until hash matches
</span>    <span class="k">while</span> <span class="ow">not</span> <span class="n">os</span><span class="o">.</span><span class="n">path</span><span class="o">.</span><span class="n">exists</span><span class="p">(</span><span class="n">file_name</span><span class="p">):</span>

        <span class="k">with</span> <span class="nb">open</span><span class="p">(</span><span class="n">file_name</span><span class="p">,</span> <span class="s">"wb"</span><span class="p">)</span> <span class="k">as</span> <span class="n">blob_file</span><span class="p">:</span>
            <span class="n">parser</span> <span class="o">=</span> <span class="n">Crc32cCalculator</span><span class="p">(</span><span class="n">blob_file</span><span class="p">)</span>
            <span class="n">blob</span><span class="o">.</span><span class="n">download_to_file</span><span class="p">(</span><span class="n">parser</span><span class="p">)</span>
           
        <span class="c1"># Compute local hash and verify correct
</span>        <span class="k">if</span> <span class="n">parser</span><span class="o">.</span><span class="n">hexdigest</span><span class="p">()</span> <span class="o">!=</span> <span class="n">blob</span><span class="o">.</span><span class="n">crc32c</span><span class="p">:</span>
            <span class="n">os</span><span class="o">.</span><span class="n">remove</span><span class="p">(</span><span class="n">file_name</span><span class="p">)</span>

    <span class="k">return</span> <span class="n">file_name</span>
</code></pre></div></div>

<p>Used as follows:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kn">from</span> <span class="nn">google.cloud</span> <span class="kn">import</span> <span class="n">storage</span>
<span class="n">client</span> <span class="o">=</span> <span class="n">storage</span><span class="o">.</span><span class="n">Client</span><span class="p">()</span>
<span class="n">bucket</span> <span class="o">=</span> <span class="n">client</span><span class="o">.</span><span class="n">get_bucket</span><span class="p">(</span><span class="s">'bucket_name'</span><span class="p">)</span>
<span class="n">blob</span> <span class="o">=</span> <span class="n">bucket</span><span class="o">.</span><span class="n">blob</span><span class="p">(</span><span class="s">'the_awesome_blob.txt'</span><span class="p">)</span>
<span class="n">file_name</span> <span class="o">=</span> <span class="s">"the_downloaded_awesome.txt"</span>
<span class="n">downloaded_file</span> <span class="o">=</span> <span class="n">download</span><span class="p">(</span><span class="n">blob</span><span class="p">,</span> <span class="n">file_name</span><span class="p">)</span>
</code></pre></div></div>

<p>Obviously the example above is overly simple - you would want to check that
the directory exists for the file to be downloaded, and that the blob.exists()
as well. Finally, you’d probably want to have some maximum number of retries,
clear messages to the user, and other error parsing.</p>

<h2 id="what-would-i-like-to-see">What would I like to see?</h2>

<p>It would be much simpler if the Google Storage Python team could provide
a function to stream to a file directly. For example:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">digest</span> <span class="o">=</span> <span class="mi">0</span>
<span class="k">with</span> <span class="nb">open</span><span class="p">(</span><span class="n">file_name</span><span class="p">,</span> <span class="s">"wb"</span><span class="p">)</span> <span class="k">as</span> <span class="n">fd</span><span class="p">:</span>
    <span class="k">for</span> <span class="n">chunk</span> <span class="ow">in</span> <span class="n">blob</span><span class="o">.</span><span class="n">stream_chunks</span><span class="p">(</span><span class="n">chunk_size</span><span class="o">=</span><span class="mi">1024</span> <span class="o">*</span> <span class="mi">1024</span> <span class="o">*</span> <span class="mi">10</span><span class="p">):</span>
        <span class="n">fd</span><span class="o">.</span><span class="n">write</span><span class="p">(</span><span class="n">chunk</span><span class="p">)</span>
        <span class="n">digest</span> <span class="o">=</span> <span class="n">crc32</span><span class="p">(</span><span class="n">chunk</span><span class="p">,</span> <span class="n">digest</span><span class="p">)</span>

<span class="n">checksum</span> <span class="o">=</span> <span class="n">base64</span><span class="o">.</span><span class="n">b64encode</span><span class="p">(</span><span class="n">struct</span><span class="o">.</span><span class="n">pack</span><span class="p">(</span><span class="s">"&gt;I"</span><span class="p">,</span> <span class="bp">self</span><span class="o">.</span><span class="n">digest</span><span class="p">))</span><span class="o">.</span><span class="n">decode</span><span class="p">(</span><span class="s">"utf-8"</span><span class="p">)</span>
<span class="k">assert</span> <span class="n">checksum</span> <span class="o">==</span> <span class="n">blob</span><span class="o">.</span><span class="n">crc32c</span>
</code></pre></div></div>

<p>That would be so much easier! But that function doesn’t exist, so until
then, this seems like a reasonable solution.</p>

<h2 id="thank-you">Thank you!</h2>

<p>Thank you again to <a href="https://github.com/rtobar" target="_blank">rtobar</a>
for your speedy help - I wasn’t expecting to get invested so quickly in working
on this later in the evening, and it was so much fun. This is probably my 
favorite kind of problem to work on - when there isn’t a plug and play solution,
and you are able to do a little digging (and get help from friends!) to ultimately
find a solution, and learn a little bit too.</p>