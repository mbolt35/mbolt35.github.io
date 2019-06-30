---
layout: post
title: AIR loadBytes() Bites
---
I have run into an annoying issue involving Adobe AIR versions 1.5.X and 2.0. While I am aware that there are some differences (security wise) between the playerglobal APIs and the airglobal APIs, I wouldn't have guessed that what I'm about to describe to be the "valid" behavior.

### The Steps:

*   Using Flash CS, create a SWF with a document class: Foo. _For the sake of example, I created an animated MovieClip and placed it on the stage._
*   Embed the newly created SWF as binary using the mimeType="application/octet-stream"
*   Use Loader::loadBytes() to load the SWF binary.
*   Upon a successful load, extract the Class reference of the MovieClip such that multiple instances can be created.

###  The Results:

*   Using the regular Flash Player (playerglobal), I am able to successfully extract the Class and create multiple instances.
*   Using AIR (airglobal v1.5.3 and v2.0) + ADL (AIR Debug Launcher), I was able to successfully extract the Class, but using new Foo() would simply create empty MovieClip instances.
