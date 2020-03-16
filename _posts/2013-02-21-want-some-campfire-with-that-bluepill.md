---
title: Want Some Campfire With That Bluepill
tags: 
  - ruby-on-rails
categories: 
  - old-archive
excerpt_separator: "<!--more-->"
---

> This is a seriously old post. I'd probably not bother reading it, but I keep it here for nostalgia.

# Custom Triggers With the Bluepill Monitoring Gem

We use [Chef](http://wiki.opscode.com/) for automating our server infrastructure, but I’ll show you the manual way of installing [Bluepill](https://github.com/arya/bluepill) (either locally or remotely) so you can play with it.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line">gem install bluepill
</span></code></pre></td>
</tr></table></div></figure>

That wasn’t hard, was it? Dependencies will be resolved, and installed too. Now we’ve got it installed, let’s take a little look at the commands we can give it.

Check the status of all applications and processes

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line">sudo bluepill status
</span></code></pre></td>
</tr></table></div></figure>

Start all applications and stop all applications, or a single group.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line">sudo bluepill start
</span><span class="line">sudo bluepill stop
</span><span class="line">sudo bluepill start resque
</span></code></pre></td>
</tr></table></div></figure>

Quit Bluepill.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line"><span class="nv">$ </span>sudo bluepill quit
</span></code></pre></td>
</tr></table></div></figure>

You configure Bluepill with a configuration file. Here’s one we use to start multiple [resque](https://github.com/defunkt/resque) queues. It’s a work in progress but does what we want for now. I won’t explain the config file because there’s plenty of blogs (and of course the Bluepill README) that go into detail here.

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
<span class="line-number">28</span>
<span class="line-number">29</span>
<span class="line-number">30</span>
<span class="line-number">31</span>
<span class="line-number">32</span>
<span class="line-number">33</span>
<span class="line-number">34</span>
<span class="line-number">35</span>
<span class="line-number">36</span>
<span class="line-number">37</span>
<span class="line-number">38</span>
<span class="line-number">39</span>
<span class="line-number">40</span>
<span class="line-number">41</span>
<span class="line-number">42</span>
<span class="line-number">43</span>
<span class="line-number">44</span>
<span class="line-number">45</span>
<span class="line-number">46</span>
<span class="line-number">47</span>
<span class="line-number">48</span>
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="n">queues</span> <span class="o">=</span> <span class="p">{}</span>
</span><span class="line"><span class="n">queues</span><span class="o">[</span><span class="s2">"comms"</span><span class="o">]</span> <span class="o">=</span> <span class="mi">1</span>
</span><span class="line"><span class="n">queues</span><span class="o">[</span><span class="s2">"rules"</span><span class="o">]</span> <span class="o">=</span> <span class="mi">2</span>
</span><span class="line"><span class="n">queues</span><span class="o">[</span><span class="s2">"patients"</span><span class="o">]</span> <span class="o">=</span> <span class="mi">4</span>
</span><span class="line"><span class="n">queues</span><span class="o">[</span><span class="s2">"import"</span><span class="o">]</span> <span class="o">=</span> <span class="mi">1</span>
</span><span class="line">
</span><span class="line"><span class="vi">@app_name</span> <span class="o">=</span> <span class="s2">"pharmmd"</span>
</span><span class="line"><span class="vi">@owner_name</span> <span class="o">=</span> <span class="s2">"rails"</span>
</span><span class="line"><span class="vi">@rails_env</span> <span class="o">=</span> <span class="s2">"production"</span>
</span><span class="line"><span class="vi">@rake_command</span> <span class="o">=</span> <span class="s2">"bundle exec rake "</span>
</span><span class="line">
</span><span class="line"><span class="no">Bluepill</span><span class="o">.</span><span class="n">application</span><span class="p">(</span><span class="s2">"</span><span class="si">#{</span><span class="vi">@app_name</span><span class="si">}</span><span class="s2">_resque"</span><span class="p">,</span> <span class="ss">:log_file</span> <span class="o">=&gt;</span> <span class="s2">"/var/log/bluepill.log"</span><span class="p">)</span> <span class="k">do</span> <span class="o">|</span><span class="n">app</span><span class="o">|</span>
</span><span class="line"> <span class="n">app</span><span class="o">.</span><span class="n">uid</span> <span class="o">=</span> <span class="n">app</span><span class="o">.</span><span class="n">gid</span> <span class="o">=</span> <span class="vi">@owner_name</span>
</span><span class="line">
</span><span class="line"> <span class="n">queues</span><span class="o">.</span><span class="n">each</span> <span class="k">do</span> <span class="o">|</span><span class="n">queue</span><span class="p">,</span> <span class="n">count</span><span class="o">|</span>
</span><span class="line"> <span class="n">count</span><span class="o">.</span><span class="n">times</span><span class="o">.</span><span class="n">each</span> <span class="k">do</span> <span class="o">|</span><span class="n">idx</span><span class="o">|</span>
</span><span class="line"> <span class="n">app</span><span class="o">.</span><span class="n">process</span><span class="p">(</span><span class="s2">"resque_worker_</span><span class="si">#{</span><span class="n">queue</span><span class="si">}</span><span class="s2">_</span><span class="si">#{</span><span class="n">idx</span><span class="si">}</span><span class="s2">"</span><span class="p">)</span> <span class="k">do</span> <span class="o">|</span><span class="n">process</span><span class="o">|</span>
</span><span class="line"> <span class="n">process</span><span class="o">.</span><span class="n">group</span> <span class="o">=</span> <span class="s1">'resque'</span>
</span><span class="line"> <span class="n">process</span><span class="o">.</span><span class="n">pid_file</span> <span class="o">=</span> <span class="s2">"/var/run/resque/</span><span class="si">#{</span><span class="vi">@app_name</span><span class="si">}</span><span class="s2">/worker_</span><span class="si">#{</span><span class="n">queue</span><span class="si">}</span><span class="s2">_</span><span class="si">#{</span><span class="n">idx</span><span class="si">}</span><span class="s2">.pid"</span>
</span><span class="line"> <span class="n">process</span><span class="o">.</span><span class="n">working_dir</span> <span class="o">=</span> <span class="s2">"/data/</span><span class="si">#{</span><span class="vi">@app_name</span><span class="si">}</span><span class="s2">/current"</span>
</span><span class="line"> <span class="n">process</span><span class="o">.</span><span class="n">start_command</span> <span class="o">=</span> <span class="vi">@rake_command</span> <span class="o">+</span>
</span><span class="line"> <span class="s2">"RACK_ENV=</span><span class="si">#{</span><span class="vi">@rails_env</span><span class="si">}</span><span class="s2"> RAILS_ENV=</span><span class="si">#{</span><span class="vi">@rails_env</span><span class="si">}</span><span class="s2"> "</span> <span class="o">+</span>
</span><span class="line"> <span class="s2">"QUEUE=</span><span class="si">#{</span><span class="n">queue</span><span class="si">}</span><span class="s2"> "</span> <span class="o">+</span>
</span><span class="line"> <span class="s2">"resque:work"</span>
</span><span class="line"> <span class="n">process</span><span class="o">.</span><span class="n">stop_command</span> <span class="o">=</span> <span class="o">&lt;&lt;-</span><span class="no">EOF</span>
</span><span class="line"><span class="sh"> kill -QUIT </span>
</span><span class="line"><span class="sh"> sleep_count=0</span>
</span><span class="line"><span class="sh"> while [-e /proc/]; do</span>
</span><span class="line"><span class="sh"> sleep 1</span>
</span><span class="line"><span class="sh"> let "sleep_count+=1"</span>
</span><span class="line"><span class="sh"> if [$sleep_count -eq 60]; then</span>
</span><span class="line"><span class="sh"> kill -TERM </span>
</span><span class="line"><span class="sh"> fi</span>
</span><span class="line"><span class="sh"> if [$sleep_count -ge 70]; then</span>
</span><span class="line"><span class="sh"> kill -KILL </span>
</span><span class="line"><span class="sh"> fi</span>
</span><span class="line"><span class="sh"> done</span>
</span><span class="line"><span class="no"> EOF</span>
</span><span class="line"> <span class="n">process</span><span class="o">.</span><span class="n">start_grace_time</span> <span class="o">=</span> <span class="mi">5</span><span class="o">.</span><span class="n">seconds</span>
</span><span class="line"> <span class="n">process</span><span class="o">.</span><span class="n">stop_grace_time</span> <span class="o">=</span> <span class="mi">75</span><span class="o">.</span><span class="n">seconds</span>
</span><span class="line"> <span class="n">process</span><span class="o">.</span><span class="n">restart_grace_time</span> <span class="o">=</span> <span class="mi">80</span><span class="o">.</span><span class="n">seconds</span>
</span><span class="line"> <span class="n">process</span><span class="o">.</span><span class="n">daemonize</span> <span class="o">=</span> <span class="kp">true</span>
</span><span class="line">
</span><span class="line"> <span class="n">process</span><span class="o">.</span><span class="n">checks</span> <span class="ss">:mem_usage</span><span class="p">,</span> <span class="ss">:below</span> <span class="o">=&gt;</span> <span class="mi">150</span><span class="o">.</span><span class="n">megabytes</span><span class="p">,</span> <span class="ss">:every</span> <span class="o">=&gt;</span> <span class="mi">1</span><span class="o">.</span><span class="n">minute</span><span class="p">,</span> <span class="ss">:times</span> <span class="o">=&gt;</span> <span class="mi">3</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line"><span class="k">end</span>
</span></code></pre></td>
</tr></table></div></figure>

What I wanted to add was a notification system that let me know when things had gone horribly wrong. Specifically I wanted [Campfire](http://campfirenow.com/) notifications, as that’s where all the cool techies at [PharmMD](http://www.pharmmd.com/) hang out.

For notifications, [Bluepill](https://github.com/arya/bluepill) allows us to write simple ‘Triggers’. These triggers can be added to a process in the config file, like zoh:

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="n">process</span><span class="o">.</span><span class="n">checks</span> <span class="ss">:some_trigger</span><span class="p">,</span> <span class="ss">:every</span> <span class="o">=&gt;</span> <span class="mi">1</span><span class="o">.</span><span class="n">minute</span>
</span></code></pre></td>
</tr></table></div></figure>

So let’s add a [Campfire](http://campfirenow.com/) notifier. First things first, I’ll be using the [tinder](https://github.com/collectiveidea/tinder) gem to talk to Campfire, have a look at their README for details on how it works (or the excellent [tinder homepage](http://tinder.rubyforge.org/)). Install tinder.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td>
<td class="code"><pre><code class="bash"><span class="line">gem install tinder
</span></code></pre></td>
</tr></table></div></figure>

At the top of the [Bluepill](https://github.com/arya/bluepill) config file (anywhere above the application block) add the following:

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
<span class="line-number">28</span>
<span class="line-number">29</span>
<span class="line-number">30</span>
<span class="line-number">31</span>
<span class="line-number">32</span>
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="nb">require</span> <span class="s1">'rubygems'</span>
</span><span class="line"><span class="nb">require</span> <span class="s1">'uri'</span>
</span><span class="line"><span class="nb">require</span> <span class="s1">'tinder'</span>
</span><span class="line">
</span><span class="line"><span class="k">class</span> <span class="nc">Campfire</span> <span class="o">&lt;</span> <span class="no">Bluepill</span><span class="o">::</span><span class="no">Trigger</span>
</span><span class="line"> <span class="no">PARAMS</span> <span class="o">=</span> <span class="o">[</span><span class="ss">:times</span><span class="p">,</span> <span class="ss">:within</span><span class="p">,</span> <span class="ss">:retry_in</span><span class="p">,</span> <span class="ss">:rails_env</span><span class="o">]</span>
</span><span class="line">
</span><span class="line"> <span class="kp">attr_accessor</span> <span class="o">*</span><span class="no">PARAMS</span>
</span><span class="line">
</span><span class="line"> <span class="k">def</span> <span class="nf">initialize</span><span class="p">(</span><span class="n">process</span><span class="p">,</span> <span class="n">options</span> <span class="o">=</span> <span class="p">{})</span>
</span><span class="line"> <span class="n">options</span><span class="o">.</span><span class="n">reverse_merge!</span><span class="p">(</span><span class="ss">:times</span> <span class="o">=&gt;</span> <span class="mi">5</span><span class="p">,</span> <span class="ss">:within</span> <span class="o">=&gt;</span> <span class="mi">1</span><span class="p">,</span> <span class="ss">:retry_in</span> <span class="o">=&gt;</span> <span class="mi">5</span><span class="p">)</span>
</span><span class="line">
</span><span class="line"> <span class="n">options</span><span class="o">.</span><span class="n">each_pair</span> <span class="k">do</span> <span class="o">|</span><span class="nb">name</span><span class="p">,</span> <span class="n">val</span><span class="o">|</span>
</span><span class="line"> <span class="nb">instance_variable_set</span><span class="p">(</span><span class="s2">"@</span><span class="si">#{</span><span class="nb">name</span><span class="si">}</span><span class="s2">"</span><span class="p">,</span> <span class="n">val</span><span class="p">)</span> <span class="k">if</span> <span class="no">PARAMS</span><span class="o">.</span><span class="n">include?</span><span class="p">(</span><span class="nb">name</span><span class="p">)</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line">
</span><span class="line"> <span class="k">super</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line"> <span class="k">def</span> <span class="nf">notify</span><span class="p">(</span><span class="n">transition</span><span class="p">)</span>
</span><span class="line"> <span class="vi">@logger</span><span class="o">.</span><span class="n">info</span> <span class="s2">"I so just got triggered with a transition!"</span>
</span><span class="line"> <span class="k">begin</span>
</span><span class="line"> <span class="k">if</span> <span class="n">transition</span><span class="o">.</span><span class="n">to</span> <span class="o">==</span> <span class="s2">"down"</span> <span class="o">&amp;&amp;</span> <span class="n">transition</span><span class="o">.</span><span class="n">from</span> <span class="o">==</span> <span class="s2">"up"</span>
</span><span class="line"> <span class="vi">@logger</span><span class="o">.</span><span class="n">info</span> <span class="s2">"BluePill: </span><span class="si">#{</span><span class="vi">@rails_env</span><span class="si">}</span><span class="s2"> worker(</span><span class="si">#{</span><span class="n">process</span><span class="o">.</span><span class="n">name</span><span class="si">}</span><span class="s2">) went from </span><span class="si">#{</span><span class="n">transition</span><span class="o">.</span><span class="n">from_name</span><span class="si">}</span><span class="s2"> to </span><span class="si">#{</span><span class="n">transition</span><span class="o">.</span><span class="n">to_name</span><span class="si">}</span><span class="s2">. Restarting them."</span>
</span><span class="line"> <span class="n">campfire</span> <span class="o">=</span> <span class="no">Tinder</span><span class="o">::</span><span class="no">Campfire</span><span class="o">.</span><span class="n">new</span> <span class="s1">'pharmmd'</span><span class="p">,</span> <span class="ss">:token</span> <span class="o">=&gt;</span> <span class="s1">'TOKEN'</span>
</span><span class="line"> <span class="n">room</span> <span class="o">=</span> <span class="n">campfire</span><span class="o">.</span><span class="n">find_room_by_name</span><span class="p">(</span><span class="s1">'General'</span><span class="p">)</span>
</span><span class="line"> <span class="n">room</span><span class="o">.</span><span class="n">speak</span> <span class="s2">"BluePill: </span><span class="si">#{</span><span class="vi">@rails_env</span><span class="si">}</span><span class="s2"> worker(</span><span class="si">#{</span><span class="n">process</span><span class="o">.</span><span class="n">name</span><span class="si">}</span><span class="s2">) died. Restarting it."</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line"> <span class="k">rescue</span> <span class="o">=&gt;</span> <span class="n">x</span>
</span><span class="line"> <span class="vi">@logger</span><span class="o">.</span><span class="n">info</span> <span class="s2">"Totally failed with: </span><span class="si">#{</span><span class="n">x</span><span class="o">.</span><span class="n">message</span><span class="si">}</span><span class="s2">"</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line"><span class="k">end</span>
</span></code></pre></td>
</tr></table></div></figure>

In the `initialize` method I set up some parameters that I can pass in, as well as provide myself with some defaults to fall back on.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="no">PARAMS</span> <span class="o">=</span> <span class="o">[</span><span class="ss">:times</span><span class="p">,</span> <span class="ss">:within</span><span class="p">,</span> <span class="ss">:retry_in</span><span class="p">,</span> <span class="ss">:rails_env</span><span class="o">]</span>
</span><span class="line"><span class="kp">attr_accessor</span> <span class="o">*</span><span class="no">PARAMS</span>
</span></code></pre></td>
</tr></table></div></figure>

I create instance variables for each option for access later on, and call super to allow the Bluepill::Trigger class to do its own initialisation.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="n">options</span><span class="o">.</span><span class="n">each_pair</span> <span class="k">do</span> <span class="o">|</span><span class="nb">name</span><span class="p">,</span> <span class="n">val</span><span class="o">|</span>
</span><span class="line"> <span class="nb">instance_variable_set</span><span class="p">(</span><span class="s2">"@</span><span class="si">#{</span><span class="nb">name</span><span class="si">}</span><span class="s2">"</span><span class="p">,</span> <span class="n">val</span><span class="p">)</span> <span class="k">if</span> <span class="no">PARAMS</span><span class="o">.</span><span class="n">include?</span><span class="p">(</span><span class="nb">name</span><span class="p">)</span>
</span><span class="line"><span class="k">end</span>
</span></code></pre></td>
</tr></table></div></figure>

Next up I create one of the required methods that Bluepill will call periodically and sprinkle in a little logging (`@logger` is provided by Bluepill::Trigger).

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="vi">@logger</span><span class="o">.</span><span class="n">info</span> <span class="s2">"I so just got triggered with a transition!"</span>
</span></code></pre></td>
</tr></table></div></figure>

Next up we wrap our [Campfire](http://campfirenow.com/) notification in a rescue block, because we don’t want ay silly exceptions like a lack of net access from bombing Bluepill out. There are a few transitions we can use, the important one for me is when a process dies (even though Bluepill will restart it), I want to know when that happens so I can find out why and stop it.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="k">if</span> <span class="n">transition</span><span class="o">.</span><span class="n">to</span> <span class="o">==</span> <span class="s2">"down"</span> <span class="o">&amp;&amp;</span> <span class="n">transition</span><span class="o">.</span><span class="n">from</span> <span class="o">==</span> <span class="s2">"up"</span>
</span></code></pre></td>
</tr></table></div></figure>

Tinder makes campfire access so simple. Connect to campfire, select a room, and then ‘speak’ into it.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="n">campfire</span> <span class="o">=</span> <span class="no">Tinder</span><span class="o">::</span><span class="no">Campfire</span><span class="o">.</span><span class="n">new</span> <span class="s1">'pharmmd'</span><span class="p">,</span> <span class="ss">:token</span> <span class="o">=&gt;</span> <span class="s1">'TOKEN'</span>
</span><span class="line"><span class="n">room</span> <span class="o">=</span> <span class="n">campfire</span><span class="o">.</span><span class="n">find_room_by_name</span><span class="p">(</span><span class="s1">'General'</span><span class="p">)</span>
</span><span class="line"><span class="n">room</span><span class="o">.</span><span class="n">speak</span> <span class="s2">"BluePill: </span><span class="si">#{</span><span class="vi">@rails_env</span><span class="si">}</span><span class="s2"> worker(</span><span class="si">#{</span><span class="n">process</span><span class="o">.</span><span class="n">name</span><span class="si">}</span><span class="s2">) died. Restarting it."</span>
</span></code></pre></td>
</tr></table></div></figure>

That’s it for the trigger. Now all we need to do is tell the process that it should fire that trigger. Add the following to the process block just beneath where we checked for memory usage.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="n">process</span><span class="o">.</span><span class="n">checks</span> <span class="ss">:campfire</span><span class="p">,</span> <span class="ss">:rails_env</span> <span class="o">=&gt;</span> <span class="vi">@rails_env</span>
</span></code></pre></td>
</tr></table></div></figure>

That’s it. Now when you load the config file in, Campfire will get notifications if the process needs restarting.

Simple Campfire notifications using Tinder and Bluepill? Done.

