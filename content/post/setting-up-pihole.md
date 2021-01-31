+++

author = "Jacob Hell"
title = "Setting Up my PiHole, a Short Story"
date = "2021-01-31"
tags = [ "pihole"]

+++

I just couldn't take answering surveys on my smart TV anymore.

<!--more-->

### Setting Up my Pi Hole, a Short Story

On a weekend night, I often find myself diving through the depths of YouTube. Unfortunately, every 5 minutes, I get interrupted by an ad for something I don't want like makeup subscriptions (I long ago disabled personalized tracking).

One day, I was perfectly content learning about Kombucha making. All of the sudden, Google absolutely needed me to fill out a survey. I held back a scream of rage and filled out the survey. This surrender truly shook me to my core.

Later that night, unable to sleep, I realized what I had to do. "I need to install Pi Hole!" I yelled with new found confidence.

## The Installation

So setting up Pi Hole is supposed to be really easy. It's often one of the first projects people install on their pi. To install it you just run a single bash command:

`curl -sSL https://install.pi-hole.net | bash`

Easy peasy I thought, I should back to Kombucha videos in no time.

## The Router, the Pace 5268AC

"Not so fast", my router/modem combo said. It continued: "You can't change DNS settings on me!" 

"Darn" I replied. A smile lit suddenly across my face. "Well, good thing I can use Pi Hole as a DHCP server too!"

The router/modem combo let out a hearty chuckle. "Not so fast, you can't change DHCP settings either!"

The blood rushed to my face.

## The New Router

I bought a new router. I plugged it into the uplink port on my old router/modem combo. I could change the DNS settings. Victory.

My Raspbian installation imploded, figuratively. It wouldn't connect to the router via ethernet or wireless. Victory had been lost. To add insult to injury, I did not own an SD card reader, so I could not re install Raspbian. 

Full of sorrow, I walked to Walmart and bought an SD card reader.

I reinstalled Raspbian. I reinstalled Pi Hole. It could connect to the router. I was blocking ads. My life became complete.

## Lessons Learned

1. Make sure you can change DNS or DHCP settings on your router. If you can't, you need a different one. Or a different firmware like [DD-WRT](https://dd-wrt.com/).
2. Have an SD card reader when working with Raspberry Pis. You never know when you need to reinstall the OS.
3. Despite the challenges, a Pi Hole is fun and worthwhile.

