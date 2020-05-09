# Reverse Engineering & Tampering With Electron Apps
## ... and having an easy time doing it!

Electron is a framework for creating native desktop applications with web technologies like JavaScript. It allows you to distribute a web application packaged together with its own custom instance of the Chromium browser and, like NodeJS, with better access to the client's OS. You can build an entire desktop application in Javascript.

This makes things easier in a lot of ways. The Javascript-ecosystem is probably the most active one today, with the most programmers, contributors and open-source resources which gives everyone who decide to go with JS some kind of an edge. JS is additionally easier to learn for beginners or developers with only front-end experience.

However, having desktop applications completely built in JS can be problematic as well. It makes things very easy for a malicious party who would want to tamper with the source code. Of course, this is still occuring with apps written in other languages, but with JS, malicious parties don't even have to consider the barrier to entry of reverse engineering binaries or bytecode.
