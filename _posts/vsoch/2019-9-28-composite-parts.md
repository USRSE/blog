---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-09-28 12:05:00'
layout: post
original_url: https://vsoch.github.io//2019/composite-parts/
title: Discovering Parts of Composite Parts
---

<p>Welcome to the universe of parts! Here we have “Parts,” or sequences of letters,
and “Composite Parts,” or combined Parts, each with an ordering and direction.
If you can’t tell, these are sequences. I’m not a biologist, so I apologize in
advance for all my misspeaks and ignorance to all the biologists reading this post. Now 
here are the two parts - let’s call them our base parts:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="n">base1</span><span class="o">=</span><span class="s">'AAACACGTGGCAAACATTCCGGTCTCTATGAAGAGCTTCGAGACCAAAGGGGCCGTCAATATCAGCAGAGACATCGGTGCGCATACAGGCTCCCTGT</span><span class="err">
</span><span class="s">AGGCTCCGCGAGATAACGTAGACAGGTTGTTTTGACTAGTGGTTCAAAGGCCTACTCCCCCAAATCTGCCTATCACAAGTTCAGTCGAAAAGCATAGTAAGCGCCCCAA</span><span class="err">
</span><span class="s">TTTGCCAGTCGCAACAAGTCTGAGACCTACTAGTAGGGGAGATGATGCTTACCGCGGTCCTGAAGCGCAAGACGGGGTGAATGGCCCCGTTCTGCTATGGTTCTCAAGG</span><span class="err">
</span><span class="s">TAGTATGTGCATTGCTCTTAGTGAACTAGGGATAACAGGGTAATATAGAGGGATTGGTACCAAGT'</span>

<span class="n">base2</span><span class="o">=</span><span class="s">'GGTCTCAAATGAATAAAAAGGAGATCTTTAACACTGACTTCTTTGAATCTGGACTTGCCTACATCTTAACCAACCTTGATTTCATCCAAGAAGAGTTAGAGCAA</span><span class="err">
</span><span class="s">GAGAAGCTGCAAACCTCTCTTGTTGAAAAACTTATTACTGATTTTGAGGACGTCGAGGATTATGAGACATGGGACCTTTTGACTAACAATTTGATTCAAAGTGAAGATA</span><span class="err">
</span><span class="s">AGATCCTTGAGGAGATTCAGAAGATCAAGGACTCTACAAAATTTAATTTGCTGAATAGCTACTTCCTTGCCAAAAACCTGGCAATCTATTTAAAGTCCAATTCTTTTCT</span><span class="err">
</span><span class="s">TATCGAACAAATCAATAAGTTACAGACAAATAGTCCTGACGACCTTTCTGAAGATAAAAAGGAGGAGTTCATTAACAACCTGAAACAAGAGATTTTGAAAAACAATTCT</span><span class="err">
</span><span class="s">GAATTATATAAGCAAAACGAACGCTTGTTTAAGGAAATCTTTGACAAAAAGGTTGAATTTAAGAAGATTTATCAACTGCTGATTAAGGAAACCGAGTTTGAGGACTTCA</span><span class="err">
</span><span class="s">ACTATGCGAACGAGTTATTGTTTAATATGTTAAATAATAATTTTAAGTTCAACAATAAACAGGACCTTTTAAAACTTGAGGTCCTGAACAACGCCCAGAGCCTGATTGA</span><span class="err">
</span><span class="s">TTTTTTGACGTTTTACGAGAGCTCTTTGTTTGATGATGAAAAGGAGTGAAGAGCTTAGAGACC'</span>
</code></pre></div></div>

<p>Let’s now combine them into a new composite part,
and create a fun name. Because we want to be tricky, we are going to reverse the second:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="o">&gt;</span> <span class="n">name</span> <span class="o">=</span> <span class="s">"Dinosaur Part"</span>
<span class="o">&gt;</span> <span class="n">sequence</span> <span class="o">=</span> <span class="n">base1</span> <span class="o">+</span> <span class="n">base2</span><span class="p">[::</span><span class="o">-</span><span class="mi">1</span><span class="p">]</span>
</code></pre></div></div>

<p>The direction of the above is represented as “&gt;&lt;” meaning that the first part is in the
forward direction, and the second is reversed. We now have a sequence that is a combination of parts. We also have (not shown) a complete query result (a JSON object) from an API that gives us ALL the parts (~3000) in a database. 
They are indexed by a unique id, and each one includes a rich set of metadata. Here is an example of one entry, and  I’ve truncated the sequences and removed empty fields for brevity:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="p">{</span><span class="s">'uuid'</span><span class="p">:</span> <span class="s">'81a92bdc-2b71-48de-bdfc-fafcf9bf26ed'</span><span class="p">,</span>
 <span class="s">'name'</span><span class="p">:</span> <span class="s">'MG_RS01350'</span><span class="p">,</span>
 <span class="s">'status'</span><span class="p">:</span> <span class="s">'Unapproved'</span><span class="p">,</span>
 <span class="s">'gene_id'</span><span class="p">:</span> <span class="s">'BBF10K_002166'</span><span class="p">,</span>
 <span class="s">'part_type'</span><span class="p">:</span> <span class="s">'cds'</span><span class="p">,</span>
 <span class="s">'genbank'</span><span class="p">:</span> <span class="p">{</span><span class="s">'gene'</span><span class="p">:</span> <span class="s">''</span><span class="p">,</span>
  <span class="s">'note'</span><span class="p">:</span> <span class="s">'Derived by automated computational analysis usinggene prediction method: Protein Homology.'</span><span class="p">,</span>
  <span class="s">'Source'</span><span class="p">:</span> <span class="s">'Mycoplasma genitalium G37'</span><span class="p">,</span>
  <span class="s">'product'</span><span class="p">:</span> <span class="s">'ribonucleoside-diphosphate reductase subunitbeta'</span><span class="p">,</span>
  <span class="s">'locus_tag'</span><span class="p">:</span> <span class="s">'MG_RS01350'</span><span class="p">,</span>
  <span class="s">'protein_id'</span><span class="p">:</span> <span class="s">'WP_009885766.1'</span><span class="p">,</span>
  <span class="s">'GenBank_acc'</span><span class="p">:</span> <span class="s">'NC_000908 NZ_U39679-NZ_U39729'</span><span class="p">,</span>
  <span class="s">'translation'</span><span class="p">:</span> <span class="s">'MAANNKKYFLES'</span><span class="p">,</span>
  <span class="s">'gene_synonyms'</span><span class="p">:</span> <span class="s">''</span><span class="p">},</span>
 <span class="s">'original_sequence'</span><span class="p">:</span> <span class="s">'ATGGCAGCTAACAATAAAAA...'</span><span class="p">,</span>
 <span class="s">'optimized_sequence'</span><span class="p">:</span> <span class="s">'ATGGCTGCAAATAATAAAAAGTAT...'</span><span class="p">,</span>
 <span class="s">'synthesized_sequence'</span><span class="p">:</span> <span class="s">'AAGGAACTATGGCATCGAGCGGTCTCA...'</span><span class="p">,</span>
 <span class="s">'primer_forward'</span><span class="p">:</span> <span class="s">'AAGGAACTATGGCATCGAGC'</span><span class="p">,</span>
 <span class="s">'primer_reverse'</span><span class="p">:</span> <span class="s">'GTACGTCTGAACTTGGGACT'</span><span class="p">,</span>
 <span class="s">'label'</span><span class="p">:</span> <span class="s">'part'</span><span class="p">,</span>
 <span class="s">'translation'</span><span class="p">:</span> <span class="s">'MAANNKKYFLESFSPLGYVKNNFQGNL*'</span><span class="p">,</span>
 <span class="s">'tags'</span><span class="p">:</span> <span class="p">[{</span><span class="s">'name'</span><span class="p">:</span> <span class="s">'mycoplasma'</span><span class="p">,</span>
   <span class="s">'uuid'</span><span class="p">:</span> <span class="s">'53edbc05-ca3f-4246-9b7f-35c766e7af3e'</span><span class="p">},</span>
  <span class="p">{</span><span class="s">'name'</span><span class="p">:</span> <span class="s">'cds'</span><span class="p">,</span> <span class="s">'uuid'</span><span class="p">:</span> <span class="s">'3bc7e754-f6b5-424c-9910-8adcb5d59d9e'</span><span class="p">},</span>
  <span class="p">{</span><span class="s">'name'</span><span class="p">:</span> <span class="s">'moclo'</span><span class="p">,</span> <span class="s">'uuid'</span><span class="p">:</span> <span class="s">'63d87aa1-782a-4f97-94c9-44587d1236d3'</span><span class="p">},</span>
  <span class="p">{</span><span class="s">'name'</span><span class="p">:</span> <span class="s">'mycoplasma genitalium'</span><span class="p">,</span>
   <span class="s">'uuid'</span><span class="p">:</span> <span class="s">'8cdc02fc-5871-4629-afc7-4903703a9edb'</span><span class="p">}],</span>
 <span class="s">'collections'</span><span class="p">:</span> <span class="p">[],</span>
 <span class="s">'author'</span><span class="p">:</span> <span class="p">{</span><span class="s">'name'</span><span class="p">:</span> <span class="s">'Mickey Mouse'</span><span class="p">,</span>
  <span class="s">'uuid'</span><span class="p">:</span> <span class="s">'47a4b50c-3176-4579-8895-3f902fee26bf'</span><span class="p">}}</span>
</code></pre></div></div>

<h2 id="the-challenge">The Challenge</h2>

<p>Our challenge is to, given a query sequence such as the one we generated above, 
find the matching parts and their directions.
The real life scenario is using a create endpoint of an API to add a composite part, but to do this we would
need the parts, a name, and the directions. Since this would be too computationally intensive to do
on a server, I decided to have the client (a Python module that wraps the API) do the bulk of
work, and then submit the final part identifier and directions for validation. Our
final algorithm (run on a client computer) should be something like this:</p>

<ol class="custom-counter">
   <li>Cache all parts from the API (one call)</li>
   <li>Find all forward and reverse substrings that match (starts, ends, and length)</li>
   <li>Model as scheduling problem</li>
</ol>

<h3 id="step-1-cache-the-parts">Step 1: Cache the Parts</h3>

<p>The user is likely going to want to make many Composite Parts in one fell swoop, and
since the existing parts in the server database is set at a few thousand, it was logical to
do a single query to get all these parts and cache it locally with the client.
I don’t need to show the logic behind this - we add a caching function to the client
that is called in our TBA function to generate a new composite part:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">&gt;</span> <span class="n">client</span><span class="o">.</span><span class="n">_cache_parts</span><span class="p">()</span>
</code></pre></div></div>

<h3 id="step-2-find-substrings">Step 2: Find Substrings</h3>

<p>This is fairly straight forward - each part can be represented in the forward direction
(e.g., ABC) or the reverse direction (CBA) in the sequence. Let’s loop through our parts,
and save all occurrences (meaning the unique (uuid) of the part, it’s direction,
start and stopping index) in a list.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="c"># Parts found to match</span>
<span class="n">coords</span> <span class="o">=</span> <span class="p">[]</span>

<span class="k">for</span> <span class="n">uuid</span><span class="p">,</span> <span class="n">part</span> <span class="ow">in</span> <span class="bp">self</span><span class="o">.</span><span class="n">cache</span><span class="p">[</span><span class="s">'parts'</span><span class="p">]</span><span class="o">.</span><span class="n">items</span><span class="p">():</span>

    <span class="c"># Only use parts with optimized sequences</span>
    <span class="k">if</span> <span class="n">part</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'optimized_sequence'</span><span class="p">):</span>
        <span class="n">forward</span> <span class="o">=</span> <span class="n">part</span><span class="p">[</span><span class="s">'optimized_sequence'</span><span class="p">]</span>
        <span class="n">reverse</span> <span class="o">=</span> <span class="n">forward</span><span class="p">[::</span><span class="o">-</span><span class="mi">1</span><span class="p">]</span>

        <span class="c"># Case 1: we found the forward sequence</span>
        <span class="k">if</span> <span class="n">forward</span> <span class="ow">in</span> <span class="n">sequence</span><span class="p">:</span>
            <span class="k">for</span> <span class="n">match</span> <span class="ow">in</span> <span class="n">re</span><span class="o">.</span><span class="n">finditer</span><span class="p">(</span><span class="n">forward</span><span class="p">,</span> <span class="n">sequence</span><span class="p">):</span> 
                <span class="n">coords</span><span class="o">.</span><span class="n">append</span><span class="p">((</span><span class="n">part</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'uuid'</span><span class="p">),</span> <span class="s">"&gt;"</span><span class="p">,</span> <span class="n">match</span><span class="o">.</span><span class="n">start</span><span class="p">(),</span> <span class="n">match</span><span class="o">.</span><span class="n">end</span><span class="p">()))</span>

        <span class="c"># Case 2: we found the reverse sequence</span>
        <span class="k">if</span> <span class="n">reverse</span> <span class="ow">in</span> <span class="n">sequence</span><span class="p">:</span>
            <span class="k">for</span> <span class="n">match</span> <span class="ow">in</span> <span class="n">re</span><span class="o">.</span><span class="n">finditer</span><span class="p">(</span><span class="n">reverse</span><span class="p">,</span> <span class="n">sequence</span><span class="p">):</span> 
                <span class="n">coords</span><span class="o">.</span><span class="n">append</span><span class="p">((</span><span class="n">part</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'uuid'</span><span class="p">),</span> <span class="s">"&lt;"</span><span class="p">,</span> <span class="n">match</span><span class="o">.</span><span class="n">start</span><span class="p">(),</span> <span class="n">match</span><span class="o">.</span><span class="n">end</span><span class="p">()))</span>

</code></pre></div></div>

<p>That is fairly simple! And we are greatly helped by the <a href="https://docs.python.org/2/library/re.html#re.finditer" target="_blank">re.finditer</a>
function, which will return all matches (as re match objects) in the string. If we used re.search
we’d only get the first (and need to chop it off to find the rest) and if we used re.findall we’d
get the strings themselves (not super useful).</p>

<h3 id="step-3-make-a-queue">Step 3: Make a Queue!</h3>

<p>For some reason, I’ve always loved algorithms that use queues. They tend to be more intuitive,
and sort of fun, because you are processing the queue like a robot. In our case,
we want to sort the queue based on the length of the match, where the longest match
will be at the end of the list, and the shortest at the start. For example, here is an entry in our
list of coords (matches):</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="c"># uuid,  direction, start, end</span>
<span class="p">(</span><span class="s">'81a92bdc-2b71-48de-bdfc-fafcf9bf26ed'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">1023</span><span class="p">)</span>
</code></pre></div></div>

<p>The above would say that “Part with unique id 81a92bdc-2b71-48de-bdfc-fafcf9bf26ed was found
in the sequence in the forward (&gt;) direction, starting at index 0, and ending at index 1023.
This means to calculate the length, we subtract the end from the start. There is a nice
Pythonic way to do that, and generate our sorted queue from the list of coords:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="c"># Make a queue sorted by how long they (end - start)</span>
<span class="n">queue</span> <span class="o">=</span> <span class="nb">sorted</span><span class="p">(</span><span class="n">coords</span><span class="p">,</span> <span class="n">key</span><span class="o">=</span><span class="k">lambda</span> <span class="n">tup</span><span class="p">:</span> <span class="n">tup</span><span class="p">[</span><span class="mi">3</span><span class="p">]</span><span class="o">-</span><span class="n">tup</span><span class="p">[</span><span class="mi">2</span><span class="p">])</span>
</code></pre></div></div>

<h3 id="step-4-overlaps-with-function">Step 4: Overlaps With Function</h3>

<p>I’d want to have a function that returns a boolean if some new contender element
overlaps with any current parts in some list of selected sequences. There are probably
cleaner ways to do this, but I spelled it out verbatim: I loop through the selected
sequences (by the way, this starts as an empty list, and we would add the first 
element in the queue to it) and check if the contender element start or end is within
the range of a selected sequence start through end (i.e., it overlaps).</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="k">def</span> <span class="nf">overlaps_with</span><span class="p">(</span><span class="n">selected_sequences</span><span class="p">,</span> <span class="n">element</span><span class="p">):</span>
    <span class="s">'''determine if an element overlaps with any current elements in the list
    '''</span>
    <span class="k">for</span> <span class="n">selected</span> <span class="ow">in</span> <span class="n">selected_sequences</span><span class="p">:</span>

       <span class="c"># If the element start is greater than selected start, less than end</span>
       <span class="k">if</span> <span class="p">(</span><span class="n">element</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">&gt;=</span> <span class="n">selected</span><span class="p">[</span><span class="mi">2</span><span class="p">])</span> <span class="ow">and</span> <span class="p">(</span><span class="n">element</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">&lt;</span> <span class="n">selected</span><span class="p">[</span><span class="mi">3</span><span class="p">]):</span>
           <span class="k">return</span> <span class="bp">True</span>

       <span class="c"># If the element end is greater than the selected start, less than end</span>
       <span class="k">if</span> <span class="p">(</span><span class="n">element</span><span class="p">[</span><span class="mi">3</span><span class="p">]</span> <span class="o">&gt;</span> <span class="n">selected</span><span class="p">[</span><span class="mi">2</span><span class="p">])</span> <span class="ow">and</span> <span class="p">(</span><span class="n">element</span><span class="p">[</span><span class="mi">3</span><span class="p">]</span> <span class="o">&lt;=</span> <span class="n">selected</span><span class="p">[</span><span class="mi">3</span><span class="p">]):</span>
           <span class="k">return</span> <span class="bp">True</span>

    <span class="k">return</span> <span class="bp">False</span>
</code></pre></div></div>

<p>We return a boolean to indicate overlapping or not.</p>

<h3 id="step-5-process-the-queue">Step 5: Process the Queue</h3>

<p>I think this is a basic scheduling algorithm - we start with a queue of start and ending
times (sorted by length), and while this isn’t the greedy “optimal” <a href="https://en.wikipedia.org/wiki/Interval_scheduling#Greedy_polynomial_solution" target="_blank">solution</a>, I chose this method
because I think what I want is to match the longer ones first. For example, it could be there’s a lot of short sequences that can assemble together to formulate some portion of the query sequence, but we probably want to match the longer ones first because those are more likely to be important. It could be that there be other matching subsequences that don’t include the longest, and we can talk more about this after. Let’s first create an empty list of selected_sequences, and then add popped elements from the queue (remember, longest first!) only if there is no overlap with already added members:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="n">selected_sequences</span> <span class="o">=</span> <span class="p">[]</span>

<span class="k">while</span> <span class="n">queue</span><span class="p">:</span>

    <span class="c"># Pop the longest element ( the last )</span>
    <span class="n">element</span> <span class="o">=</span> <span class="n">queue</span><span class="o">.</span><span class="n">pop</span><span class="p">()</span>

    <span class="c"># If there is no overlap add</span>
    <span class="k">if</span> <span class="ow">not</span> <span class="n">overlaps_with</span><span class="p">(</span><span class="n">selected_sequences</span><span class="p">,</span> <span class="n">element</span><span class="p">):</span>
        <span class="n">selected_sequences</span><span class="o">.</span><span class="n">append</span><span class="p">(</span><span class="n">element</span><span class="p">)</span>

</code></pre></div></div>

<p>At the end of this loop I have a nice list of (non-sorted) sequences that don’t overlap.</p>

<h3 id="step-6-sort-by-start">Step 6: Sort by Start</h3>

<p>Here is a peek at the result from running the algorithm with our dummy data - we have a subset of the initial matches that don’t overlap.</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="p">[(</span><span class="s">'e7d46d00-e32e-417b-8628-0f5287d55840'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1023</span><span class="p">,</span> <span class="mi">1866</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'81a92bdc-2b71-48de-bdfc-fafcf9bf26ed'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">1023</span><span class="p">)]</span>
</code></pre></div></div>

<p>As you can see, the ordering can be off! So we need to do one more sort, but this time based
on the starting position (the second index counting with 0) and not the length:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">&gt;</span> <span class="n">selected_sequences</span> <span class="o">=</span> <span class="nb">sorted</span><span class="p">(</span><span class="n">selected_sequences</span><span class="p">,</span> <span class="n">key</span><span class="o">=</span><span class="k">lambda</span> <span class="n">tup</span><span class="p">:</span> <span class="n">tup</span><span class="p">[</span><span class="mi">2</span><span class="p">])</span>
</code></pre></div></div>

<p>And then we’re done! We can make a POST request to the API with our final 
list of part unique ids (they will be looked up and validated to exist in the database)
along with a direction string (which must be the same length as the number of parts)</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="n">data</span> <span class="o">=</span> <span class="p">{</span><span class="s">"name"</span><span class="p">:</span> <span class="n">name</span><span class="p">,</span> 
        <span class="s">"parts"</span><span class="p">:</span> <span class="p">[</span><span class="s">'81a92bdc-2b71-48de-bdfc-fafcf9bf26ed'</span><span class="p">,</span> <span class="s">'e7d46d00-e32e-417b-8628-0f5287d55840'</span><span class="p">],</span> 
        <span class="s">"sequence"</span><span class="p">:</span> <span class="n">sequence</span><span class="p">,</span> 
        <span class="s">"direction_string"</span><span class="p">:</span> <span class="s">"&gt;&lt;"</span><span class="p">}</span> 
</code></pre></div></div>

<p>That’s the POST data, I don’t need to show you the actual request. The API returns
the created object, in case the user wants it locally or to verify anything. But actually,
the user isn’t exposed to all of that code, the client presents all of the above
as a single, clean function:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="o">&gt;</span> <span class="n">composite_part</span> <span class="o">=</span> <span class="n">client</span><span class="o">.</span><span class="n">create_composite_part</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="n">name</span><span class="p">,</span> <span class="n">sequence</span><span class="o">=</span><span class="n">sequence</span><span class="p">)</span>
</code></pre></div></div>

<h2 id="all-possible-matches">All Possible Matches?</h2>

<p>In practice, when I did this there were about ~20 matches in my original list, 2 of which were the original
parts I selected, and the rest which were small optimized sequences (3 base pairs!) that
tended to repeat a ton of times. What I tried doing (just for testing) was to remove
the first (the longest, the real first half) and then the second (the second longest, the real second half)
from the queue, and run it again. I did find some form of result with the small
optimized sequence (of length 3) repeated many times - here is removing the first:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="p">[(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">19</span><span class="p">,</span> <span class="mi">22</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">104</span><span class="p">,</span> <span class="mi">107</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">182</span><span class="p">,</span> <span class="mi">185</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">205</span><span class="p">,</span> <span class="mi">208</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">226</span><span class="p">,</span> <span class="mi">229</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">250</span><span class="p">,</span> <span class="mi">253</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">284</span><span class="p">,</span> <span class="mi">287</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">310</span><span class="p">,</span> <span class="mi">313</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">326</span><span class="p">,</span> <span class="mi">329</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">475</span><span class="p">,</span> <span class="mi">478</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">505</span><span class="p">,</span> <span class="mi">508</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">515</span><span class="p">,</span> <span class="mi">518</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">529</span><span class="p">,</span> <span class="mi">532</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">622</span><span class="p">,</span> <span class="mi">625</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">632</span><span class="p">,</span> <span class="mi">635</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">715</span><span class="p">,</span> <span class="mi">718</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">731</span><span class="p">,</span> <span class="mi">734</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">736</span><span class="p">,</span> <span class="mi">739</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">748</span><span class="p">,</span> <span class="mi">751</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">752</span><span class="p">,</span> <span class="mi">755</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">772</span><span class="p">,</span> <span class="mi">775</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">817</span><span class="p">,</span> <span class="mi">820</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">919</span><span class="p">,</span> <span class="mi">922</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">989</span><span class="p">,</span> <span class="mi">992</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">1004</span><span class="p">,</span> <span class="mi">1007</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">1020</span><span class="p">,</span> <span class="mi">1023</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'e7d46d00-e32e-417b-8628-0f5287d55840'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1023</span><span class="p">,</span> <span class="mi">1866</span><span class="p">)]</span>
</code></pre></div></div>

<p>And here is removing the second:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="p">[(</span><span class="s">'81a92bdc-2b71-48de-bdfc-fafcf9bf26ed'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">1023</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1023</span><span class="p">,</span> <span class="mi">1026</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1052</span><span class="p">,</span> <span class="mi">1055</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1088</span><span class="p">,</span> <span class="mi">1091</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1124</span><span class="p">,</span> <span class="mi">1127</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1127</span><span class="p">,</span> <span class="mi">1130</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1148</span><span class="p">,</span> <span class="mi">1151</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1193</span><span class="p">,</span> <span class="mi">1196</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1225</span><span class="p">,</span> <span class="mi">1228</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1322</span><span class="p">,</span> <span class="mi">1325</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1355</span><span class="p">,</span> <span class="mi">1358</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1394</span><span class="p">,</span> <span class="mi">1397</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1418</span><span class="p">,</span> <span class="mi">1421</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1477</span><span class="p">,</span> <span class="mi">1480</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1496</span><span class="p">,</span> <span class="mi">1499</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1501</span><span class="p">,</span> <span class="mi">1504</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1571</span><span class="p">,</span> <span class="mi">1574</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1595</span><span class="p">,</span> <span class="mi">1598</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1637</span><span class="p">,</span> <span class="mi">1640</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1690</span><span class="p">,</span> <span class="mi">1693</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&gt;'</span><span class="p">,</span> <span class="mi">1753</span><span class="p">,</span> <span class="mi">1756</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1760</span><span class="p">,</span> <span class="mi">1763</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1778</span><span class="p">,</span> <span class="mi">1781</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1832</span><span class="p">,</span> <span class="mi">1835</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1841</span><span class="p">,</span> <span class="mi">1844</span><span class="p">),</span>
 <span class="p">(</span><span class="s">'129e6622-1f82-4de0-a24d-427d513f005d'</span><span class="p">,</span> <span class="s">'&lt;'</span><span class="p">,</span> <span class="mi">1853</span><span class="p">,</span> <span class="mi">1856</span><span class="p">)]</span>
</code></pre></div></div>

<p>And you could imagine a third case of removing both the first and second,
and getting an answer of entirely the same Part in various directions.
And maybe these would be wanted? But at least to start, i didn’t implement anything
to search for these. Hey, I’m not exactly a biologist, but I’ll figure this out as I go!</p>

<h3 id="anyway-that-was-fun">Anyway, that was fun!</h3>