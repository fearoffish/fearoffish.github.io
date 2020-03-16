---
title: Home Automation and HA Bridge
tags: 
  - home-automation
  - home-assistant
categories: 
  - old-archive
excerpt_separator: "<!--more-->"
---

# Home Assistant and HA Bridge

Today I decided to get [home-assistant](http://home-assistant.io) voice activated with my [Amazon Echo](http://www.amazon.com/Amazon-SK705DI-Echo/dp/B00X4WHP5E). There are multiple ways of doing this, but I chose the easiest and laziest for now.

What does it enable me to do?

> “Alexa, turn on my living room lights”

My living room lights turn on and Alexa says “Okay”.

> “Alexa, turn on my Apple TV”

My Harmony Hub turns on my Samsung TV, Onkyo A/V receiver and Apple TV. The Onkyo changes the HDMI input to the Apple TV. Alexa says “Okay”.

> “Alexa, turn on sexy time”

All the lights in the house turn off, except the bedroom lights which turn red. Yeah, this is a thing. All I need now is for some Barry White to stat playing and I have a real creepy scenario complete! Oh and Alexa says “Okay”.

So let’s take a quick look on the pieces needed to begin. I won’t go into detail on everything you _can_ do, just what you need to know to get going.

#### Requirements

- A computer of some sort, I use a [Raspberry Pi](https://www.raspberrypi.org) running Raspbian.
- An [Amazon Echo](http://www.amazon.com/Amazon-SK705DI-Echo/dp/B00X4WHP5E)
- A working installation of [home-assistant](http://home-assistant.io) on the computer
- A working installation of [ha-bridge](https://github.com/bwssytems/ha-bridge) on the computer

There are many ways to configure this…again, I’m going for simple. You can work out the rest yourself.

#### How does this work?

Here is the secret sauce that makes the magic happen:

- The [Amazon Echo](http://www.amazon.com/Amazon-SK705DI-Echo/dp/B00X4WHP5E) listens to my voice commands
- The [Amazon Echo](http://www.amazon.com/Amazon-SK705DI-Echo/dp/B00X4WHP5E) can turn on, off or dim [Philips Hue](http://www2.meethue.com/en-gb/) lights
- [HA-Bridge](https://github.com/bwssytems/ha-bridge) can pretend to be a [Philips Hue](http://www2.meethue.com/en-gb/) bridge
- [HA-Bridge](https://github.com/bwssytems/ha-bridge) can send instructions to other systems
- [HA-Bridge](https://github.com/bwssytems/ha-bridge) fools the [Amazon Echo](http://www.amazon.com/Amazon-SK705DI-Echo/dp/B00X4WHP5E) into thinking that it is controlling lights when in fact it’s controlling whatever we want

The only downside is that we’re restricted to “turn on”, “turn off” and “dim” commands. But, we’re going with simple so it’ll do for now.

#### Show me

Okay, you want a full example. Let’s take a look at getting a simple light to turn on and off.

In [home-assistant](http://home-assistant.io) I have set up my LimitlessLED (also known as Easybulb) lights.

<figure class="highlight"><pre><code class="language-yaml" data-lang="yaml"><span class="s">platform</span><span class="pi">:</span> <span class="s">limitlessled</span>
<span class="s">bridges</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="s">host</span><span class="pi">:</span> <span class="s">10.0.0.5</span>
    <span class="s">version</span><span class="pi">:</span> <span class="s">5</span>
    <span class="s">port</span><span class="pi">:</span> <span class="s">8899</span>
    <span class="s">groups</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s">number</span><span class="pi">:</span> <span class="s">1</span>
      <span class="s">type</span><span class="pi">:</span> <span class="s">rgbw</span>
      <span class="s">name</span><span class="pi">:</span> <span class="s">Office</span>
    <span class="pi">-</span> <span class="s">number</span><span class="pi">:</span> <span class="s">2</span>
      <span class="s">type</span><span class="pi">:</span> <span class="s">rgbw</span>
      <span class="s">name</span><span class="pi">:</span> <span class="s">Living Room</span>
    <span class="pi">-</span> <span class="s">number</span><span class="pi">:</span> <span class="s">3</span>
      <span class="s">type</span><span class="pi">:</span> <span class="s">rgbw</span>
      <span class="s">name</span><span class="pi">:</span> <span class="s">Kitchen</span>
    <span class="pi">-</span> <span class="s">number</span><span class="pi">:</span> <span class="s">4</span>
      <span class="s">type</span><span class="pi">:</span> <span class="s">rgbw</span>
      <span class="s">name</span><span class="pi">:</span> <span class="s">Bedroom</span></code></pre></figure>

Now when I run [home-assistant](http://home-assistant.io) I can see those in my interface and interact with them by flicking the ‘switch’.

![Lights Panel in Home Assistant](http://share.fearoffish.com/fveZ/Image%202016-04-28%20at%206.45.53%20pm.png)

[Home-assistant](http://home-assistant.io) very kindly exposes all of your devices over a REST API with a nice and simple syntax. Note that I have HTTPS enabled _and_ an API password.

<figure class="highlight"><pre><code class="language-bash" data-lang="bash"><span class="c"># Turn on the lights</span>
curl -X POST -H application/json -H X-HA-Access:API_PASSWORD -d <span class="s1">'{"entity_id":"light.office"}'</span> https://my-pi:8123/api/services/homeassistant/turn_on
<span class="c"># Turn off the lights</span>
curl -X POST -H application/json -H X-HA-Access:API_PASSWORD -d <span class="s1">'{"entity_id":"light.office"}'</span> https://my-pi:8123/api/services/homeassistant/turn_off</code></pre></figure>

Let’s take a look at how to set up [HA-Bridge](https://github.com/bwssytems/ha-bridge) to talk to those lights.

Open up the `Manual Add` tab in the [HA-Bridge](https://github.com/bwssytems/ha-bridge) web interface:

![Manual Add Tab](http://share.fearoffish.com/fwum/Image%202016-04-28%20at%206.52.42%20pm.png)

For more information on the REST API we will call, check out the [HA help documentation](https://home-assistant.io/developers/rest_api/)

Using the example URL’s above we enter in the following details:

==Name==: Whatever you want to say to Alexa, e.g. “Office Lights”

==Device Type==: Custom

== On URL==: `https://my-pi:8123/api/services/homeassistant/turn_on`

== Dim URL==: Blank

== Off URL ==: `https://my-pi:8123/api/services/homeassistant/turn_off`

== HTTP Headers ==: `[{"name":"X-HA-Access", "value":"API_PASSWORD"}]`

==HTTP Verb==: POST

==Content Type==: application/json

==Content Body ON==: `{"entity_id":"light.office"}`

==Content Body OFF==: `{"entity_id":"light.office"}`

Now press “Add Bridge Device” and [HA-Bridge](https://github.com/bwssytems/ha-bridge) will now be ready to let _Alexa_ discover it. Open your _Alexa_ mobile app or open up [The Amazon Echo Smart Home page](http://echo.amazon.com/#smart-home) and _Discover devices_.

Alexa will pick up your newly created bridge device as a [Philips Hue](http://www2.meethue.com/en-gb/) light and make it available for voice activation. I use lights, scripts, scenes and more. As long as I can ‘turn on’ and ‘turn off’ then I’m voice activating it.

Try as I might, I can’t get Alexa to turn on my fiancé though.

<iframe width="560" height="315" src="https://www.youtube.com/embed/UP0ufa9AkuU" frameborder="0" allowfullscreen></iframe>

Boom.

