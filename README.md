# The real history of async networking
I constantly get the impression today's employers don't really know what they are looking for or what they really need. I see how hip and modern start-ups working with everyday non-mindblowing server solutions search for people with experience in Node.js and nothing else.

They somehow believe a C programmer with decades of experience cannot possibly understand this completely new async networking stack. Because, we all know Node.js and JavaScript are completely new technologies never before seen. A C programmer cannot possibly wrap their head around async code since C is not an async language like JavaScript. You cannot possibly grasp the concept of Node.js and JavaScript if you do not have 20 years of experience with *only* it.

It's not like a C programmer can go into the Node.js universe and write something better than the best available. That would be completely impossible, C programmers know nothing about Node.js.

## Berkeley sockets (1983)
Our async networking trip begins with the legendary BSD non-blocking sockets from the early 80s. By marking a socket non-blocking you could handle multiple connections from a single user space thread. If your intention was to manage 100 connections you would perform the syscall `select` to block your thread until at least one of the involved non-blocking sockets had triggered an event to be handeled. Identical in structure to how modern event-loops work, you implement the loop something like this:

```c
// the event-loop blocks until select returns
while (select(...)) {
// the event-handler for the correct socket executes
}
```

The biggest difference between modern event-loops and loops based on `select` is the linear performance overhead that comes with not knowing which socket actually triggered the event. You see, `select` doesn't tell you *which* socket fired, but only the fact that *some* socket fired an event. The single thread managing all connections would have to loop over all the involved sockets and check their flag values to determine if they were the cause of this trigger:

```c
// the event-loop blocks until select returns
while (select(...)) {
// iterate over all sockets
for (...) {
  // is this the socket?
  if (...) {
    // the event-handler for the correct socket executes
    break;
  }
}
}
```

## poll (1986)
With the `select` syscall solving the problem of handling many connections per user space thread, the `poll` syscall iterated on the interface and had a couple of different properties. One big difference is how `select` destroys the input data while `poll` does not. There are more details but in a nutshell, they are irrelevant to this history.

## WinSock (1991)
The WinSock API was designed with heavy influence from the BSD non-blocking sockets from the Unix world. The BSD interface is still very relevant in the Unix world today but has been (optionally) partly replaced with a different interface in the Windows world.

## Windows 3.1 (1992)
As a point of reference, not really related to networking but more of a design pattern acknowledgement. The Win16 API from Windows 3.1 had C programmers build async GUI programs pretty much the exact same way as they would build an async networking app. By having one user space thread call `GetMessage` in a loop, the thread would block until an event (here called a message) was available:

```c
// the event-loop
while (GetMessage(...)) {
// the event-handler
}
```

Instead of `select`, a GUI program calls `GetMessage`. This pattern of design where one single thread blocks until an event is available and the handles this event is called an event-loop. All event-loops, no matter what language, work like this. Some kind of blocking syscall in a loop. That's it, the event-loop pattern.

## IOCP (1995)
The problem of having a linear search over all involved sockets each triggered event started to really harm performance as the number of connnections increased. Surpassing 1000 connections with `poll` is not without a fair share of looping overhead.

With Windows NT 3.5, Microsoft had implemented I/O Completion Ports, IOCP. Instead of looping over the sockets, the call to `GetQueuedCompletionStatus` would block and return with a pointer to the socket that fired the event. This would make the looping unnecessary and allow appliactions to scale to a much higher number of connections using only a single user space thread.

## Visual Basic 6.0 (1998)
Again, not related to networking history but rather as a point of reference. Visual Basic 6 was an entire development environment based on having a single thread handle a bunch of different events. These could be networking events from the built-in WinSock support, or GUI events such as when the user clicked a graphical button. It was an entire, complete event-driven environment based on async programming.

In my personal opinion, Visual Basic 6 was the true grandfather of today's Node.js. Nothing that Node.js brings today was not already thought about, solved, and delivered back in the late 90s. We had an easy-to-use single-threaded BASIC language modelled after the English language. Anyone could program async apps, even a child, like me.

I remember doing a lot of VB6 up until 2004. I had created everything from simple file-transfer services we used in school to share akwardly low-res pirated videos and games, to fully working 2d MMORPGs. VB6 :')

## kqueue (2000)
FreeBSD 4.1 implemented the first version of `kqueue`, the Unix response to IOCP. In a nutshell `kqueue` has the same advantages and solves the big problem with `poll`.

## epoll (2002)
With the release of Linux 2.5.44 we now had truly scalable networking I/O cross-platform.

## libevent (2002)
A cross-platform async event-loop written in C that wraps epoll, kqueue and IOCP in an easy to use interface. Grandfather of libuv.

## NGINX (2004)
It is important to know that NGINX, a modern and very popular high-performance webserver, load-balancer and proxy was written many years before Node.js and that Ryan Dahl himself designed Node.js with NGINX as an example of how it should be done.

## ASIO (2005)
Boost ASIO is one of the most widely known asynchronous and modern C++ networking libraries to date. Work initially started 2003 and the library was committed into Boost 2005. Now used by many and looks to be included into standard C++17.

## Node.js (2009)
"Node.js® is a JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient."

Node.js is nothing new and nothing revolutionary. From the perspective of a PHP programmer it might be something new but from the perspective of an experienced C programmer this piece of software is nothing more than a simplification of common (ancient) async techniques wrapped up in JavaScript for non-experienced programmers to toy with. Any experienced programmer can easily understand this software stack.

Node.js is a runtime written in C++ (V8) and C (libuv, OpenSSL, zlib). You write server logic in JavaScript, bound to underlying C code. Everything you do (tcp, udp, files, events) is directly connected to C code. Any net.Socket is bound via V8 to an underlying uv_tcp_t responsible for handling any async events, any non-blocking event stems from the Linux kernel via epoll (or whatever system you use). Anything you do in JavaScript is really just acting as a director, controlling the underlying C functionalities which in turn implement the real async functionality.

## A note on "async" languages

So how come C, the language from the 70s is able to implement this kind of async software stack? Basically, JavaScript is not async at all. Async is not a property of the language but rather a property of the libraries which the language makes use of. You can believe JavaScript to be async, but according to computer science JavaScript is rather *less* async than C given that JavaScript has no threading support. An imperative computer program cannot possibly execute anything exept for what is described at the PC register, unless you have threads. What happens if you perform some kind of computation in your Node.js app? Simple, the app will freeze because there is only one event-loop and only one thread of execution. Good luck implementing anything async in JavaScript without the use of threads.

## The brainwashing procedure (~2014 and beyond)

Ever since Node.js has gotten itself blown up in terms of trendiness most everyone seems to have completely forgotten about the history. I see people working as high-paid software developers claim async networking to be something new and never before seen. These are Silicon Valley hot-shots making ridiculously false claims about the histroy while showing absolutely no knowledge about what really happens under the surface, completely incapable of applying the most basic computer science to their claims. People cannot believe me when I tell them I wrote async servers back in the 90s, even though async networking has been the very foundation of any Unix system since the 80s. Node.js is marketed with such enormous focus on "high performance" and "event-driven" making other solutions which take these design decisions for granted seem irrelevant and ancient. People are starting to lose grasp of what is reality and what is just a marketing campaign gone explosive.
