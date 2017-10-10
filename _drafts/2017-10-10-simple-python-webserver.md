---
layout: post
date: 2017-10-10
category: Notes
tags: 
description: Using python as a local webserver
---

Sometimes it can be handy to run a webserver on your local machine for testing purposes. It is not, however, always possible to install a webserver such as Apache. Never fear! Python has you covered.

With one python command you can run a local webserver in which ever directory you're in. First we need to install python
```
pacman -S python
```

Next check the version
```
python -V
```

If you have `Python 2.x` then run the command
```
python -m SimpleHTTPServer <port>
```

If you have `Python 3.x` then run the command
```
python -m http.server <port>
```

If you don't specify the port then the default is `8000` for both commands.

Use `Ctrl-c` to stop the webserver.

That's it, simple!
