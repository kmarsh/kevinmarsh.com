---
date: "2021-03-04"
title: "20 Years Ago: SongMeanings"
slug: 20-years-ago-songmeanings
---

_20 years_ ago today I posted a [message](http://forums.winamp.com/showthread.php?threadid=43674) on the Winamp Forums[^1] announcing a project I started during Christmas vacation and had been working on for a few months:

![Screenshot from Winamp Forums announcing SongMeanings](https://icdn.remarkedusercontent.com/s/sh:0.5/rs:fit:1200/aHR0cHM6Ly9jZG4ucmVtYXJrZWR1c2VyY29udGVudC5jb20vZmlsZS9yZW1hcmtlZC1wcm9kLzEvbWFya3MvODZzTWlMblkvMjAyMTAxMTgtMTAyNTQ3LnBuZw.jpg)

The reaction was amazing and the next few months were some of the most thrilling of my life. Some of the very same ups and downs of traffic, user growth, and server issues that dot-com era startups with millions in funding were dealing with were playing out in my bedroom, in-between classes my Senior year of high school, on AIM, and on various budget shared web hosts. I never told anyone in the real world about SongMeanings. My parents didn't know, my school friends didn't know. It was just my own online thing, part of my virtual persona. It even sounded weird to say "SongMeanings" out loud because I had uttered it so few times.

I let SongMeanings slip out of my hands when I start college and it was moved forward from a scappy (illegal) site to a legit business. But money was never part of the equation with me and SongMeanings. I never put any in and never got any out. But that never mattered to me.

Looking back I realized I learned more about my software career building SongMeanings than any book, class, or degree. It was just about wanting to build something, reading how others did it, copying and hacking away until it (mostly) worked, sharing it with the world, and doing it all over again.

## Up And Running 2 Decades Later

<img class="w-64 mx-auto mb-4 sm:float-right sm:ml-4" src="https://icdn.remarkedusercontent.com/s/sh:0.5/rs:fit:512:512/aHR0cHM6Ly9jZG4ucmVtYXJrZWR1c2VyY29udGVudC5jb20vZmlsZS9yZW1hcmtlZC1wcm9kLzEvbWFya3MvZGxzbmlrTG0vU00tUmVzb3VyY2UtQ0QuanBn.jpg" />

I found an old CD-R I had burned of the code and a database backup from October 2001 and wondered if I could get it running today. To my surprise, it took about 30 minutes to get the entire site back up and running again in a Docker container, looking exactly like it did 20 years ago. Like a mosquito trapped in amber. PHP4 and MySQL 3.23 installed in fractions of a second in a [`debian/eol:potato` Docker image](https://hub.docker.com/r/debian/eol/) on a modern 32-core machine[^2] and pages rendered flawlessly in a modern browser, just like they did in IE 6 on Windows 2000 at the time:

![](https://icdn.remarkedusercontent.com/s/q:90/sh:0.5/rs:fit:1200/aHR0cHM6Ly9jZG4ucmVtYXJrZWR1c2VyY29udGVudC5jb20vZmlsZS9yZW1hcmtlZC1wcm9kLzEvbWFya3MvYVlzWWk5TW0vMjAyMTAxMTgtMTA0NjQzLnBuZw.jpg)

Practically everything works: viewing artists, commenting on lyrics, even logging in. I typed in my old username and password and was instantly shown a summary of what had happened since the last time I logged in, including 3 unread Private Messages. I viewed the very first lyric I posted to SongMeanings, Evaporated by Ben Folds Five:

![](https://icdn.remarkedusercontent.com/s/q:90/sh:0.5/rs:fit:1200/aHR0cHM6Ly9jZG4ucmVtYXJrZWR1c2VyY29udGVudC5jb20vZmlsZS9yZW1hcmtlZC1wcm9kLzEvbWFya3MvbGJzMGk0RFcvMjAyMTAxMTgtMTEyOTA3LnBuZw.jpg)

That song still has the same ID on SongMeanings today, 1: https://songmeanings.com/songs/view/1/. So much has changed but the roots are still there, intact.

The latest blog post was me trying to sound like I had any idea about what had happened just _2 days after_ September 11, 2001[^3]:

![](https://icdn.remarkedusercontent.com/s/q:90/sh:0.5/rs:fit:1200/aHR0cHM6Ly9jZG4ucmVtYXJrZWR1c2VyY29udGVudC5jb20vZmlsZS9yZW1hcmtlZC1wcm9kLzEvbWFya3MvRWJzd2k3R1kvMjAyMTAxMTgtMTExMjA0LnBuZw.jpg)

The code looks like a jumbled mess to me now, as most PHP probably was in that day. SQL queries interspersed with HTML (mostly without CSS) riddled with XSS and SQL-injection vulnerabilities. Here's a piece of PHP typical in SongMeanings 1 from probably the most run file, `showsong.php` (yep, every URL was a file... no RESTful routes back then!)

```php
<?  // showsong based on $id
GLOBAL $admin;
require_once("shared.inc");

db_connect();
//$qwer = mysql_query("UPDATE songs SET pageviews = pageviews + 1 WHERE id = \"$id\"");
//$ok = mysql_query($qwer);

session_start();
$qwer = mysql_query("INSERT INTO lyricviews (userid, lyric, date, ip) VALUES ('$user_identification', '$id', NOW(), '$REMOTE_ADDR')");
$ok = mysql_query($qwer);

$rs = mysql_query("SELECT lyrics.cover, lyrics.userid, date_format(dateadded, '%b %e, %Y') as submitdate, lyrics.artist_id as artistid, lyrics.title, lyrics.pageviews, lyrics.mbviews, lyrics.ratingcount as ratingcount, lyrics.rating as rating, lyrics.lyrics, lyrics.id, artists.name as artistname, users.username as username FROM songs lyrics, artists, users WHERE lyrics.id like \"$id\" and artists.id = lyrics.artist_id and users.id = lyrics.userid");
$exists = mysql_num_rows($rs);
```

There are probably a dozen things wrong just these 12 lines of code (out of just about 14K total.) Not to mention it was all edited by hand, in real time, on the production site via FTP. No local development, no staging server, no version control, no tests, no QA: and it was amazing.

I thought about trying to get that version up somehow to browse around, but it'd likely be compromised within hours. If that's a trip down memory lane you'd like to take, Archive.org has [captured much of the journey](https://web.archive.org/web/20020118073653/http://songmeanings.net:80/).

There's nothing tangible left from that era in my life. I never saw anyone I worked with on SongMeanings in real life or keep in contact with any of the friends I spent so many long nights chatting with on AOL and AIM. Once I was disconnected, they were gone.

I sometimes wonder what could've been, what might've become of SongMeanings and myself had I stuck around. But it's a fleeting thought, what happened happened. I instead took everything I learned and built a foundation for a career 20 years running and left nothing but fond memories.

[^1]: Much to my surprise, this original link on the Winamp Forums *still works* even after Nullsoft had been acquired and Winamp sold a couple times since!
[^2]: Of course, back in those days running real PHP was just as easy because most web hosts had it pre-installed. All you had to do was FTP up your scripts and you were up and running (well, to a point.)
[^3]: Odd how I decided to lead with news about the site instead
