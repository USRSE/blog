---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-10-03 01:27:00'
layout: post
original_url: https://vsoch.github.io//2019/regression-wasm/
title: Linear Regression with Web Assembly
---

<p>Web assembly can allow us to interact with compiled code directly in the browser,
doing away with any need for a server. While I don’t do a large amount of data analysis
for my role proper, I realize that many researchers do, and so with this in mind, 
I wanted to create a starting point for developers to interact with data in the browser.
The minimum conditions for success meant:</p>

<ol class="custom-counter">
 <li>being able to load a delimited file into the browser</li>
 <li>having the file render as a table</li>
 <li>having the data be processed by a compiled wasm</li>
 <li>updating a plot based on output from 3.</li>
</ol>

<p>And of course we would also want to start with dummy data, and provide input fields
to customize inputs and outputs.</p>

<h2 id="in-a-nutshell">In a Nutshell</h2>

<p>I’ve created <a href="https://vsoch.github.io/regression-wasm/" target="_blank">regression-wasm</a>,
a <a href="https://webassembly.org/" target="_blank">web assembly</a> (wasm) + GoLang (static!) 
application that can be used to run and plot a linear regression. It can serve as a starting point 
for some simple exploratory data analysis, or for a developer, a starting point to develop some 
custom GoLang-based data science tool that generally interacts with tabular data. We are using
this <a href="https://github.com/sajari/regression" target="_blank">regression library</a>, and
supporting both simple and multiple linear regression. To make it fun, I added a cute gopher. Because
<a href="https://gopherize.me" target="_blank">this site is amazing</a>. Here he is,
in all his glory:</p>

<div style="margin-bottom: 20px;">
   <img src="https://vsoch.github.io/regression-wasm/gopher.png" style="width: 30%;" />
</div>

<p>I’d really like to see others developing in Web Assembly, and making better static tools 
than are currently afforded, and this seems like a good way to do that. If a researcher publishes
a model, an interface should be provided to play with it. It needs to be static so we don’t need
to pay for a server, and can use something like GitHub pages. If we have these basic interfaces,
it becomes easier for developers to test more advanced things like streaming data, and interacting
with other input sources and resources.</p>

<h2 id="how-does-it-work">How does it work?</h2>

<p>Here is how it works. The data you input drives the graph produced.</p>

<ol class="custom-counter">
  <li>TLDR: Run a multiple or simple linear regression using Web Assembly</li>
  <li>Two variables (one predictor, one regressor) will generate a line plot showing X vs. Y and predictions</li>
  <li>More than two variables performs multiple regression to generate a residual histogram</li>
  <li>Upload your own data file, change the delimiter, the file name to be saved, or the predictor column</li>
</ol>

<h2 id="can-you-show-me-an-example">Can you show me an example?</h2>

<p>When you load the page, you are presented with a loaded data frame. The data is a bit dark - it’s trying
to show how poverty, employment, and population might relate to murder.
It’s actually a nice dataset to show how this works. The first column is the number of murders (per
million habitants) for some city, and each of the remaining columns are variables that might
be used to predict it (inhabitants, percent with incomes below $5000, and percent unemployed).</p>

<div style="padding: 20px;">
   <img src="https://raw.githubusercontent.com/vsoch/regression-wasm/master/img/basics.png" style="width: 100%;" />
</div>

<h3 id="formula">Formula</h3>

<p>The formula for our regression model is shown below the plot, in human friendly terms.</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>Predicted = -36.7649+ Inhabitants*0.0000 + Percent with incomes below $5000*1.1922 + Unemployed*4.7198
</code></pre></div></div>

<p>This comes directly from the model library, which is provided by <a href="https://www.github.com/sajari" target="_blank">@Sajari</a>.</p>

<h3 id="residuals">Residuals</h3>

<p>Given that we have more than one regressor variable, we need to run a multiple regression,
and so the plot in the upper right is a histogram of the residuals.</p>

<blockquote>
  <p>the residuals are the difference between the actual values (number of murders per million habitants) and the values predicted by our model.</p>
</blockquote>

<p>I’m not sure what the “best” plot to show here would be - we could do a matrix of plots,
but I wanted to keep it simple, and chose a histogram of the residuals.</p>

<h3 id="filtering">Filtering</h3>

<p>If you remove any single value from a row, it invalidates it, and it won’t be included
in the plot. If you remove a column heading, it’s akin to removing the entire column.</p>

<h3 id="plotting">Plotting</h3>

<p>But what if we want to plot the relationship between one of the variables X, and our Y?
This is where the tool gets interesting! By removing a column header, we essentially
remove the column from the dataset. Let’s first try removing just one, Inhabitants.
In the plot below, we still see a residual plot, and this is because there are still
three dimensions to plot.</p>

<div style="padding: 20px;">
   <img src="https://raw.githubusercontent.com/vsoch/regression-wasm/master/img/remove1.png" style="width: 100%;" />
</div>

<p><br /></p>

<p>Let’s remove another one, the percent unemployed, leaving only two dimensions.</p>

<div style="padding: 20px;">
   <img src="https://raw.githubusercontent.com/vsoch/regression-wasm/master/img/line-plot.png" style="width: 100%;" />
</div>

<p>Now we see a line plot, along with the plotting of the predictions! By simply removing
each column one at a time (and leaving only one Y, and one X) we are actually running
a single regression, and we can do this for each variable. Now it gets kind of fun! This
could actually be useful for someone:</p>

<p><br /></p>

<h4 id="inhabitants-to-predict-murders">Inhabitants to predict murders</h4>

<div style="padding: 20px;">
   <img src="https://raw.githubusercontent.com/vsoch/regression-wasm/master/img/inhabitants-predict-murders.png" style="width: 100%;" />
</div>

<p>Yeah, so this variable is fairly useless. The number of inhabitants of some city doesn’t really say anything
about murders.</p>

<p><br /></p>

<h4 id="unemployment-to-predict-murders">Unemployment to predict murders</h4>

<div style="padding: 20px;">
   <img src="https://raw.githubusercontent.com/vsoch/regression-wasm/master/img/unemployment-predict-murders.png" style="width: 100%;" />
</div>

<p>Now this is more interesting! There is definitely a correlation between unemployment and murders.</p>

<p><br /></p>

<h4 id="low-income-percentage-to-predict-murders">Low Income Percentage to predict murders</h4>

<div style="padding: 20px;">
   <img src="https://raw.githubusercontent.com/vsoch/regression-wasm/master/img/incomes-predict-murders.png" style="width: 100%;" />
</div>

<p>And logically, I would guess if unemployment is correlated with number of murders, income would be too.</p>

<h2 id="download-data">Download Data</h2>

<p>This of course is a very superficial overview, you would want to download the full model data to get more detail:
The “Download Result” button (shown in the images above) will appear after you generate any kind of plot, and it downloads a text file with the model output. Here is an example:</p>

<div class="language-text highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
Dinosaur Regression Wasm
Predicted = -36.7649 + Inhabitants*0.0000 + Percent with incomes below $5000*1.1922 + Percent unemployed*4.7198
Murders per one million inhabitants|Inhabitants|Percent with incomes below $5000|Percent unemployed
11.20|	587000.00|	16.50|	6.20
13.40|	643000.00|	20.50|	6.40
40.70|	635000.00|	26.30|	9.30
5.30|	692000.00|	16.50|	5.30
24.80|	1248000.00|	19.20|	7.30
12.70|	643000.00|	16.50|	5.90
20.90|	1964000.00|	20.20|	6.40
35.70|	1531000.00|	21.30|	7.60
8.70|	713000.00|	17.20|	4.90
9.60|	749000.00|	14.30|	6.40
14.50|	7895000.00|	18.10|	6.00
26.90|	762000.00|	23.10|	7.40
15.70|	2793000.00|	19.10|	5.80
36.20|	741000.00|	24.70|	8.60
18.10|	625000.00|	18.60|	6.50
28.90|	854000.00|	24.90|	8.30
14.90|	716000.00|	17.90|	6.70
25.80|	921000.00|	22.40|	8.60
21.70|	595000.00|	20.20|	8.40
25.70|	3353000.00|	16.90|	6.70

N = 20
Variance observed = 92.76010000000001
Variance Predicted = 75.90724706481737
R2 = 0.8183178658153383
</code></pre></div></div>

<p>Mind you I’m a terrible data scientist. I’ll leave this up to you!</p>

<h2 id="how-would-i-do-it-over">How would I do it over?</h2>

<p>As soon as I realized that I wanted data and functions to update dynamically,
I regretted not using Vue.js, so if I did this over, I’d use it. A few issues that I’d like to figure out
are:</p>

<ol class="custom-counter">
 <li>maintaining a state from some previously built model or loaded data</li>
 <li>starting with a pre-built model to predict something</li>
 <li>optimizing sending data to and from JavaScript to GoLang</li>
</ol>

<p>If anyone has ideas about the above, please let me know! I’ll probably try working
on some of the above for some future fun project. I’m also looking for an excuse to 
develop an API of web assembly functions, so if you are a researcher and need a thing,
please reach out!</p>

<h2 id="development-and-questions">Development and Questions!</h2>

<p>For notes on development (local and Docker based) please <a href="https://www.github.com/vsoch/regression-wasm" target="_blank">see the repository</a>. If you need any help, or want to request a custom tool, 
please don’t hesitate to <a href="https://www.github.com/vsoch/regression-wasm/issues" target="_blank">open up an issue</a>.</p>