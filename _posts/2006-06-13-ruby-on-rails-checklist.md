---
title: Ruby on Rails Checklist
tags: 
  - ruby-on-rails
categories: 
  - old-archive
excerpt_separator: "<!--more-->"
---

> This is a seriously old post. I'd probably not bother reading it, but I keep it here for nostalgia.

It seems that plenty of people have problems getting their apps to work on many different hosting packages, so I thought it might be useful to have a checklist for people to check through that cause the most problems. This is not the definitive resource, but in my experience if you get to the end of this list, you're usually up and running.

1. 

The shebang in your

    
      1. dispatch.fcgi

 file must point to ruby. Uploading and overwriting this can catch you out. On Site5 it's #!/usr/bin/ruby. **Update:** The safer option is to ensure **all** dispatch.\* files have the correct shebang, as pointed out by Victor Kane in [this post](http://forums.site5.com/showthread.php?p=58270#post58270 "Site5 Forum"), which should be 

    
      1. #!/usr/bin/env ruby

. This uses the environment variable to select the correct ruby binary.
2. 

Check permissions on your directories. Most importantly make sure that the web server has execute access to

    
      1. dispatch.fcgi

, and write access to the log directory. So set 

    
      1. chmod -R 755

 on public and log.
3. 

Double check your

    
      1. environment.rb

 file, fcgi doesn't enjoy working with a development site. Remove the 

    
      1. #

 on the ENV line.
4. 

Using

    
      1. killall -9 dispatch.fcgi

 is nasty, use 

    
      1. touch -m public/dispatch.fcgi

 which will mark the 

    
      1. dispatch.fcgi

 as modified, and thus apache will reload it.
5. 

If you have used

    
      1. killall -9 dispatch.fcgi

, then make sure you give all errors 600 seconds to timeout and create a new fcgi process.
6. 

When you upload, re-check the permissions as it overwrites files, and also make sure you don't overwrite

    
      1. dispatch.fcgi

 or your config files.
7. 

Delete your Ruby Session files, they tend to give you Application errors if things have changed. On Site5, that means running

    
      1. rm -rf /tmp/ruby\_sess.\*

. **Note** You'll get a chunk of errors as you only have permission to delete your own, and the 

    
      1. tmp

 directory is full of everyones.

