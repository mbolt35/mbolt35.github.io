---
layout: post
title: SOS Max Logging for Silverlight
---
If you are not familiar with [SOS Max](http://www.sos.powerflasher.com/developer-tools/sosmax/home/?L=122), it's a socket output server that is used to display logging messages sent via a client application for debugging. The beauty behind this concept is that it supports any language with socket support, and since SOS Max is written in Java, it's cross platform. At Electrotank, we use it for most all our client side projects in AS3. To do this, we use an SOSLogTarget class written by Sönke Rohde. You  can find more information for the AS3 version at:  [SOSLogTarget](http://soenkerohde.com/2008/08/sos-logging-target/)  
  
[](http://soenkerohde.com/2008/08/sos-logging-target/)I was piddling around on Twitter this morning and saw that [Jobe Makar](http://jobemakar.blogspot.com/) had tweeted that he was going to have a go at Silverlight. The next few tweets were his remarks on some annoying issues with simply logging output to the Console in Firefox (which, off-topic, he found the solution here: [http://bit.ly/9YpZF7](http://bit.ly/9YpZF7) in the bottom post). His issues oddly inspired me to check out Silverlight myself, as I have some experience with C# and XNA, so it was coming one way or another -- why not today?  
  
Immediately, I noted that Jobe's problem was, in fact, just as annoying as he said. So, to solve this issue on a larger scale, I wrote a **very basic** SOS Max logging target that works with the Silverlight C# API. The full source (with an example) can be found here:  [SOSLog.cs](http://github.com/mbolt35/OpenSource/raw/master/blogspot/silverlight/Bolt/CSharp/SOSLog.cs)

However, because Silverlight has some pretty annoying security policies when it comes to sockets, you have to run the Silverlight application "Out of Browser." To do this:  
*   In the **Solution Explorer**, right-click your application and select **Properties.**
*   Select the **Silverlight** tab and check the option that says "Enable running application out of the browser"

[![](http://2.bp.blogspot.com/_yO39pG0HdLE/TF27dhgkI6I/AAAAAAAAACk/_Tiptox_Jog/s320/silver-light1.png)](http://2.bp.blogspot.com/_yO39pG0HdLE/TF27dhgkI6I/AAAAAAAAACk/_Tiptox_Jog/s1600/silver-light1.png)

  

*   Then click the now enabled "Out-of-Browser Settings..." button.
*   Check the option that says "Require elevated trust when running outside the browser."

[](http://4.bp.blogspot.com/_yO39pG0HdLE/TF26yzWldVI/AAAAAAAAACc/hSnO1wLMBt4/s1600/silver-light1.png) [![](http://1.bp.blogspot.com/_yO39pG0HdLE/TF24wW6MBmI/AAAAAAAAACM/LY1n6fwttxI/s400/silver-light2.png)](http://1.bp.blogspot.com/_yO39pG0HdLE/TF24wW6MBmI/AAAAAAAAACM/LY1n6fwttxI/s1600/silver-light2.png)  

*   Press "OK"

Now you should be all set to go... 
[![](http://2.bp.blogspot.com/_yO39pG0HdLE/TF25Okm80NI/AAAAAAAAACU/JjgOe7-QtYc/s400/silverlight3.png)](http://2.bp.blogspot.com/_yO39pG0HdLE/TF25Okm80NI/AAAAAAAAACU/JjgOe7-QtYc/s1600/silverlight3.png)

Hope you find this useful - The source is written under the Apache License, so you can feel free to use it how you want.
