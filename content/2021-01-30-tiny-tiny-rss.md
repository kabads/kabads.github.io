---
categories:
- reading
date: "2021-01-30T00:00:00Z"
title: Tiny Tiny RSS
---

[Tiny Tiny RSS](https://tt-rss.org) is a simple syndication application.  I've been using RSS syndication readers for a long time. I just don't have time to scour the web to check if a website had added new content. I started with [Google Reader](https://www.google.com/reader/about/) a long time ago (as did many of us), but was also sad that it went away. So, then I moved to [Newsblur](https://newsblur.com/). This suited me and helped me read my blog posts for another year or so. It also has (or had - I haven't checked recently) an open source model. However, with the pricing it was offering, I didn't need to host it myself as it was doing just a fine job, for the price that I paid. Then the price went up. For some reason, I felt that it was too much. I was probably hasty, but also ready for a change. 

{{< rawhtml >}}
<a data-flickr-embed="true" href="https://www.flickr.com/photos/kabads/50889452698/in/dateposted/" title="rss"><img src="https://live.staticflickr.com/65535/50889452698_c0e227c6c9_n.jpg" width="300" height="150" alt="rss"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

I switched my feeds to [The Old Reader](https://theoldreader.com). This was an obvious attempt to recreate the experience with Google reader. 

Recently, I installed [TinyTinyRss](https://tt-rss.org) on a raspberry pi. There was a package already, but it attempted to install Apache - but I was already running Nginx, so didn't want this extra dependency - I therefore downloaded the source code, put it in a directory that was served via the webserver and everything worked instantly. I had already creaed a database and user/pass combination in mariadb, so the setup was simple. At this point, I imported an ~.ompl~ file that I already had. I then read about running the update script. I started a tmux terminal and issued the command ~/usr/bin/php /path/to/tt-rss/update.php --feeds --quiet~ and this runs automatically. 

You can add feeds using the following dialogue:

{{< rawhtml >}}
<a data-flickr-embed="true" href="https://www.flickr.com/photos/kabads/50891429801/in/dateposted/" title="ttrss3"><img src="https://live.staticflickr.com/65535/50891429801_00e949529a_n.jpg" width="320" height="291" alt="ttrss3"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

The output is really good to look at:

{{< rawhtml >}}
<a data-flickr-embed="true" href="https://www.flickr.com/photos/kabads/50891546277/in/dateposted/" title="ttrss1"><img src="https://live.staticflickr.com/65535/50891546277_90a54ae621.jpg" width="500" height="229" alt="ttrss1"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{{< /rawhtml >}}

There is also an [app for your phone](https://play.google.com/store/apps/details?id=org.fox.ttrss) to hook up to your server, so you can read your articles on the go.



