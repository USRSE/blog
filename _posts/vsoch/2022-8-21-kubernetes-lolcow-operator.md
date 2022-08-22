---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io/
category: vsoch
date: '2022-08-21 12:30:00'
layout: post
original_url: https://vsoch.github.io/2022/kubernetes-lolcow-operator/
title: My First Kubernetes Operator
---

<p>I’m hoping to eventually help out on projects that are working with Kubernetes, and I don’t mean writing yaml configs for apps (I’ve done this a few times before),
but rather developing <em>for</em> Kubernetes. This is something that before last weekend, I had never done. But I love Go, and I was excited, so I dived in!
This post will detail how I went about creating my first Kubernetes operator - the lolcow operator, which was a challenge presented to me by
my long time friend and colleague <a href="https://twitter.com/CarlosEArango" target="_blank">Eduardo</a>. I got it done in a little over a week, and that includes not actually working
on it during the workweek, but the two weekends around it, so I’m pretty pleased. I also want to say thank you to <a href="https://twitter.com/mogsie" target="_blank">@mogsie</a> in the Kubernetes
slack #kubebuilder channel for some of his good practices wisdom. There is a lot to learn in this space, and wisdom from those that are experienced
is invaluable. Don’t care to read? Browse the ⭐️ <a href="https://github.com/vsoch/lolcow-operator/tree/881d4a13ff31aa5261fd2fdba5d8eba726669e66" target="_blank">lolcow-operator</a> ⭐️ at the state when I wrote this article.</p>

<h2 id="using-the-operator">Using the Operator</h2>

<p>If you are a developer and just want to try things out, here is a quick summary of how to do that.
The complete instructions are in <a href="https://github.com/vsoch/lolcow-operator/tree/881d4a13ff31aa5261fd2fdba5d8eba726669e66" target="_blank">README</a>
and I’ll abbreviate here. If you haven’t ever installed minikube, you can follow the <a href="https://minikube.sigs.k8s.io/docs/start/" target="_blank">install instructions here</a>.</p>

<h3 id="1-build-and-deploy">1. Build and Deploy</h3>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># Clone the source code</span>
<span class="nv">$ </span>git clone https://github.com/vsoch/lolcow-operator
<span class="nv">$ </span><span class="nb">cd </span>lolcow-operator

<span class="c"># Start a minikube cluster</span>
<span class="nv">$ </span>minikube start

<span class="c"># Build the operator</span>
<span class="nv">$ </span>make

<span class="c"># How to make your manifests</span>
<span class="nv">$ </span>make manifests

<span class="c"># And install. This places an executable "bin/kustomize"</span>
<span class="nv">$ </span>make <span class="nb">install</span>
</code></pre></div></div>

<p>I’ve been told that <a href="https://kind.sigs.k8s.io/" target="_blank">kind</a> is a better tool to use, so I’ll try this next time.
At this point we can apply a <a href="https://github.com/vsoch/lolcow-operator/blob/881d4a13ff31aa5261fd2fdba5d8eba726669e66/config/samples/_v1alpha1_lolcow.yaml" target="_blank">our lolcow config</a> 
that has information about the greeting to show, and the port.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>bin/kustomize build config/samples | kubectl apply <span class="nt">-f</span> -
lolcow.my.domain/lolcow-pod created
</code></pre></div></div>

<p>And finally we can run it!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>make run
</code></pre></div></div>

<h3 id="2-view-the-lolcow">2. View the Lolcow?</h3>

<p>And you should be able to open the web-ui (when kubectl get svc shows the lolcow service is “ready” and not “pending”).</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>minikube service lolcow-pod
</code></pre></div></div>

<p>Ha!</p>

<div style="padding: 20px;">
 <img src="https://raw.githubusercontent.com/vsoch/lolcow-operator/881d4a13ff31aa5261fd2fdba5d8eba726669e66/img/hello-lolcow.png" />
</div>

<p>At this point you might say, “Wait, that’s not the lolcow, that’s the Nyan cat!” and you are right. 🎨️ Thank you 🎨️ and shout out 
to <a href="https://codepen.io/eusonic/pen/nrjqKn" target="_blank">eusonic</a> for the css that drives this UI! I was able to take it
and modify it into a containerized Flask application (with added text that can dynamically change). And it’s actually a container you
can run isolated from this application, e.g.,:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>docker run <span class="nt">-p</span> 8080:8080 <span class="nt">-it</span> ghcr.io/vsoch/lolcow-operator  <span class="s2">"Were you expecting someone else?"</span>
</code></pre></div></div>
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code> __________________________________
&lt; Were you expecting someone <span class="k">else</span>? <span class="o">&gt;</span>
 <span class="nt">----------------------------------</span>
        <span class="se">\ </span>  ^__^
         <span class="se">\ </span> <span class="o">(</span>oo<span class="o">)</span><span class="se">\_</span>______
            <span class="o">(</span>__<span class="o">)</span><span class="se">\ </span>      <span class="o">)</span><span class="se">\/\</span>
                <span class="o">||</span><span class="nt">----w</span> |
                <span class="o">||</span>     <span class="o">||</span>
 <span class="k">*</span> Serving Flask app <span class="s1">'app'</span>
 <span class="k">*</span> Debug mode: off
WARNING: This is a development server. Do not use it <span class="k">in </span>a production deployment. Use a production WSGI server instead.
 <span class="k">*</span> Running on all addresses <span class="o">(</span>0.0.0.0<span class="o">)</span>
 <span class="k">*</span> Running on http://127.0.0.1:8080
 <span class="k">*</span> Running on http://172.17.0.2:8080
Press CTRL+C to quit
</code></pre></div></div>

<p>You can see the container (an automated build from the repository) on <a href="https://github.com/vsoch/lolcow-operator/pkgs/container/lolcow-operator" target="_blank">GitHub packages</a>.
And a challenge for new folks! When you run the operator, try getting the name of the pod and seeing if you can figure out how to see logs and then see this same entrypoint (with your message).</p>

<h3 id="3-test-the-operator">3. Test the Operator</h3>

<p>We are good so far that the operator has created the deployment and service, and we can see it in our browser. If you look at the operator logs, you’ll also see
that it is giving you detail about state (with emojis, because I added emojis to code and I’m a monster 🤣️). If you were to Control+C and restart the controller, you’d see the greeting hasn’t changed:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>1.6611115156486864e+09	INFO	👋️ No Change to Greeting! 👋️: ...
</code></pre></div></div>

<p>But we want to test this. E.g., if we edit our control resource and change the greeting or port, we want it to be the case that the user interface is updated too!
First to change the greeting, we can update the sample config:</p>

<div class="language-diff highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">apiVersion: my.domain/v1alpha1
kind: Lolcow
metadata:
</span>  name: lolcow-pod
<span class="p">spec:
</span>  port: 30685
<span class="gd">-  greeting: Hello, this is a message from the lolcow!
</span><span class="gi">+ greeting: What, you've never seen a poptart cat before?
</span></code></pre></div></div>

<p>I made this change while it’s running (in a separate terminal) and then re-applied the config:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>bin/kustomize build config/samples | kubectl apply <span class="nt">-f</span> -
lolcow.my.domain/lolcow-pod configured
</code></pre></div></div>

<p>The change might be quick, but if you scroll up you should see:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>1.6611116913288918e+09	INFO	👋️ New Greeting! 👋️: 	...
 "What, you've never seen a poptart cat before?": "Hello, this is a message from the lolcow!"}
</code></pre></div></div>

<p>and the interface should change too!</p>

<div style="padding: 20px;">
 <img src="https://raw.githubusercontent.com/vsoch/lolcow-operator/881d4a13ff31aa5261fd2fdba5d8eba726669e66/img/poptart-cat.png" />
</div>

<p>And then we can do the same to change the port - first seeing that (without changes) updating the config reflects this state:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>1.6611135948097324e+09	INFO	🔁 No Change to Port! 🔁:
</code></pre></div></div>

<p>And now to change it - let’s try one number higher:</p>

<div class="language-diff highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">apiVersion: my.domain/v1alpha1
kind: Lolcow
metadata:
</span>  name: lolcow-pod
<span class="p">spec:
</span><span class="gd">-  port: 30685
</span><span class="gi">+  port: 30686
</span></code></pre></div></div>

<p>And then apply the config.</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>... "🔁 New Port! 🔁"
</code></pre></div></div>

<p>Refreshing the current browser should 404, and you should be able to tweak the port number in your browser and see the user interface again!
Yay, it works! I can’t tell you how happy I was when this finally worked, there were many bugs that I had to work through that I’ll discuss shortly.</p>

<h3 id="4-cleanup">4. Cleanup</h3>

<p>When cleaning up, you can control+C to kill the operator from running, and then:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># Make sure before deleting all services and pods these are the only ones running!</span>
<span class="nv">$ </span>kubectl delete pod <span class="nt">--all</span>
<span class="nv">$ </span>kubectl delete svc <span class="nt">--all</span>
<span class="nv">$ </span>minikube stop
</code></pre></div></div>

<p>And that’s it! You can also delete your minikube cluster if you like.</p>

<h2 id="whats-happening">What’s Happening?</h2>

<p>So I’m not an expert, but I can talk generally about what the operator is doing. First, I think it’s called an operator in reference to a <em>human</em> operator. Before we had operators, it was up to
humans to deploy things and then manage state changes. From <a href="https://www.redhat.com/en/topics/containers/what-is-a-kubernetes-operator" target="_blank">the RedHat site</a>:</p>

<blockquote>
  <p>A Kubernetes operator is an application-specific controller that extends the functionality of the Kubernetes API to create, configure, and manage instances of complex applications on behalf of a Kubernetes user.</p>
</blockquote>

<p>So given that you have some set of pods / services / deployments (anything Kubernetes related?) that you want to manage, I imagine an operator being like a continuous loop that is constantly checking the current
state, comparing it to some desired state, and then adjusting as needed. The extra config file that we created is called a custom resource (CR) that helps the user (me or you!) to manage that state. In our case,
we cared about the lolcow greeting, and the port it’s deployed on, and I explicitly chose these two things to have a variable to watch and update for each of the deployment and service.</p>

<p>If I understand correctly, operators help in the case when an application has a state. A stateless app (in a container) doesn’t need any kind of special management, because you can just deploy it and re-create it.
But something like a database couldn’t just be recreated (without messing something up). Since we are always moving toward a desired state, this is why the main function of a controller is called “Reconcile,” which
is another term to describe that. Finally, <a href="https://joshrosso.com/docs/2019/2019-10-13-controllers-and-operators/" target="_blank">this article</a> offers a nice definition of operators and controllers.
TLDR: an operator is a kind of controller, but not necessarily vice versa. A cool place to browse (to get a sense of different kinds of operators) is the <a href="https://operatorhub.io/" target="_blank">Operator Hub</a>.</p>

<h2 id="design-decisions">Design Decisions</h2>

<p>You can read my entire development process (starting from zero to this application) in the <a href="https://github.com/vsoch/lolcow-operator/tree/881d4a13ff31aa5261fd2fdba5d8eba726669e66" target="_blank">README</a>,
and here I briefly want to share some design decisions that I started to make. I suspect this set will change as I learn more.</p>

<h3 id="named-apis">Named APIs</h3>

<p>The original template had just one API, and organized by versions. I anticipate that if an application wants to have more than one named API, it makes
sense to introduce another level of organization there, so that’s what I did.</p>

<div class="language-console highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="go">api/
└── lolcow        (- lolcow api namespace
    └── v1alpha1
        ├── groupversion_info.go
        ├── lolcow_types.go
        └── zz_generated.deepcopy.go
</span></code></pre></div></div>

<p>I realized that if we want more than one controller (likely), we should have subdirectories in controllers too. I mirrored the kueue design and made one called “core.”</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>controllers/
├── core
│   └── core.go
└── lolcow
    ├── deployment.go
    ├── lolcow_controller.go
    ├── service.go
    └── suite_test.go
</code></pre></div></div>

<h3 id="named-packages">Named Packages</h3>

<p>By default, we didn’t have any packages. But what if the API needs some custom (and shared) logic? I was thinking that there are probably two cases here - importing the guts of the thing that you want to control, or defining it here. For the latter, 
I decided to try creating a pkg directory.</p>

<div class="language-console highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="gp">$</span><span class="w"> </span>tree pkg/
<span class="go">pkg/
└── lolcow
    └── lolcow.go
</span></code></pre></div></div>
<p>For my lolcow operator, this is overkill. But if different versions of an API might want to use shared code, this would be helpful. And indeed there could be other reasons to have nicely
organized packages. I also noticed that some operators (like 
<a href="https://github.com/kubernetes-sigs/kueue/tree/e571d42e390f96a95efa799d720777e92e4f69a4/pkg" target="_blank">kueue</a> put a lot of the controller logic under the pkg directory.
I think this is something that could be done if there is a lot of additional code to add to the operator, and the top level gets crowded. However, given that many people are using this
common SDK to generate the template, I decided to leave as is, so it would be more easily recognized (and thus easier to develop).</p>

<h3 id="naming">Naming</h3>

<p>In most examples using an API, the names were like <code class="language-plaintext highlighter-rouge">mydomainv1alpha1</code> to reference the API package, and I suspect this is done to maintain some sense of namespacing the APIs. Maybe it could even be that more than one version
is used per application? Since my application is just using one and isn’t expected to be extended or changed, I opted for something simple like <code class="language-plaintext highlighter-rouge">api</code>.</p>

<h2 id="bugs">Bugs</h2>

<p>So what prevented me from finishing this entirely the first weekend? The bugs, of course! 🐛️
I’ll review them here briefly, in case anyone else runs into them. I suspect they will,
because when you haven’t done this before there is really no way to know these things unless you stumble on it,
add the right thing by accident, or (what I don’t do) read an entire book before writing a line of code.</p>

<h4 id="service--deployment-detection">Service / Deployment Detection</h4>

<p>For the longest time, the original service and deployment would start (because they were not found) but they would <em>continue</em> to be not found
and sort of spiral into a chain of error messages. This took me many evenings to figure out, but it comes down to these (sort of hidden) lines
at the top of the controllers file:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>//+kubebuilder:rbac:groups=my.domain,resources=lolcows,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups=my.domain,resources=lolcows/status,verbs=get;update;patch
//+kubebuilder:rbac:groups=my.domain,resources=lolcows/finalizers,verbs=update
//+kubebuilder:rbac:groups=my.domain,resources=deployments,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups=my.domain,resources=pods,verbs=get;list;watch;create;
//+kubebuilder:rbac:groups=my.domain,resources=services,verbs=get;list;watch;create;update;patch;delete
</code></pre></div></div>

<p>I honestly didn’t even notice they were there at first. The template only had the first three (for lolcows) and I needed to add the last three, giving my contoller permission (RBAC refers
to a set of rules that represent a set of permissions) to interact with services and deployments. I think what was happening
before is that my controller couldn’t see them, period, so of course the Get always failed. I found <a href="https://cluster-api.sigs.k8s.io/developer/providers/implementers-guide/controllers_and_reconciliation.html" target="_blank">this page</a> 
and <a href="https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole" target="_blank">this page</a> useful for learning about this.
Another important note (that I didn’t do here) is that you can namespace these, which I suspect is best practice but I didn’t do for this little demo. The other bit that
seemed important was to say that my controller owned services and deployments:</p>

<div class="language-go highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">LolcowReconciler</span><span class="p">)</span> <span class="n">SetupWithManager</span><span class="p">(</span><span class="n">mgr</span> <span class="n">ctrl</span><span class="o">.</span><span class="n">Manager</span><span class="p">)</span> <span class="kt">error</span> <span class="p">{</span>
	<span class="k">return</span> <span class="n">ctrl</span><span class="o">.</span><span class="n">NewControllerManagedBy</span><span class="p">(</span><span class="n">mgr</span><span class="p">)</span><span class="o">.</span>
		<span class="n">For</span><span class="p">(</span><span class="o">&amp;</span><span class="n">api</span><span class="o">.</span><span class="n">Lolcow</span><span class="p">{})</span><span class="o">.</span>
		<span class="n">Owns</span><span class="p">(</span><span class="o">&amp;</span><span class="n">appsv1</span><span class="o">.</span><span class="n">Deployment</span><span class="p">{})</span><span class="o">.</span>
		<span class="n">Owns</span><span class="p">(</span><span class="o">&amp;</span><span class="n">corev1</span><span class="o">.</span><span class="n">Service</span><span class="p">{})</span><span class="o">.</span>
		<span class="c">// Defaults to 1, putting here so we know it exists!</span>
		<span class="n">WithOptions</span><span class="p">(</span><span class="n">controller</span><span class="o">.</span><span class="n">Options</span><span class="p">{</span><span class="n">MaxConcurrentReconciles</span><span class="o">:</span> <span class="m">1</span><span class="p">})</span><span class="o">.</span>
		<span class="n">Complete</span><span class="p">(</span><span class="n">r</span><span class="p">)</span>
<span class="p">}</span>
</code></pre></div></div>
<p>Another level of detail that I didn’t get into was <a href="https://sdk.operatorframework.io/docs/building-operators/golang/references/event-filtering/" target="_blank">Events</a>, which
I suspect gives you better control (and could also be a source of error if you aren’t watching the right ones). I didn’t get a chance to play around with different derivations
of the above yet, because as soon as the above worked I was 😁️.</p>

<h4 id="pod-names">Pod Names</h4>

<p>For some reason, at one point I switched my name from ‘lolcow-sample’ to ‘lolcow-pod’, and although I thought I cleaned everything up,
whenever I’d create the cluster again it would show me <em>two</em> pods made, one lolcow-pod and one lolcow-sample. I had to try resetting and
“starting fresh” multiple times (e.g., deleting stuff in bin and reinstalling everything) until <code class="language-plaintext highlighter-rouge">kubectl get pod</code> didn’t show the older
and new name. If you run into errors about not finding a service, it could be that somewhere the older name is still being created or referenced,
so it’s a good sanity check to do.</p>

<h2 id="summary">Summary</h2>

<p>What a fun time working on this! Despite the complexity, because I love Go so much, it was truly a pleasure to debug.
What I learned most from this experience is that details are really important. The smallest detail about a permission/scope,
or how you configure your operator can mean it won’t work as you expect, and it takes a lot of careful testing to even get it working.</p>

<p>Now some caveats - this is my first time doing any kind of development for Kubernetes, and this is a very basic intro
that doesn’t necessarily reflect best practices. For example, I’m just using the default namespace, and I’m aware that for a more realistic
setup we’d want to be using more controlled namespaces. But I think this is a good start? When I’m newly learning something, my main goal
is to get it to work (period!) and then to slowly learn better practices over time (and use them
as a standard). I suspect future me will look at this and wonder what I was thinking? Anyway, I hope this has been useful to you!</p>