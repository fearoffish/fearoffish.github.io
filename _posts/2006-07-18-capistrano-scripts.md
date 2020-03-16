---
title: Capistrano Scripts
tags: 
  - ruby-on-rails
categories: 
  - old-archive
excerpt_separator: "<!--more-->"
---

> This is a seriously old post. I'd probably not bother reading it, but I keep it here for nostalgia.

Here's a quick note for those using DreamHost and Capistrano (a.k.a SwitchTower) to

    
    1. rake deploy

 your sites to production. You no doubt use this restart command:

    
    
        
        1. killall -9 dispatch.fcgi
    
    

It no longer works for me, and I have had to change all my deploy scripts to instead run:

    
    
        
        1. killall -9 ruby1.8
    
    

Be aware of this if your

    
    1. rake deploy

 commands are no longer forcing an update. 