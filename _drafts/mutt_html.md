---
layout: post
date: 2017-05-27
category: Notes
tags: tech email
description: Displaying HTML emails from mutt
---

I love minimal terminal based applications where ever I can use them. This means that I use [mutt](http://www.mutt.org){:target="_blank" rel="noopener noreferrer"} for viewing emails on my Linux system. One thing that is becoming more common is the use of HTML emails, which don't display very well in the terminal. Instead of seeing the content as the sender would like you to, all you see is the HTML code.

By adding the following lines to your ".muttrc" config file
```
alternative_order text/plain text/html
auto_view text/html
```

and creating a new ".mailcap" file in your home directory with the following (change browsers Firefox and Lynx to your preferred applications)
```
text/html; /usr/bin/firefox %s >/dev/null 2>&1; needsterminal
text/html; lynx -dump %s; nametemplate=%s.html; copiousoutput
```

next time you open HTML emails in mutt the content is displayed in your browser.

Thanks to [TerminalMage.net](http://terminalmage.net/2014/03/16/how-i-read-html-email-with-mutt.html){:target="_blank" rel="noopener noreferrer"} for the tip.