---
layout: post
title:  "Insanity: Jekyll Installation"
date:   2021-07-28 12:35:00 -0700
categories: rant
---

If you want to drive yourself completely insane, try installing Jekyll on multiple machines, expecting to be able to edit your Github.io blog.  

I don't know anything about Jekyll or Ruby and don't want to know anything about it.  Suffice it to say, Ruby was the flagship web development language for decades, and Jekyll is used by what seems like at least tens of thousands of people.   So it should be easy to install and use, even by someone who does not know anything about the underlying technology.  Somehow, beginner's luck, I got it working on my dev machine, and put out a few posts.  Now I want to put it on my laptop, so I can post on the go.  How did I install Jekyll?  I can't quite remember...  

Back to the web to see one of the guies - install Ruby, install Jekyll, generate your site, create a post, meticulously typing in the date in the filename, then inside the post, the time, the name of the post again, tags, markdown text.  Easy enough.

How do you install Ruby?  `sudo apt install ruby`, I suppose.  Eh, not quite.  `ruby-full` for some reason.  

Can I just `sudo apt install jekyll`?  Yea, that works.  But it does not work - some bullshit about the bundler version being out of date, use gem to install the specific version of bundler, whatever that is.  Ok, type in the gem install command, it says it did the install of the latest version.  

Oh, it says I should `gem install jekyll`.  OK, `sudo apt remove jekyll; gem install jekyll bundler`.  No - now there is no jekyll executable.  And a bunch of ruby libraries seems to be gone.  Total hell.  Another hour of dicking around just to get jekyll to do something, even if it's the same error.

`jekyll version` does not work - same thing, wrong bundler version.  OK, back to the google.  Many suggestions about this kind of thing.  Remove bundler and install the new one.  Sounds good, but I can't remove the damn bundler - it is a `default gem`, whatever that means.  Back to the google.  Oh, you are supposed to do `gem env`, not the directories that gems (by now I am guessing gems are some kind of packed ruby libraries - I told you I did not want to know that) reside in, then go and delete the offending ones.  OK, deleted.

Next: reinstall bundler (gem does not seem to be aware that I already installed it several times, and installs it again).  Now `gem update --system`.  Bah, that fails after a few minutes of silence.  WTF.  Back to the google.

So somewhere, on Reddit I think, someone says "No, you can't install Ruby using your Linux package manager and later try to update it using gem.  Use your package manager to update, or reinstall Ruby the other way to use gem update --system".  What?  

Oh, now I am getting errors about using `sudo` for gem installation.  It may render the gems unusable for non-root users... Before I was getting errors about not using sudo.  Ugh.

So now I am about six hours into this.  There seems to be no way to do this, short of blowing away the Ubuntu version of Ruby and Jekyll, and getting it from the Ruby website, possibly recompiling.  To hell with all of that.  

100MBytes of bullshit, just to translate markdown to html.  This is complete garbage, and I can't believe I fell for it.  I could've written something in Forth or Lisp if I cared, and I don't.  Everyone says 'don't reinvent the wheel', but the wheel is inevitably 100Megabytes of shit code that does not work.

Oh, now jekyll does not work on my main machine - it silently refuses to process this post.  I will now delete jekyll and ruby and bundler and gems and all that garbage.  At least Github will just publish all posts...  

No wonder everything on the web is buggy - with tools like these who needs enemas...


