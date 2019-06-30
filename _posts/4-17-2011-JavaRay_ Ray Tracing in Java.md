---
layout: post
title: JavaRay - Ray Tracing in Java
---
I've been tinkering around lately with Java, and trying to catch up on the last 8 years I've missed. I was first introduced to Java in 2002 at UNC-CH, but it was nothing like the Java today.  
  
One thing I still find very interesting is that while Java's performance is exceptional, and the language itself is flexible and OO-centric, you don't see many games written in Java (besides the highly successful title, MineCraft). Perhaps my perspective is blurred, but nonetheless; I didn't understand why.  
  
At Clemson University in 2004, I took a Computer Science course from a professor named Mike Westall in which we built a ray tracer in C. It was a very basic single-threaded application that took a "scene configuration" and image dimensions as parameters, then output a PPM file.  
  
I thought re-writing this project in Java would be fun, so I started working on it this weekend, and here's what I have so far:  
[![](http://4.bp.blogspot.com/-1t-W5V2lmVE/TatDZYahhwI/AAAAAAAAADU/mkcjfzpiRQU/s320/image.png)](http://4.bp.blogspot.com/-1t-W5V2lmVE/TatDZYahhwI/AAAAAAAAADU/mkcjfzpiRQU/s1600/image.png)

It supports both ambient and diffuse lighting. I plan on adding:  
* specular lighting  
*  multithreading  
* custom scene configuration  
  
### Update
Made a few updates tonight including multi-threaded rendering (blog post to come) and specular lighting, although there is a bug in the specular lighting that I can't figure out. Here's the latest:  
[![](http://1.bp.blogspot.com/-slBq8Xart94/Ta5122E-M2I/AAAAAAAAADg/1XuAZV1xVEY/s320/test.png)](http://1.bp.blogspot.com/-slBq8Xart94/Ta5122E-M2I/AAAAAAAAADg/1XuAZV1xVEY/s1600/test.png)

Anyone have ideas about why the objects reflect on the \*other\* side of the sphere? It should only be the objects between the sphere and eye point. Looks as if there's some incorrect calculation concerning the normal, but I've been unable to locate it. :(

It's nothing too exciting, but I'm making progress. You can find the source to the project here: [http://github.com/mbolt35/javaray](http://github.com/mbolt35/javaray)  

