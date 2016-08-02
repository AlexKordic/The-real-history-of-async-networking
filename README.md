# The real history of async networking

## The misconception (2016)

I constantly get the impression today's employers don't really know what they are looking for or what they really need. I see how hip and modern start-ups working with everyday non-mindblowing server solutions search for people with experience in Node.js and nothing else.

They somehow believe a C programmer with decades of experience cannot possibly understand this completely new async networking stack. Because, we all know Node.js and JavaScript are completely new technologies never before seen. A C programmer cannot possibly wrap their head around async code since C is not an async language like JavaScript. You cannot possibly grasp the concept of Node.js and JavaScript if you do not have 20 years of experience with *only* it.

It's not like a C programmer can go into the Node.js universe and write something better than the best available. That would be completely impossible, C programmers know nothing about Node.js.

## Berkeley sockets (1983)
Our async networking trip begins with the legendary BSD nonblocking sockets from the early 80s. We had nonblocking sockets and accompanying event-loop mechanisms like select and poll before we even had the internet.

These non-blocking socket interfaces are still used today and are the base of any async networking in any modern networking application. Virtually unchanged, except for the evolution of event mechanisms like epoll and kqueue.

You implement the event-loop by calling select or poll, which will return when any event has been fired (readable, writable) on any of the involved sockets. You then handle this event with your event-handler by either calling recv to receive the data or send to send data using the writable socket:

```c
// the event-loop
while (poll(...)) {
// the event-handler
}
```

## Winsock (1991)
Pretty much a clone of Berkeley sockets but for Windows systems.

## Windows 3.1 (1992)
Even though Windows is by far newer than X from 1984, the introduction of Windows 3.1 marks a huge impact on async programming in C. Every GUI program ever written for Windows is async. That's how all GUI programs are made. They work pretty much identical to the Berkeley sockets, you implement an event-loop with GetMessage (analogous to poll/epoll):

```c
// the event-loop
while (GetMessage(...)) {
// the event-handler
}
```

Events in GUI applications could be WM_PAINT, WM_CLICK, and the like to notify the application of clicks, redraws, etc.

## I/O Completion Ports (1995)
With Windows NT 3.5 and forward, Microsoft had implemented the epoll / kqueue equivalent which allowed one to efficiently wait for events on multiple sockets.

## Visual Basic 6.0 (1998)
A completely async scripting-like environment in BASIC with TCP sockets and GUI events based on Windows and Winsock. You could write complete graphical UIs with async click handlers, async network handlers, etc. A complete "scripting environment" far before anything Node.js ever existed.

You had your usual list of events like:

* Close
* Connect
* ConnectionRequest
* DataArrival
* Error
* SendComplete
* SendProgress

I've personally written countless async servers in VB6 multiple years before anything like Node.js ever existed.

## kqueue (2000)
With FreeBSD 4.1 we got a high performance event-system via kqueue. Just like epoll in Linux this feature allows servers to scale to millions of connections with no linear search of ready sockets.

## epoll (2002)
With the release of Linux 2.5.44 we got an upgrade to the event-system via the epoll syscalls. epoll is what is being used today and is what allows any networking server to scale to millions of connections without having to perform a linear seach like poll does.

## libevent (2002)
A cross-platform async event-loop written in C that wraps epoll and kqueue in an easy to use interface. Grandfather of libuv.

## NGINX (2004)
Async modern webserver, load-balancer and proxy written in C. Very popular still to this date.

## ASIO (2005)
Boost ASIO is one of the most widely known asynchronous and modern C++ networking libraries to date. Work initially started 2003 and the library was committed into Boost 2005.

## Node.js (2009)
"Node.js® is a JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient."

Node.js is nothing new and nothing revolutionary. From the perspective of a PHP programmer it might be something new but from the perspective of an experienced C programmer this piece of software is nothing more than a simplification of common (ancient) async techniques wrapped up in JavaScript for non-experienced programmers to toy with. Any experienced programmer can easily understand this software stack.

Node.js is a runtime written in C++ (V8) and C (libuv, OpenSSL, zlib). You write server logic in JavaScript, bound to underlying C code. Everything you do (tcp, udp, files, events) is directly connected to C code. Any net.Socket is bound via V8 to an underlying uv_tcp_t responsible for handling any async events, any non-blocking event stems from the Linux kernel via epoll (or whatever system you use). Anything you do in JavaScript is really just acting as a director, controlling the underlying C functionalities which in turn implement the real async functionality.

## A note on "async" languages

So how come C, the language from the 70s is able to implement this kind of async software stack? Basically, JavaScript is not async at all. Async is not a property of the language but rather a property of the libraries which the language makes use of. You can believe JavaScript to be async, but according to computer science JavaScript is rather *less* async than C given that JavaScript has no threading support. An imperative computer program cannot possibly execute anything exept for what is described at the PC register, unless you have threads. What happens if you perform some kind of computation in your Node.js app? Simple, the app will freeze because there is only one event-loop and only one thread of execution. Good luck implementing anything async in JavaScript without the use of threads.

## The brainwashing procedure (~2014 and beyond)

Ever since Node.js has gotten itself blown up in terms of trendiness most everyone seems to have completely forgotten about the history. I see people working as high-paid software developers claim async networking to be something new and never before seen. These are Silicon Valley hot-shots making ridiculously false claims about the histroy while showing absolutely no knowledge about what really happens under the surface, completely incapable of applying the most basic computer science to their claims. People cannot believe me when I tell them I wrote async servers back in the 90s, even though async networking has been the very foundation of any Unix system since the 80s. Node.js is marketed with such enormous focus on "high performance" and "event-driven" making other solutions which take these design decisions for granted seem irrelevant and ancient. People are starting to lose grasp of what is reality and what is just a marketing campaign gone explosive.
