---
title: Git Log Visualisation With PEW PEW!
tags: 
  - ruby-on-rails
categories: 
  - old-archive
excerpt_separator: "<!--more-->"
---

> This is a seriously old post. I'd probably not bother reading it, but I keep it here for nostalgia.

# Git Log Visualisation With PEW PEW!

[Gource](http://code.google.com/p/gource/) is an OpenGL capable visualisation tool for difference SCM’s, it’s easy to install and looks pretty damn cool. If you have [Homebrew](https://github.com/mxcl/homebrew) installed it’s simply:

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line">brew install gource
</span></code></pre></td>
</tr></table></div></figure>

Which happily goes off and grabs all the dependencies like pkg-config, libpng and others. Once it’s done, change into your projects directory (I’m assuming you use [git](http://git-scm.com/)) and then run gource.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line"><span class="nb">cd</span> /projects/myproject
</span><span class="line">gource
</span></code></pre></td>
</tr></table></div></figure>

Then watch the asteroids takes on space invaders style 3D visualisation of your developers doing their thaaang. If you want to make it cooler, then how about a little perl script to grab gravatars for each of your developers and make it more personal? On the gource google code project wiki there’s a script for just that, grab it off [this page](http://code.google.com/p/gource/wiki/GravatarExample) and save it as `grab_avatars.pl` and run it on your commandline.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line">perl grab_avatars.pl
</span></code></pre></td>
</tr></table></div></figure>

It’ll grab all of the gravatars for each developer and then save them in a `.git/avatars` folder, now we just need to tell gource to use that and we’re laffin’.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line">gource --user-image-dir .git/avatar/
</span></code></pre></td>
</tr></table></div></figure>

For an example of what it looks like, here’s a snippet of our project with some developers PEW PEWing at the code:

<embed src="http://s3.amazonaws.com/jamievandyke-blog-assets/pewpew-with-gource.mov" width="766px" height="694px" autoplay="false" controller="true" loop="false" href="http://s3.amazonaws.com/jamievandyke-blog-assets/pewpew-with-gource.mov" target="quicktimeplayer"></embed>

<time datetime="2010-11-18T22:26:02Z" pubdate data-updated="true">Nov 18<span>th</span>, 2010</time>

[Comments](/visualise-your-git-log-with-some-pew-pew-pew//blog/page/2/index.html#disqus_thread)

# [Engine Yard AppCloud Experiences](/what-an-engine-yard-appcloud-experience/)

## In Summary

As most of you probably know, I used to work at Engine Yard. The developers there are super intelligent and super knowledgeable and I loved working with them. Their vision for the AppCloud is a great idea in practise. To be able to scale your app as traffic increases and decreases, to replace broken server instances at the click of a button, this is the dream. Now the bad news. _It’s unstable, woefully under-tested and not production ready._

## In Detail

The client I was working with was working with medium sized clusters of instances, of varying sizes but usually along the lines of the following:

- 1 App server (64 bit CPU, 8 ECU’s, 15GB RAM, 1.7TB Persistent Storage)
- 1 DB Server (64 bit CPU, 13 ECU’s, 34GB RAM, 850GB Persistent Storage)
- 1 Mongo Instance (64 bit CPU, 20 ECU’s, 7GB RAM, 1.7TB Persistent Storage)
- 1 Util Instance Server (64 bit CPU, 13 ECU’s, 34GB RAM, 850GB Persistent Storage)

There are about 6-7 of these clusters, costing well into the 10’s of thousands a month. Don’t worry about the setup because I didn’t design it, the previous contractors did. We’re in the middle of a re-design now, e.g. we should 2 application servers for redundancy. We should be able to do this on the fly but this requires some manual DevOps to take place, specifically we need custom chef recipes to handle things that have been done manually, and this hasn’t been done yet.

Now in theory the AppCloud dashboard should also be able to clone an entire environment so you can run test deploys and more on it. In my experience those clones worked 1 day out of 7, the other times I was met with cryptic messages like _“There was an unexpected error while provisioning your instance.”_. Not terribly useful. Some of them worked sometimes, others failed. It turns out that for us the unexpected errors were one or all of the following at different times spread over a few weeks:

- Reaching the Amazon EC2 EBS Device storage limit (20TB)
- Reaching the Amazon EC2 API limit (due to having a few people using the dashboard at the same time)
- One of many unknown dashboard errors
- The data from a utility instance mounted on the application instance (!)

Let’s look at a few of these in detail.

### Reaching the Amazon EC2 EBS Device storage limit (20TB)

That’s a huge amount of data, I know! How did the client come to get so much data, well they used EBS snapshots as backups. That’s crazy considering they range from 256GB to 512GB, and they need to keep them all! Snapshots are done daily automatically, but usually someone will fire off a snapshot at some other time during the day. Meaning every day the amount of data that is held in snapshots increases by about 2TB, instead of being pushed up to S3 (another TODO that needs sorting). So the Amazon EC2 limit was hit, which is not Engine Yard’s fault but they should have some API warnings in the dashboard so we know what’s happening. Otherwise errors like the next one pop up too.

### Reaching the Amazon EC2 API limit

Too many queries at one time will get you temporarily banned from EC2 API queries and commands. That makes sense, but not when we don’t know how much of the query limit we’ve used. If you don’t grasp the horrors of not knowing, let’s just say if you bring the production cluster down for some quick maintenance (resizing EBS devices for example, which requires a terminate and rebuild) and you hit your limit before it comes back up….bad news, your cluster isn’t going to rebuild. On top of that, while it’s failing to rebuild you’ll probably terminate the cluster in the dashboard and try again which just prolongs the problem.

It turns out this problem was due to the dashboard making too many queries (polling Amazon for data regularly), note the past tense, the Engine Yard developers promise me that this error has gone away now.

### One of many unknown dashboard errors

On many occasions I would see a 500 error page when trying to use their dashboard, and when I asked their support guys what was up the answer was usually “We need to roll back a new version of the dashboard we just pushed”. Seriously, this happened on a multitude of times. You lose your confidence after number 3 I can tell you.

There needs to be more testing done for different scenarios if it’s going to break so regularly. From their point of view I guess they will say it’s because of the Amazon backend and the cryptic API responses they get sometimes. Sure, but I don’t care about that. If you choose to use Amazon EC2 as your backend for a cloud service, you better be damn sure you know exactly what’s what before using it!

### The data from a utility instance mounted on the application instance (!)

Oh, this one was just brilliant. 1 week the problem was that if you wanted to boot a cluster from old snapshots, the utility instances couldn’t actually re-attach old snapshots, it had to be done manually. That one sort of scared me a little because I was in the middle of production deploy testing when I was informed of it.

After they fixed the above issue in the dashboard, they introduced a brand new one. I cloned an environment and when it came online the utility instance had no data, which threw me. Until I looked on the application instance and found what should have been mounted to the utility instance. How exactly does this go wrong so badly?!

## However…

I’m obliged to say again, the developers and support people over at Engine Yard are fantastic, but clearly this product has a long way to go before it can be used for large applications of any sort. If you are working with a smaller application it makes sense to use them, but as your application grows, hope to the Gods that they’ve matured it slightly.

The moral of the story here is simple and something you all know already. _If you can’t rely on something 100% then don’t use it in production_. This is a hard lesson that my client has learnt and suffered greatly from. The client is (of course) trying out other services from different providers to move off the AppCloud platform. More on that in the future.

> > If you’re looking for help with Application deployment, please do [get in touch](mailto:jamie@fearoffish.com) and I’ll see what I can do for you.

<time datetime="2010-07-20T21:39:15+01:00" pubdate data-updated="true">Jul 20<span>th</span>, 2010</time>

[Comments](/what-an-engine-yard-appcloud-experience//blog/page/2/index.html#disqus_thread)

# [Local Hostnames the Easy Way](/ghost-and-local-hostnames/)

I was just reading [running rails local development with nginx, postgres, and passenger with homebrew](http://samsoff.es/post/running-rails-local-development-with-nginx-postgres-and-passenger-with-homebrew) on Sam Soffes site, and found it very useful for a quick copy and paste of his configs.

One thing that I found strange though is that he is using `/etc/hosts` to make local hostnames for web development. If you haven’t found out about the `ghost` gem yet:

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
<span class="line-number">9</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line">gem install ghost --no-ri --no-rdoc
</span><span class="line">Successfully installed ghost-0.2.4
</span><span class="line">1 gem installed
</span><span class="line">
</span><span class="line">ghost add blah.local
</span><span class="line"> <span class="o">[</span>Adding<span class="o">]</span> blah.local -&gt; 127.0.0.1
</span><span class="line">ghost list
</span><span class="line">Listing 1 host<span class="o">(</span>s<span class="o">)</span>:
</span><span class="line"> blah.local -&gt; 127.0.0.1
</span></code></pre></td>
</tr></table></div></figure>

No need to piss about with your `/etc/hosts` file anymore, this is as simple as it gets.

<time datetime="2010-02-08T18:26:02+00:00" pubdate data-updated="true">Feb 8<span>th</span>, 2010</time>

[Comments](/ghost-and-local-hostnames//blog/page/2/index.html#disqus_thread)

# [Railties Error?](/the-inherited-resources-railties-error/)

If you’re like me and you skip reading a gem’s README when you install the latest, you’ll be seriously sorry when you don’t for [inherited\_resources](http://github.com/josevalim/inherited_resources) because it is Rails 3 only. The same can be said for [responders](http://github.com/plataformatec/responders). You’ll get a big fat punch in the face with this error:

<figure class="code"><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td>
<td class="code"><pre><code class=""><span class="line">uninitialized constant Rails::Railtie</span></code></pre></td>
</tr></table></div></figure>

But fear not, just reverting to the following version will get you back on track:

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="s2">"inherited_resources"</span><span class="p">,</span> <span class="ss">:version</span> <span class="o">=&gt;</span> <span class="s2">"1.0.0"</span>
</span><span class="line"><span class="s2">"responders"</span><span class="p">,</span> <span class="ss">:version</span> <span class="o">=&gt;</span> <span class="s2">"0.4.0"</span>
</span></code></pre></td>
</tr></table></div></figure>

Now, on your bike. Get back to work!