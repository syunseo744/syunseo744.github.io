---
layout: post
title: Bulk adding iradio stations in Rhythmbox
date: '2012-11-15T06:46:00.003-05:00'
author: Bhaveek Desai
tags:
- tech
- rhythmbox
- linux
- blog
modified_time: '2018-01-08T09:40:13.834-05:00'
thumbnail: https://2.bp.blogspot.com/-Ay6xtb0ChhI/UKSHIEhHQ4I/AAAAAAAAARo/mPkabQ3rJ7g/s72-c/Screenshot+from+2012-11-15+11:39:20.png
blogger_id: tag:blogger.com,1999:blog-799716689661607658.post-8830144774454187742
blogger_orig_url: https://bhaveekdesai.blogspot.com/2012/11/bulk-adding-iradio-stations-in-rhythmbox.html
---

Every time I upgraded my system fresh installing a new Ubuntu release, I
had this habit of taking a backup of my config files/folders, like for
example..
`~/.mozilla` for retaining bookmarks, browser
preferences and settings, etc
`~/.xchat2` for retaining irc logs,
autoconnect channel listings, channel settings, etc
Some more like these have really saved me all those hours of
reconfiguring some of my software after reinstalling them on my new
system.

Now, I never really cared which particular configuration file was used
for what as long I ensured backing up and restoring the complete
configuration folders. Rhythmbox, which is an awesome music player for
GNU/Linux systems, has been my choice of music player since I switched
from Windows. Awesome it is and so I had to maintain my config files so
that I didn't have to re-rate my favourites, create all my playlists
over and over again every time I re-installed it on an upgraded system.

But this one time, while switching from Ubuntu 12.04 to 12.10, I took
backup of all my config folders as I do, and as I think I did, I somehow
missed out Rhythmbox. So after re-installing, I couldn't find my
Rhythmbox backed up files. Re-rating and creating playlists began. After
3 days, I was done, except for one thing.. recovering my list of
favourite radio stations that I had added to the default listing
Rhythmbox provides.

I remember the first time I added my own bit to the list. Hours spent
searching for Internet radio stations of specific genres and sub-genres,
adding them, one by one on Rhythmbox. It was only a one time headache,
of course if the files were backed up.

Now what to do?
Should I go back to the old way of repeating my configuration or do I do
something more optimized?

Firstly, I searched around for the file(s) which had information about
the radio stations. I used "gnome-search-tool", which is a lightweight
searching tool with optional parameters. The parameter I needed was -
search string in content. I picked up one of the default listed radio
station which had the name - "University of Wisconsin", searched for any
files in Rhythmbox folder locations which had this name in its content.

If you sneak around in your file system, you will find multiple
locations where Rhythmbox resides for different purposes..

```
~/.local/share/rhythmbox
~/.cache/rhythmbox
/usr/share/rhythmbox
/usr/share/rhythmbox-radio-browser
```

each having many files in them. I triggered the search in these
locations. Found 2 files containing the mentioned string in their
content..

`/usr/share/rhythmbox/plugins/iradio/iradio-initial.xspf` and,  
`~/.local/share/rhythmbox/rhythmdb.xml`

So I found the files where I could add my list of radio stations. But
which one is the useful one?

The first one seemed like a useless file to me. I moved the file to
another location and restarted Rhythmbox. As opposed to the expectation
of finding the default list empty, I saw no change in the default
listing.

Then I started exploring the 2nd file - `rhythmdb.xml`. As the name
suggests, it is the maintained database of Rhythmbox mapping each and
every song in the library to their location on the disk. If you scroll
down, you will find the default list of radio stations appended at the
end.

Now, each radio station had an entry of the following format as depicted
using the example of one of the default stations:

{% highlight xml %}
<entry type="iradio">
    <title>WSUM 91.7 FM (University of Wisconsin)</title>
    <genre>College Radio</genre>
    <artist></artist>
    <album></album>
    <location>http://vu.wsum.wisc.edu/wsumlive.m3u</location>
    <play-count>2</play-count>
    <last-played>1352942258</last-played>
    <date>0</date>
    <media-type>application/octet-stream</media-type>
</entry>
{% endhighlight %}

Next, the tags, in a structured format, which matter here are:

{% highlight xml %}
<entry type="iradio">
   <title></title>
   <location></location>
</entry>
{% endhighlight %}

The genre is picked up automatically when you play the particular
station.
Okay, so now I needed to get a list of my favourite radio stations with
the information about their name and location.

Okay, now you must be wondering, "What the hell! It's the same bullshit
over again. What's new in this?". Be patient. The donkey work is just an
illusion. I have about 60 radio stations to be added. Instead of adding
a new radio station the traditional way via the Rhythmbox interface, and
then right clicking on it and then copy pasting the name/title of the
radio station, I did the following..

1. I created a new text file containing the list of my favourite radio
stations with their title and location.
Mind that, I haven't yet added the XML tags, only created a simple list
of radio stations.
An excerpt from my `stations.txt` file:  
![](https://2.bp.blogspot.com/-Ay6xtb0ChhI/UKSHIEhHQ4I/AAAAAAAAARo/mPkabQ3rJ7g/s1600/Screenshot+from+2012-11-15+11:39:20.png)
Each Entry has the title followed by the location of the radio station.

2. Next, and the most important step is to append XML tags at
appropriate locations. Now, I would, in fact anyone would, lose their
head if they'd sit down and add the tags for each entry element; like I
said, I have about 60 stations in the list.
That's where the power of vim comes in to picture. Edit one entry in a
macro and apply the macro x@a where -
```
x: number of remaining entries and,
a: name of the macro
```

3. Before this step, ensure rhythmbox is **not** running in the
background. Copy the revised contents of stations.txt files and paste it
in the end of `rhythmdb.xml` file, **but just before the closing </rhythmdb> tag** (that's the last line of the file)
4. Save it
5. Open Rhythmbox
6. Voila!
7. Ok, enough of unnecessary steps.
8. Watch this screencast I uploaded showing how each of the above
steps (except steps 6-8) are executed including how to make and run a
macro in vi.  
<iframe width="640" height="532" src="https://www.youtube.com/embed/WAASMEsxVWk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


  
Cheers!

Comment in the section below.. your thoughts, if you faced the same
problem, what did you do, etc.
