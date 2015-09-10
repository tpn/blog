---
layout: post
title:  "I Wonder If... I Can Implement Automatic Coroutines in PyParallel"
date:   2015-11-08 07:30:00
categories: iwonderif pyparallel advanced
---

> Thursday, September 10th, 2015.

I wonder if I can implement something that will intercept a normal,
synchronous blocking call, and automatically convert it into an asynchronous
coroutine-esque call behind the scenes if the current load warrants it.

Synchronous blocking I/O calls have less overhead than overlapped asynchronous
I/O calls, however, they prevent a thread from doing anything else until that
call has completed.

Windows can detect this and automatically create new threads for the given
thread pool in response.  However, thread creation is relatively expensive.

The assumption is that an overlapped asynchronous I/O call is more expensive
than a synchronous I/O call, but less expensive than creating an entirely new
thread on demand.

Ideally, we would like to dynamically be able to switch between synchronous
I/O and asynchronous I/O, depending on the circumstances.  If we have a low
client count and a lot of cores, synchronous I/O will provide the lowest
latency.  If we have a high client count, asynchronous I/O will provide the
fairest latency.

Luckily, PyParallel already has this functionality built into it for socket
I/O. We detail this [here][pyparallel-speakerdeck-#116].

The code that achieves this is the [`PxSocket_IOLoop()`][github-pxsocket-ioloop]
function.

What we'd like to do is extend the functionality to cover two additional
cases: file I/O and ODBC database calls.  We'll try file I/O first as we
perceive it to be a bit easier; we already have some asynchronous file I/O
support within `pyparallel.c`, and we don't need to fiddle with the `pxodbc`
module, which is in a separate DLL.

> *Do we want async file I/O?  Is ODBC going to be better?  ReceiveFile should
 be a thing.*

```python
from async.http.server import (
    HttpServer,
)

class Server(HttpServer):
    def do_POST(self, request):
        request.transport.open(request.path, 'w')

```

[github-pxsocket-ioloop]: ...
[pyparallel-speakerdeck-#116]: https://speakerdeck.com/trent/pyparallel-how-we-removed-the-gil-and-exploited-all-cores?slide=116
