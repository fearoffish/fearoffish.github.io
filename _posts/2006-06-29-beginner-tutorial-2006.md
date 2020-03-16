---
title: Beginner Tutorial (2006)
tags: 
  - ruby-on-rails
categories: 
  - old-archive
excerpt_separator: "<!--more-->"
---

> This is a seriously old post. I'd probably not bother reading it, but I keep it here for nostalgia.

In the spirit of the rails community, I'm beginning a short tutorial series on the overview of [Ruby on Rails](http://www.rubyonrails.org/ "Ruby on Rails"), how it works and where to get started. In Part 1 we're going to look at how it all knits together into a web application. We'll talk about which bits do what, and what to use them for.

When you first create a rails application with

    
    1. rails my\_whizzy\_app

 you are given a structure like so:

![File Structure](http://www.fearoffish.co.uk/files/file_structure.png "File Structure of a Rails project")

But, jebus! When I create a php applicaiton it's empty, what the heck is all that? Well, let's drill through each directory in this part of our beginners series and check out the most important ones.

![config directory](http://www.fearoffish.co.uk/files/config_dir.png "config directory")

The first place you're going to find yourself after creating your spiffy new application is the

    
    1. config

 directory. 
- _boot.rb_ - You don't need to look in here until much later (if ever) in your learnings...step away slowly, and avert your eyes.
- _database.yml_ - This file is the key to your database, it tells rails what type, the location, and the names and passwords needed to get at your database.
- _environment.rb_ - Application wide settings can be placed in here, the most common reason for editing this file is if you don't have control over your web server and need to switch from development mode to production mode manually, or to put ActionMailer settings in to tell rails how to send mail.
- _routes.rb_ - This file lets you control what your url's look like. No need to mess around with scary mod\_rewrite directives, just take a look in here and marvel at the wondrous ingenuity (keep your eye out for one here later).

![app directory](http://www.fearoffish.co.uk/files/app_dir.png "app directory")

If you're not familiar with the object-orientated term MVC you'll need to do a little research first, a good start would be the [wikipedia](:http://www.wikipedia.org/ "Wikipedia") article on [model-view-controller](http://en.wikipedia.org/wiki/Model-view-controller "Model View Controller architecture") architecture.

Put simply, the

    
    1. apps

 folder contains the intelligent parts of your application.
- _The models folder_ - The classes that control your data will be placed here. For example, if your application deals with [chunky bacon](http://poignantguide.net/ruby/ "Whys Poignant Guide to Ruby"), then your ChunkyBacon model will be in here, you would use this to interact with your database of piggery pieces.
- _The views folder_ - You certainly want the world to view your bacon bits, hence a necessity for this folder. In terms that might be easier to understand, this is where your web templates go, the pieces of html and view logic.
- _The controllers folder_ - Melding this all together we have our controllers, which control what's going in and out of your models, and what's being displayed to the user.

![public dir](http://www.fearoffish.co.uk/files/public_dir.png "public directory")

Essentially this folder is where you place anything that you want the public to be able to access. Rails automatically creates a

    
    1. stylesheets

, 

    
    1. images

 and 

    
    1. javascripts

 folder, and I'll leave it to your imagination what you will be placing in there...it's not your dirty laundry, that's for sure. If you take a look at the screenshot of my folder structure, you can see a miriad of files in there. Let's have a quick run through theM:
- _404.html, 500.html_ - We all hate to admit it, but errors can occur, and we can't catch them all. These files will be used by rails to handle [http status error codes](http://www.ilovejackdaniels.com/apache/http-status-codes-explained/ "http status error codes").
- _dispatch files_ - Depending how you set up your web server, one of these files will be used to take browser requests and forward them to your controllers to be handled. Most rails setup problems arise from these files, so be careful when editing them (If you desperately need to) because it could be the cause of a serious hair loss in future.
- _index.html_ - This is the default page you see when you first browse to your rails application, you'll need to delete this when you adjust your routes in future, it shortcuts the process of passing control to rails. What that means is, even if you set up your routes.rb to automatically direct you to an index action, if there is an index.html file here, your app will never even get a look in sideways, so bear this in mind.

### Looking to host a Ruby on Rails site?

Dreamhost offers cheap Rails hosting packages. Signup now and enter the promotion code **fearoffish** to receive up to $47 off your first year of hosting. Your total cost for one year will be $50 - that's only about $4 per month! You get a Terabyte of bandwidth and a huge amount of diskspace, and you support my bandwidth costs here too!

[Signup Now!](http://www.dreamhost.com/r.cgi?165954)

![test dir](http://www.fearoffish.co.uk/files/test_dir.png "test dir")!

You're going to be spending a lot of time in this directory, because Rails encourages [Test Driven Development](http://en.wikipedia.org/wiki/Test_driven_development "Test Driven Development"). It is possible to never set eyes on the testing side, but seriously, who wants to write an application that 'might' work. It's not about trusting your coding, or questioning your abilities. It's all about guaranteeing to the best of your ability, that the code you've written functions correctly.

Rails gives you access to different types of tests, categorised into the directories yousee in the image above:

- _Fixtures_ - Your tests need data too! Fixtures are your way of having data available for all the tests you run, without having to re-enter it manually every time. They're entered in [YAML](http://en.wikipedia.org/wiki/YAML "YAML Format") format, and allow sprinklings of ruby magic. Which basically means you can generate your data without having to enter it all manually.

- _Functional tests_ - Testing your controllers. At this level we're looking at details like "Did we pass authentication?" or "Has my username been stored in the session file?". It will put a smile on your face watching your little test stories run along ticking all the boxes, think of the sleepless nights you'll no longer have to have!

- _Integration tests_ - This is a newer folder that was introduced in Rails 1.1. Integration tests give you the ability to mimic the users path through your website, which opens up new possibilities for your test cases. It's not just all down to QA now! Imagine a user reporting a bug in your application, but to replicate it you have to visit 8 pages in a specific order, and do specific actions. Can you imagine the pain of testing this as you try and debug it? Integration tests...feel the joy.

- _Mock tests_ - Although this is a little advanced, what you need to know is , as an example, if you're going to be using network code, and don't always have a network available, or maybe you use a service over the internet but don't always have access to it, you can use a mock to simulate the other end, or the part of your test that isn't always available or maybe isn't ready yet.

- 

_Unit tests_ - Model testing, that's what this folder is all about. Test the business logic in your model classes e.g. Does your

    
      1. chunk\_bacon

 model return the correct odour and texture when fried?

This simple look into a default rails folder structure will hopefully help you to understand the bits and bobs you first see when you create your rails application. In Part 2 we'll be looking at creating a simple messaging system. Stay tuned!


> Seriously old posts had seriously old comments. More nostalgia!

## Muyiwa wrote:

Great tutorial. When is the 2nd part going to be live?

## Fear of Fish wrote:

The minute I get time, I assure you. I jsut have a few projects to complete first.

## Abraham wrote:

Thanks for this... I have been wanting to get into Ruby on Rails casually for some time now. Although I'm already a Ruby user and Ruby fan, and although RoR tutorials are supposed to be so easy, for some reason I could not get into them. Your tutorial seems a bit more accessible... maybe different people learn in different ways. I like the approach of discussing the overall directory structure.

## Andrea wrote:

The moment you do, would you add to this page and update an index to the existing 'episodes'? It would be really helpful to be able to bookmark the start of the series knowing that you can always each the rest from there. Since this is the single best chance I have to dip my finger in ruby waters, I'm already bookmarking this in anticipation! Thks.

## Fear of Fish wrote:

What browser are you using? I've changed a few bits and bobs here and there and haven't had the chance to test on all of them yet.

## Andrea wrote:

Firefox 1.5.0.4/WinXP

## Andrea wrote:

... and looks like everything's ok now. Cheers.
