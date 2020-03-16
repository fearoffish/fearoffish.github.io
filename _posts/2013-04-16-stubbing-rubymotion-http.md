---
title: Stubbing Rubymotion Http
tags: 
  - ruby-on-rails
categories: 
  - old-archive
excerpt_separator: "<!--more-->"
---

> This is a seriously old post. I'd probably not bother reading it, but I keep it here for nostalgia.

# Stubbing RubyMotion HTTP

Coming from the Ruby and Rails world, I’m anchored in the paradigm that we should test our code. Our code should not only be tested, but it should be sure not to be hitting external services during those specs. I’ve had a hard time figuring out the best way to stub out HTTP calls effectively and easily in RubyMotion. I’m also not the best Objective-C programmer, so porting libraries already built was not on the cards in the timeframe I wanted.

However, I came across the [WebStub](https://github.com/mattgreen/webstub) library by Matt Green in my search for answers, and played around with it over the weekend. It’s fantastic. Coupled with Clay Allsop’s [AFMotion](https://github.com/clayallsopp/afmotion) library that creates a ruby bridge to the [AFNetworking](https://github.com/AFNetworking/AFNetworking) library.

Let’s run through how I went about testing a little API driven app I’m writing. I assume you have RubyMotion, and that you’re not new to using it, but you’re unfamiliar with how to test HTTP calls. It uses BubbleWraps HTTP calls, not AFMotion, however it’s the same testing method.

## Create a RubyMotion Project

We’re going to clone the brilliant [API Driven Example](http://rubymotion-tutorial.com/10-api-driven-example/) written by Clay Allsop. Did I mention he wrote the [RubyMotion Book](http://pragprog.com/book/carubym/rubymotion)? It’s good. Buy it. The API Driven example on the RubyMotion Tutorial is ripe for a little sprinkle of test. So go ahead and run through that example if you want to play along with me adding specs to it.

Once you have that, we’ll start adding our specs. I’ve cloned the [git repository](https://github.com/clayallsopp/rubymotion-tutorial/tree/master/10-api-driven-example/Colr) that Clay has kindly placed this in, and taken the Colr application straight out of it.

In the Color class, we have 2 HTTP calls, one is a GET and the other is a POST. Let’s wrap a test around that GET request first. The `find` method looks like this:

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
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="k">def</span> <span class="nc">self</span><span class="o">.</span><span class="nf">find</span><span class="p">(</span><span class="n">hex</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">block</span><span class="p">)</span>
</span><span class="line"> <span class="no">BW</span><span class="o">::</span><span class="no">HTTP</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s2">"http://www.colr.org/json/color/</span><span class="si">#{</span><span class="n">hex</span><span class="si">}</span><span class="s2">"</span><span class="p">)</span> <span class="k">do</span> <span class="o">|</span><span class="n">response</span><span class="o">|</span>
</span><span class="line"> <span class="n">result_data</span> <span class="o">=</span> <span class="no">BW</span><span class="o">::</span><span class="no">JSON</span><span class="o">.</span><span class="n">parse</span><span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="n">body</span><span class="o">.</span><span class="n">to_str</span><span class="p">)</span>
</span><span class="line"> <span class="n">color_data</span> <span class="o">=</span> <span class="n">result_data</span><span class="o">[</span><span class="s2">"colors"</span><span class="o">][</span><span class="mi">0</span><span class="o">]</span>
</span><span class="line">
</span><span class="line"> <span class="c1"># Colr will return a color with id == -1 if no color was found</span>
</span><span class="line"> <span class="n">color</span> <span class="o">=</span> <span class="no">Color</span><span class="o">.</span><span class="n">new</span><span class="p">(</span><span class="n">color_data</span><span class="p">)</span>
</span><span class="line"> <span class="k">if</span> <span class="n">color</span><span class="o">.</span><span class="n">id</span><span class="o">.</span><span class="n">to_i</span> <span class="o">==</span> <span class="o">-</span><span class="mi">1</span>
</span><span class="line"> <span class="n">block</span><span class="o">.</span><span class="n">call</span><span class="p">(</span><span class="kp">nil</span><span class="p">)</span>
</span><span class="line"> <span class="k">else</span>
</span><span class="line"> <span class="n">block</span><span class="o">.</span><span class="n">call</span><span class="p">(</span><span class="n">color</span><span class="p">)</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line"><span class="k">end</span>
</span></code></pre></td>
</tr></table></div></figure>

Add [WebStub](https://github.com/mattgreen/webstub) to your Rakefile.

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
<td class="code"><pre><code class="ruby"><span class="line"><span class="c1"># -*- coding: utf-8 -*-</span>
</span><span class="line"><span class="vg">$:</span><span class="o">.</span><span class="n">unshift</span><span class="p">(</span><span class="s2">"/Library/RubyMotion/lib"</span><span class="p">)</span>
</span><span class="line"><span class="nb">require</span> <span class="s1">'motion/project'</span>
</span><span class="line"><span class="nb">require</span> <span class="s1">'bubble-wrap'</span>
</span><span class="line"><span class="nb">require</span> <span class="s1">'webstub'</span>
</span><span class="line">
</span><span class="line"><span class="no">Motion</span><span class="o">::</span><span class="no">Project</span><span class="o">::</span><span class="no">App</span><span class="o">.</span><span class="n">setup</span> <span class="k">do</span> <span class="o">|</span><span class="n">app</span><span class="o">|</span>
</span><span class="line"> <span class="c1"># Use `rake config' to see complete project settings.</span>
</span><span class="line"> <span class="n">app</span><span class="o">.</span><span class="n">name</span> <span class="o">=</span> <span class="s1">'Colr'</span>
</span><span class="line"><span class="k">end</span>
</span></code></pre></td>
</tr></table></div></figure>

Here’s the `color_spec.rb` spec I built to demonstrate testing that call.

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
<span class="line-number">49</span>
<span class="line-number">50</span>
<span class="line-number">51</span>
<span class="line-number">52</span>
<span class="line-number">53</span>
<span class="line-number">54</span>
<span class="line-number">55</span>
<span class="line-number">56</span>
<span class="line-number">57</span>
<span class="line-number">58</span>
<span class="line-number">59</span>
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="n">describe</span> <span class="no">Color</span> <span class="k">do</span>
</span><span class="line"> <span class="kp">extend</span> <span class="no">WebStub</span><span class="o">::</span><span class="no">SpecHelpers</span>
</span><span class="line">
</span><span class="line"> <span class="n">before</span> <span class="k">do</span>
</span><span class="line"> <span class="n">disable_network_access!</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line">
</span><span class="line"> <span class="n">describe</span> <span class="s2">"#find"</span> <span class="k">do</span>
</span><span class="line"> <span class="n">before</span> <span class="k">do</span>
</span><span class="line"> <span class="vi">@successful_color_response</span> <span class="o">=</span> <span class="s1">'{"colors": [{"timestamp": 1348628320, "hex": "ffba13", "id": 5490, "tags": [{"timestamp": 1129240205, "id": 14386, "name": "cheesy"}, {"timestamp": 1108442341, "id": 5076, "name": "fries"}, {"timestamp": 1108442340, "id": 5075, "name": "cheese"}, {"timestamp": 1108442341, "id": 5076, "name": "fries"}, {"timestamp": 1344743165, "id": 26414, "name": "yellowsnow"}, {"timestamp": 1109734601, "id": 6414, "name": "bee"}, {"timestamp": 1108110850, "id": 2542, "name": "yellow"}, {"timestamp": 1346850231, "id": 26442, "name": "kahit"}, {"timestamp": 1120839389, "id": 13877, "name": "ano"}, {"timestamp": 1348492735, "id": 26461, "name": "uniqum"}, {"timestamp": 1348627829, "id": 26463, "name": "somecolor"}, {"timestamp": 1348627829, "id": 26463, "name": "somecolor"}, {"timestamp": 1348627829, "id": 26463, "name": "somecolor"}, {"timestamp": 1348627829, "id": 26463, "name": "somecolor"}]}], "schemes": [], "schemes_history": {}, "success": true, "colors_history": {"ffba13": [{"d_count": 3, "id": "5075", "a_count": 5, "name": "cheese"}, {"d_count": 2, "id": "5076", "a_count": 5, "name": "fries"}, {"d_count": 1, "id": "5077", "a_count": 2, "name": "ham"}, {"d_count": 2, "id": "5078", "a_count": 2, "name": "spazz"}, {"d_count": 1, "id": "13840", "a_count": 1, "name": "01oct05"}, {"d_count": 1, "id": "14849", "a_count": 1, "name": "998"}, {"d_count": 1, "id": "3336", "a_count": 1, "name": "love"}, {"d_count": 0, "id": "14386", "a_count": 1, "name": "cheesy"}, {"d_count": 0, "id": "26414", "a_count": 1, "name": "yellowsnow"}, {"d_count": 0, "id": "6414", "a_count": 1, "name": "bee"}, {"d_count": 1, "id": "2542", "a_count": 2, "name": "yellow"}, {"d_count": 0, "id": "26442", "a_count": 1, "name": "kahit"}, {"d_count": 0, "id": "13877", "a_count": 1, "name": "ano"}, {"d_count": 0, "id": "26461", "a_count": 1, "name": "uniqum"}, {"d_count": 0, "id": "26463", "a_count": 4, "name": "somecolor"}]}, "messages": [], "new_color": "ffba13"}'</span>
</span><span class="line"> <span class="vi">@unsuccessful_color_response</span> <span class="o">=</span> <span class="s1">'{"colors": [{"timestamp": 1356329465, "hex": "ffba1", "id": -1, "tags": []}], "schemes": [], "schemes_history": {}, "success": true, "colors_history": {}, "messages": [], "new_color": "ffba1"}'</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line">
</span><span class="line"> <span class="n">it</span> <span class="s2">"converts a json color into a Color object"</span> <span class="k">do</span>
</span><span class="line"> <span class="n">stub_request</span><span class="p">(</span><span class="ss">:get</span><span class="p">,</span> <span class="s2">"http://www.colr.org/json/color/ffba13"</span><span class="p">)</span><span class="o">.</span>
</span><span class="line"> <span class="n">to_return</span><span class="p">(</span><span class="n">body</span><span class="p">:</span> <span class="vi">@successful_color_response</span><span class="p">,</span> <span class="n">content_type</span><span class="p">:</span> <span class="s2">"application/json"</span><span class="p">)</span>
</span><span class="line">
</span><span class="line"> <span class="vi">@color</span> <span class="o">=</span> <span class="kp">nil</span>
</span><span class="line"> <span class="no">Color</span><span class="o">.</span><span class="n">find</span><span class="p">(</span><span class="s1">'ffba13'</span><span class="p">)</span> <span class="k">do</span> <span class="o">|</span><span class="n">color</span><span class="o">|</span>
</span><span class="line"> <span class="vi">@color</span> <span class="o">=</span> <span class="n">color</span>
</span><span class="line"> <span class="n">resume</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line">
</span><span class="line"> <span class="n">wait_max</span> <span class="mi">1</span><span class="o">.</span><span class="mi">0</span> <span class="k">do</span>
</span><span class="line"> <span class="vi">@color</span><span class="o">.</span><span class="n">class</span><span class="o">.</span><span class="n">should</span> <span class="o">==</span> <span class="no">Color</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line">
</span><span class="line"> <span class="n">it</span> <span class="s2">"returns a nil when no color is found"</span> <span class="k">do</span>
</span><span class="line"> <span class="n">stub_request</span><span class="p">(</span><span class="ss">:get</span><span class="p">,</span> <span class="s2">"http://www.colr.org/json/color/nothing"</span><span class="p">)</span><span class="o">.</span>
</span><span class="line"> <span class="n">to_return</span><span class="p">(</span><span class="n">body</span><span class="p">:</span> <span class="vi">@unsuccessful_color_response</span><span class="p">,</span> <span class="n">content_type</span><span class="p">:</span> <span class="s2">"application/json"</span><span class="p">)</span>
</span><span class="line">
</span><span class="line"> <span class="vi">@color</span> <span class="o">=</span> <span class="kp">nil</span>
</span><span class="line"> <span class="no">Color</span><span class="o">.</span><span class="n">find</span><span class="p">(</span><span class="s1">'nothing'</span><span class="p">)</span> <span class="k">do</span> <span class="o">|</span><span class="n">color</span><span class="o">|</span>
</span><span class="line"> <span class="vi">@color</span> <span class="o">=</span> <span class="n">color</span>
</span><span class="line"> <span class="n">resume</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line">
</span><span class="line"> <span class="n">wait_max</span> <span class="mi">1</span><span class="o">.</span><span class="mi">0</span> <span class="k">do</span>
</span><span class="line"> <span class="vi">@color</span><span class="o">.</span><span class="n">class</span><span class="o">.</span><span class="n">should</span> <span class="o">==</span> <span class="no">NilClass</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line">
</span><span class="line"> <span class="n">it</span> <span class="s2">"tells the Colr api to add a tag to the color"</span> <span class="k">do</span>
</span><span class="line"> <span class="n">stub_request</span><span class="p">(</span><span class="ss">:post</span><span class="p">,</span> <span class="s2">"http://www.colr.org/js/color/ffba13/addtag/"</span><span class="p">)</span><span class="o">.</span>
</span><span class="line"> <span class="n">to_return</span><span class="p">(</span><span class="n">body</span><span class="p">:</span> <span class="vi">@successful_color_response</span><span class="p">,</span> <span class="n">content_type</span><span class="p">:</span> <span class="s2">"application/json"</span><span class="p">)</span>
</span><span class="line">
</span><span class="line"> <span class="vi">@color</span> <span class="o">=</span> <span class="no">Color</span><span class="o">.</span><span class="n">new</span><span class="p">(</span><span class="n">hex</span><span class="p">:</span> <span class="s1">'ffba13'</span><span class="p">)</span>
</span><span class="line"> <span class="vi">@color</span><span class="o">.</span><span class="n">add_tag</span><span class="p">(</span><span class="s1">'mustard'</span><span class="p">)</span> <span class="k">do</span> <span class="o">|</span><span class="n">color</span><span class="o">|</span>
</span><span class="line"> <span class="vi">@color</span> <span class="o">=</span> <span class="n">color</span>
</span><span class="line"> <span class="n">resume</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line">
</span><span class="line"> <span class="n">wait_max</span> <span class="mi">1</span><span class="o">.</span><span class="mi">0</span> <span class="k">do</span>
</span><span class="line"> <span class="vi">@color</span><span class="o">.</span><span class="n">class</span><span class="o">.</span><span class="n">should</span> <span class="o">==</span> <span class="no">NilClass</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line"> <span class="k">end</span>
</span><span class="line"><span class="k">end</span>
</span></code></pre></td>
</tr></table></div></figure>
## Things to Note

First, we need to engage the WebStub library, and disable any network access. WebStub will let us know of any unauthorised networked access and block it, perfect for debugging url’s.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="kp">extend</span> <span class="no">WebStub</span><span class="o">::</span><span class="no">SpecHelpers</span>
</span><span class="line">
</span><span class="line"><span class="n">before</span> <span class="k">do</span>
</span><span class="line"> <span class="n">disable_network_access!</span>
</span><span class="line"><span class="k">end</span>
</span></code></pre></td>
</tr></table></div></figure>

Set up the response that WebStub should give you if you call the stubbed URL. This is better done from fixtures, but for verbosity I’m putting it here for now.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="vi">@successful_color_response</span> <span class="o">=</span> <span class="s1">'{"colors": [{"timestamp": 1348628320...olor": "ffba13"}'</span>
</span></code></pre></td>
</tr></table></div></figure>

Now, set up WebStub to listen on the URL that we’re calling for this test.

<figure class="code"><figcaption><span></span></figcaption><div class="highlight"><table><tr>
<td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td>
<td class="code"><pre><code class="ruby"><span class="line"><span class="n">stub_request</span><span class="p">(</span><span class="ss">:get</span><span class="p">,</span> <span class="s2">"http://www.colr.org/json/color/ffba13"</span><span class="p">)</span><span class="o">.</span>
</span><span class="line"> <span class="n">to_return</span><span class="p">(</span><span class="n">body</span><span class="p">:</span> <span class="vi">@successful_color_response</span><span class="p">,</span> <span class="n">content_type</span><span class="p">:</span> <span class="s2">"application/json"</span><span class="p">)</span>
</span></code></pre></td>
</tr></table></div></figure>

Here’s the fun part. HTTP calls are done in a separate thread from the main thread, so catching when they’re done requires a little ‘hooking’. We put the `resume` method call in the block, my understanding is that this is like calling `thread.join` in Ruby. We want to wait for it to return in our test thread, not go running off all by itself and returning results to the abyss.

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
<td class="code"><pre><code class="ruby"><span class="line"><span class="c1"># Local variables can go out of scope before a block finishes in RubyMotion, so we use an instance variable that won't be garbage collected.</span>
</span><span class="line"><span class="vi">@color</span> <span class="o">=</span> <span class="kp">nil</span>
</span><span class="line"><span class="c1"># Call our Color#find method and set the instance variable to the response</span>
</span><span class="line"><span class="no">Color</span><span class="o">.</span><span class="n">find</span><span class="p">(</span><span class="s1">'nothing'</span><span class="p">)</span> <span class="k">do</span> <span class="o">|</span><span class="n">color</span><span class="o">|</span>
</span><span class="line"> <span class="vi">@color</span> <span class="o">=</span> <span class="n">color</span>
</span><span class="line"> <span class="n">resume</span>
</span><span class="line"><span class="k">end</span>
</span><span class="line">
</span><span class="line"><span class="c1"># Now we need to wait for the thread to join (this is to go with the resume above)</span>
</span><span class="line"><span class="n">wait_max</span> <span class="mi">1</span><span class="o">.</span><span class="mi">0</span> <span class="k">do</span>
</span><span class="line"> <span class="c1"># Normal test patterns resume here</span>
</span><span class="line"> <span class="vi">@color</span><span class="o">.</span><span class="n">class</span><span class="o">.</span><span class="n">should</span> <span class="o">==</span> <span class="no">NilClass</span>
</span><span class="line"><span class="k">end</span>
</span></code></pre></td>
</tr></table></div></figure>

That’s all there is to it. If you’re using [AFMotion](https://github.com/clayallsopp/afmotion) like me, your specs will look just the same. The only difference being I usually abstract the service layer to it’s own module/class so I can switch it out later if I have issues.

I hope that helped.

