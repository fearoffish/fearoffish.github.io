---
title: Web Development With Source Code Management A Basic Guide
tags: 
  - ruby-on-rails
categories: 
  - old-archive
excerpt_separator: "<!--more-->"
---
> This is a seriously old post. I'd probably not bother reading it, but I keep it here for nostalgia.

# Web Development With SCM Basics

## Pre-Amble

I remember when I first started development. Even more fun was doing the same but over a remote ftp connection through Transmit, or just directly on the server. I got used to pretty GUI’s doing things for me and visually feedback for whether I’d got it right. Oh, how things have changed. My work flow was:

- Make changes to my code
- Flip to my browser and refresh
- See what I’ve done has changed
- Go back to my code and make more changes
- Go back to my browser and refresh
- See I broke stuff
- Go back and try fix it
- etc.

I want to take you through how a basic version of what I do now, and the benefits of what I do. I have confidence in my changes and a general feeling of happiness that I’d like to pass on to you. Whether or not I work in a team, I use distributed version managed code to do so. Any changes I make locally are pushed to a remote computer, why? What if my hard drive failed or I’m working on another machine for the day, you never know. Either way, I have access to my code, including all the changes I’ve made.

Why was my initial plan inefficient? Well, it was brittle and to test everything meant looking at all the pages sequentially. Making changes in one place might affect another and I had to double check it manually. Also, the changes I made weren’t reversible. I worked around that by saving multiple copies of my files or folders, keeping old version of sites hanging around so I could always revert back if I wanted. _This just doesn’t scale_.

What’s the alternative? Source Code Management. There are many systems around and I’ve certainly used a myriad of them, but I’ve settled on [git](http://www.git-scm.org/) which makes me warm and fuzzy inside. Git was developed initially by Linus Torvalds to manage the complicated process of building a Linux kernel in a distributed manner (multiple people, multiple places editing one set of files). Quick list of benefits we’re interested in:

- Revert changes without saving different versions manually
- Develop multiple version of your site concurrently
- Log changes you’ve made without a separately managed log file

Enough talk, let’s just run through what I do. I use [Ruby on Rails](http://www.rubyonrails.org/) and [Sinatra](http://www.sinatrarb.com/) for web development, but this will work with most languages too, if not all.

## Getting Setup

Create a new project in your tool of choice, I’ll create one called ‘scm\_project’ with Ruby on Rails:

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line"><span class="nv">$ </span>rails scm_project
</span><span class="line">create
</span><span class="line">create app/controllers
</span><span class="line">...
</span><span class="line">create log/test.log
</span></code></pre></td>
</tr></table></div></figure>

This is just a plain old Ruby on Rails project, but it could be anything for you, even just a folder with some php files. We need to initialise the folder with our source code manager ‘git’.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line"><span class="nv">$ </span><span class="nb">cd </span>scm_project
</span><span class="line">scm_project <span class="nv">$ </span>git init
</span><span class="line">Initialized empty Git repository in /a/demos/scm_project/.git/
</span></code></pre></td>
</tr></table></div></figure>

So git has set up our folder as a version managed folder. We can now, for example, ask git for the status of this folder:

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
<span class="line-number">10</span>
<span class="line-number">11</span>
<span class="line-number">12</span>
<span class="line-number">13</span>
<span class="line-number">14</span>
<span class="line-number">15</span>
<span class="line-number">16</span>
<span class="line-number">17</span>
<span class="line-number">18</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line"><span class="nv">$ </span>git status
</span><span class="line"><span class="c"># On branch master</span>
</span><span class="line"><span class="c">#</span>
</span><span class="line"><span class="c"># Initial commit</span>
</span><span class="line"><span class="c">#</span>
</span><span class="line"><span class="c"># Untracked files:</span>
</span><span class="line"><span class="c"># (use "git add &lt;file&gt;..." to include in what will be committed)</span>
</span><span class="line"><span class="c">#</span>
</span><span class="line"><span class="c"># README</span>
</span><span class="line"><span class="c"># Rakefile</span>
</span><span class="line"><span class="c"># app/</span>
</span><span class="line"><span class="c"># config/</span>
</span><span class="line"><span class="c"># doc/</span>
</span><span class="line"><span class="c"># log/</span>
</span><span class="line"><span class="c"># public/</span>
</span><span class="line"><span class="c"># script/</span>
</span><span class="line"><span class="c"># test/</span>
</span><span class="line">nothing added to commit but untracked files present <span class="o">(</span>use <span class="s2">"git add"</span> to track<span class="o">)</span>&lt;/pre&gt;
</span></code></pre></td>
</tr></table></div></figure>

Don’t worry, this is nothing scary. It’s just telling me that this folder has some unmanaged (‘untracked’) files. Source code managers need to told which files they’re managing. In this case we just need to tell git what to add and what to ignore. Some things shouldn’t be added, for example a database credentials file. If you have multiple people on a project and you all have your own local database server, then there’s no guarantee that the access details are the same, so by leaving it out (telling git to ignore it) it will never be added and everyone can have their own version without it affecting others.

Let’s set up our file ignore list. Create a file in the root of that folder called `.gitignore` and put in contents like this:

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line">log/*.log
</span><span class="line">tmp/
</span><span class="line">.DS_Store
</span><span class="line">doc/api
</span><span class="line">doc/app
</span><span class="line">doc/plugins
</span><span class="line">db/*.sqlite3
</span><span class="line">config/database.yml
</span></code></pre></td>
</tr></table></div></figure>

This is my base `.gitignore` file, you can add to this later as you go. The idea is that anything you don’t want version managed, you add to this file.

Next up, we need to tell the version management to add all files.

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
<span class="line-number">10</span>
<span class="line-number">11</span>
<span class="line-number">12</span>
<span class="line-number">13</span>
<span class="line-number">14</span>
<span class="line-number">15</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line"><span class="nv">$ </span>git add .
</span><span class="line"><span class="nv">$ </span>git status
</span><span class="line"><span class="c"># On branch master</span>
</span><span class="line"><span class="c">#</span>
</span><span class="line"><span class="c"># Initial commit</span>
</span><span class="line"><span class="c">#</span>
</span><span class="line"><span class="c"># Changes to be committed:</span>
</span><span class="line"><span class="c"># (use "git rm --cached &lt;file&gt;..." to unstage)</span>
</span><span class="line"><span class="c">#</span>
</span><span class="line"><span class="c"># new file: .gitignore</span>
</span><span class="line"><span class="c"># new file: README</span>
</span><span class="line">...
</span><span class="line"><span class="c"># new file: test/performance/browsing_test.rb</span>
</span><span class="line"><span class="c"># new file: test/test_helper.rb</span>
</span><span class="line"><span class="c">#</span>
</span></code></pre></td>
</tr></table></div></figure>

So by running `git add .` we’re telling git to add everything within this directory. `.` means ‘this directory’ in linux talk. That’s easy enough. Adding files to git means you’re telling it what you want to store in the system, but you need to ‘commit’ them too. Committing the changes that you’ve racked up tells the version system to store them.

The pattern is generally to rack up related changes, and then commit them with a message that logs those changes. In the future you can look at those logs and see what changes were made, even over multiple files, because of the way you add them.

The changes I’ve added here are simply adding a bare project, so I’m going to commit them now with a relevant message.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line"><span class="nv">$ </span>git commit -m <span class="s1">'initial bare rails project'</span>
</span><span class="line"><span class="o">[</span>master <span class="o">(</span>root-commit<span class="o">)</span> a085392<span class="o">]</span> initial bare rails project
</span><span class="line"> 41 files changed, 8438 insertions<span class="o">(</span>+<span class="o">)</span>, 0 deletions<span class="o">(</span>-<span class="o">)</span>
</span><span class="line"> create mode 100644 .gitignore
</span><span class="line"> create mode 100644 README
</span><span class="line"> ...
</span><span class="line"> create mode 100644 <span class="nb">test</span>/performance/browsing_test.rb
</span><span class="line"> create mode 100644 <span class="nb">test</span>/test_helper.rb
</span></code></pre></td>
</tr></table></div></figure>

After running the command, notice that git has told me how many insertions and deletions it’s accomplished. These are line counts. Where are we now? We’ve added the files we want the version management system to track, we can add more later, and we’ve committed the first set of changes to those files. Let’s hope through a few more changes and see what happens.

## Making Changes

I’m going to change my application’s README file.

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
<span class="line-number">10</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line"><span class="nv">$ </span><span class="nb">echo</span> <span class="s1">'My New README'</span> &gt; README
</span><span class="line"><span class="nv">$ </span>git status
</span><span class="line"><span class="c"># On branch master</span>
</span><span class="line"><span class="c"># Changed but not updated:</span>
</span><span class="line"><span class="c"># (use "git add &lt;file&gt;..." to update what will be committed)</span>
</span><span class="line"><span class="c"># (use "git checkout -- &lt;file&gt;..." to discard changes in working directory)</span>
</span><span class="line"><span class="c">#</span>
</span><span class="line"><span class="c"># modified: README</span>
</span><span class="line"><span class="c">#</span>
</span><span class="line">no changes added to commit <span class="o">(</span>use <span class="s2">"git add"</span> and/or <span class="s2">"git commit -a"</span><span class="o">)</span>
</span></code></pre></td>
</tr></table></div></figure>

Our status shows me that I’ve got changes in the README but I’ve not added them to a commit. So this highlights an important fact, adding files to git doesn’t mean ‘add the file’ it means ‘add the changes’. Let’s prove this fact by adding our changed README.

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
<span class="line-number">10</span>
<span class="line-number">11</span>
<span class="line-number">12</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line"><span class="nv">$ </span>git add README
</span><span class="line"><span class="nv">$ </span>git status
</span><span class="line"><span class="c"># On branch master</span>
</span><span class="line"><span class="c"># Changes to be committed:</span>
</span><span class="line"><span class="c"># (use "git reset HEAD &lt;file&gt;..." to unstage)</span>
</span><span class="line"><span class="c">#</span>
</span><span class="line"><span class="c"># modified: README</span>
</span><span class="line"><span class="c">#</span>
</span><span class="line"><span class="nv">$ </span>git commit -m <span class="s1">'showing what modifying files that are tracked does'</span>
</span><span class="line"><span class="o">[</span>master b9b57e5<span class="o">]</span> showing what modifying files that are tracked does
</span><span class="line"> 1 files changed, 1 insertions<span class="o">(</span>+<span class="o">)</span>, 243 deletions<span class="o">(</span>-<span class="o">)</span>
</span><span class="line"> rewrite README <span class="o">(</span>100%<span class="o">)</span>
</span></code></pre></td>
</tr></table></div></figure>

Our status changed from ‘Changed but not updated’ to ‘Changes to be committed’. Changes can be over multiple files and be added in groups, what’s important is that when you commit those changes they must be relevant to each other. Notice also that git has counted the new lines I’ve added as well as the ones I deleted.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line">1 files changed, 1 insertions<span class="o">(</span>+<span class="o">)</span>, 243 deletions<span class="o">(</span>-<span class="o">)</span>
</span></code></pre></td>
</tr></table></div></figure>

The pattern I generally follow is:

- Make changes for a feature or bugfix
- Ensure my tests are passing
- Add the changes for that commit
- Commit them as a group describing what I changed overall

Some people like to describe it on a per file basis, but for me it comes down to a feature or bugfix. ’ **why** did I change it’, not ’ **what** did I change’. This is an important fact to note because the version manager is already logging ‘what’ changed, so all that remains is ‘why’.

## Looking at Changes

What is it logging? Let’s take a look:

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
<span class="line-number">10</span>
<span class="line-number">11</span>
<span class="line-number">12</span>
<span class="line-number">13</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line"><span class="nv">$ </span>git log
</span><span class="line">2009-08-30 21:15:10 fearoffish ttys001
</span><span class="line">commit b9b57e53fa0bdcd29b14fb0ded6ab4355ef3cb92
</span><span class="line">Author: Jamie van Dyke &lt;fearoffish@fearofpro.local&gt;
</span><span class="line">Date: Fri Aug 28 23:03:31 2009 +0100
</span><span class="line">
</span><span class="line"> showing what modifying files that are tracked does
</span><span class="line">
</span><span class="line">commit a08539202e8bf908106e1736dabf588c3b367151
</span><span class="line">Author: Jamie van Dyke &lt;fearoffish@fearofpro.local&gt;
</span><span class="line">Date: Fri Aug 28 21:58:47 2009 +0100
</span><span class="line">
</span><span class="line"> initial bare rails project
</span></code></pre></td>
</tr></table></div></figure>

What we see is who committed changes, when, and what they logged as the message. You also get a reference id to the commit, which is a 40 character SHA-1 value of the commit contents. When you reference a commit in the future you use this id, like others I mostly just use 6 characters as that’s more than likely the amount of characters you need to uniquely identify it. The beauty of the `git log` is that if you keep good logs of the changes you’ve made and you group your commits correctly, then you can revert blocks of changes as required.

## Reverting Changes

Let’s revert the last commit and see how easy it is to go back to previous version of your app.

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
<span class="line-number">10</span>
<span class="line-number">11</span>
<span class="line-number">12</span>
<span class="line-number">13</span>
<span class="line-number">14</span>
<span class="line-number">15</span>
<span class="line-number">16</span>
<span class="line-number">17</span>
<span class="line-number">18</span>
<span class="line-number">19</span>
<span class="line-number">20</span>
<span class="line-number">21</span>
<span class="line-number">22</span>
<span class="line-number">23</span>
<span class="line-number">24</span>
<span class="line-number">25</span>
<span class="line-number">26</span>
<span class="line-number">27</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line"><span class="nv">$ </span>git revert b9b57e
</span><span class="line">2009-08-30 21:20:59 fearoffish ttys001
</span><span class="line">Finished one revert.
</span><span class="line"><span class="o">[</span>master 47d2c28<span class="o">]</span> Revert <span class="s2">"showing what modifying files that are tracked does"</span>
</span><span class="line"> 1 files changed, 243 insertions<span class="o">(</span>+<span class="o">)</span>, 1 deletions<span class="o">(</span>-<span class="o">)</span>
</span><span class="line"> rewrite README <span class="o">(</span>100%<span class="o">)</span>
</span><span class="line"><span class="nv">$ </span>git log
</span><span class="line">2009-08-30 21:33:04 fearoffish ttys001
</span><span class="line">commit 47d2c28a1c90ed95a32c5e76fa6e669b89ca5f09
</span><span class="line">Author: Jamie van Dyke &lt;fearoffish@fearofpro.local&gt;
</span><span class="line">Date: Sun Aug 30 21:33:03 2009 +0100
</span><span class="line">
</span><span class="line"> Revert <span class="s2">"showing what modifying files that are tracked does"</span>
</span><span class="line">
</span><span class="line"> This reverts commit b9b57e53fa0bdcd29b14fb0ded6ab4355ef3cb92.
</span><span class="line">
</span><span class="line">commit b9b57e53fa0bdcd29b14fb0ded6ab4355ef3cb92
</span><span class="line">Author: Jamie van Dyke &lt;fearoffish@fearofpro.local&gt;
</span><span class="line">Date: Fri Aug 28 23:03:31 2009 +0100
</span><span class="line">
</span><span class="line"> showing what modifying files that are tracked does
</span><span class="line">
</span><span class="line">commit a08539202e8bf908106e1736dabf588c3b367151
</span><span class="line">Author: Jamie van Dyke &lt;fearoffish@fearofpro.local&gt;
</span><span class="line">Date: Fri Aug 28 21:58:47 2009 +0100
</span><span class="line">
</span><span class="line"> initial bare rails project
</span></code></pre></td>
</tr></table></div></figure>

When you run the git revert command, you give it the SHA-1 reference – or six characters, like I did – and git will remove any changes you made for that commit. Cool, huh? So you are keeping multiple versions of your application without keeping multiple files.

Okay, that’s enough for this episode. I’ll be creating more episodes on subjects like local and remote repositories, hosting your Rails applications and more, soon.

