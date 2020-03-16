---
title: The Inherited Resources Railties Error
tags: 
  - ruby-on-rails
categories: 
  - old-archive
excerpt_separator: "<!--more-->"
---

> This is a seriously old post. I'd probably not bother reading it, but I keep it here for nostalgia.

# Railties Error?

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

