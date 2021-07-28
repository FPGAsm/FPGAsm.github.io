---
layout: post
title:  "Insanity: delete your /var/lib and see what happens"
date:   2021-07-28 14:08:51 -0700
categories: rant
---
I am on a roll.  After a couple of days of trying to get Jekyll to work, I gave up and tried to delete it.  `sudo apt remove` what?  `ruby`? `jekyll`? `gem`? `bundler`?  I think all of those worked.  But when deleting ruby, a warning showed up that the directory `/var/lib/ruby/gems..` or something like that was not deleted.  OK, `sudo rm...` F\*\*\* S\*\*\*.  Slip of the finger, and *POOF* the entire `/var/lib/` is gone.

## A tragedy of errors

OK, it can't be that bad, right?  It's var, so it should not be anything too permanent.  My system still works, apparently, enough to open browsers, edit this file, pcmanfm file manager still works, command line is ok.  So, a deep breath...

First, back up!  Copied the entire home directory to the backup drive.  Copied my m2 work directory to the backup drive.  Now, how to recover?

`dpkg` is broken, and the google answers say I am mostly out of luck.  If I remember which applications are installed, maybe there is hope.  How the **** can anyone  be expected to remember the thousands of packages that were installed over several years?  But there is another way, maybe, although it is asking for trouble -- I have a notebook with the same distro and at least some package overlap.  I will copy the `/var/lib/` directory from there and hope for the best....

## On backup

Oddly enough, about a month ago I had a terrible feeling about this.  I ordered an LTO-3 tape drive, just for this eventuality.  I should've had it weeks ago, preventing this disaster, but two packages from different vendors, one with the drive and the other with the SCSI card got lost in the mail.  I watched with growing frustration as one was bouncing for 15 days between two states, while the other one was stuck in the same place, according to the tracking info.  I requested a refund, but since the packages were in the mail, no luck.  Eventually they showed up a few days ago, but the SCSI card was defective, and was returned.

In the meantime, I ordered a used LTO-5 drive from EBAY, as it is more appropriate with its 1.4TB capacity.  I suppose datacenters are always upgrading, and these can be had for a couple of hundred bucks.  Or you can pay a few thousand dollars to some unlucky individual who paid five grand for one and now wants at least a couple of thousand back...  I will take my chances.   But it's of no help here, since the drive is coming later today and the Fiber Channel card in a couple of days.

## More Linux Idiocy - rsync failures

I've always used rsync for backups (which I haven't really done lately).  It seems to be what Linux people do.  So why the **** did it lock up on me while trying to copy the var/log directory from my laptop?  Is it my transfer drive?  No, it is always the same file - a 1.4kb text file I can open and look at with no issues, or copy by hand!

Google this... Apparently it's a thing - rsync has been locking up like this for years!  I see a message from 2018 where the guy can't believe they haven't fixed it yet.  Is it because I am copying to a drive formatted for windows and the -a argument does not work?  Many think so.  But it does not work without the -a either.  Many say that -v, the verbose command is the culprit.  How the hell can you tell if it's working without it - I have a couple of gigabytes to copy.  Why is `/var/log` more than 2GB on my humble notebook?  That is completely crazy.

Screw `rsync`.  I copied the directory using my file manager, started as root.  Let's see if it works...

Maybe one of these days I can actually do some work.

To be continued
