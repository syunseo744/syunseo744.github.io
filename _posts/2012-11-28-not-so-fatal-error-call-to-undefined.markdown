---
layout: post
title: 'Not so "Fatal error: Call to undefined function __()"'
date: '2012-11-28T22:03:00.005-05:00'
author: Bhaveek Desai
tags:
- tech
- lampp
- linux
- blog
- xampp
modified_time: '2018-01-08T09:39:31.956-05:00'
thumbnail: https://4.bp.blogspot.com/-ywl4SZsnfPU/ULbVW8QbuFI/AAAAAAAAAWw/GCyfP1y3TFk/s72-c/Screenshot+from+2012-11-29+08:53:42.png
blogger_id: tag:blogger.com,1999:blog-799716689661607658.post-850301545328108295
blogger_orig_url: https://bhaveekdesai.blogspot.com/2012/11/not-so-fatal-error-call-to-undefined.html
thumbnail_hd: https://4.bp.blogspot.com/-ywl4SZsnfPU/ULbVW8QbuFI/AAAAAAAAAWw/GCyfP1y3TFk/s640/Screenshot+from+2012-11-29+08:53:42.png
---

I wake up early morning with an athlete's enthusiasm to start my
project. I look outside my window at the sky still dark blue. I smile at
myself and with a little sense of achievement, I growl (in my mind of
course), "Aye! I am up before you Mr. Sun. Hell Yeah!"

Yes, it's uncustomary for me to wake up before dawn. With a heightened
excitement, I hit the switch and turn on my laptop, login and cry out a
little war-cry, "March!"

With that, opens up a terminal I call my "dArKd3ViL's c0n$0l3" (not
voice operated if you're wondering :p)

And then type in `sudo /opt/lampp/lampp start`

Shoot out my browser and type in `localhost`.
XAMPP welcomes me royally and loyally. I nod at it's grace and go to
visit my another loyal servent, phpmyadmin.

That's when I hit the ground.

![](https://4.bp.blogspot.com/-ywl4SZsnfPU/ULbVW8QbuFI/AAAAAAAAAWw/GCyfP1y3TFk/s640/Screenshot+from+2012-11-29+08:53:42.png)

"What the fuck is this?"  
Oh ok, 1st instinct told me to correct the function syntax on the
specified line in the file mentioned. I did it as root to not trifle
with permissions. I did ponder with my chin resting on my hand and my
finger on lip, "how the hell can the syntax change by itself?"

When I couldn't think of an apt answer, I saved the change and restarted
xampp/lampp whatever you wanna call it.

This time phpmyadmin curses with:

![](https://3.bp.blogspot.com/-j_iIcHSRVFE/ULbKuBCxW1I/AAAAAAAAAWQ/4md-I6Agh0M/s400/Screenshot+from+2012-11-29+08:08:35.png)


"Thou shalt burn in hell for not obeying thy master's orders!", I was
furious.
Then I looked closely at my terminal. I had been ignoring the messages
lampp was displaying on starting lampp; I looked closely:

![](https://2.bp.blogspot.com/-eSZdW2v7l-U/ULbNmUsi4jI/AAAAAAAAAWg/z74PRX3q9Ak/s640/Screenshot+from+2012-11-29+08:20:40.png)

MySQL was not getting started!
Next, I chmoded `/opt/lampp/etc/my.cnf` to
`755` after looking at the warning and thinking
this is probably why I am stuck and the Sun who was already up then was
laughing at me.

So I again restarted and yet again the same dreadful sight of

![](https://3.bp.blogspot.com/-j_iIcHSRVFE/ULbKuBCxW1I/AAAAAAAAAWQ/4md-I6Agh0M/s400/Screenshot+from+2012-11-29+08%253A08%253A35.png)

Aaaarrrgghhhh!
Wrong permissions, wrong permissions!!!
OF WHAT?
The answer was in front of me and in my angst I hadn't noticed it.
`wrong permissions on **configuration
file**`

Since phpmyadmin had the problem, I suspected the config file of
phpmyadmin is what I needed to fiddle with in the 1st place.

So I chmoded
`/opt/lampp/phpmyadmin/config.inc.php` to
`755` and made it "not world writable"

Restarted xampp/lampp, visited the fucker phpmyadmin and it did bow down
loyally this time.

Reply in the comment section below if you had a similar problem with
phpmyadmin or what you think about this blog.

Cheers! :)
