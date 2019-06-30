---
layout: post
title: Silverlight Vector<T>
---
I spent some time over the weekend porting the AS3 `Vector.<T>` class to C# for use with Silverlight. It uses `System.Collections.Generic.List<T>` as a backbone, so it leverages a lot of the existing functionality, but the main purpose is to keep the same method signatures as AS3.

Source:
[h](http://github.com/mbolt35/OpenSource/blob/master/silverlight/Bolt/CSharp/Vector.cs)[http://github.com/mbolt35/OpenSource/blob/master/silverlight/Bolt/CSharp/Collections/Vector.cs](http://github.com/mbolt35/OpenSource/blob/master/silverlight/Bolt/CSharp/Collections/Vector.cs)

This class would most likely assist in porting large AS3 projects to Silverlight, depending on how much Vector.<T> was used in your AS3 application of course :)

Code is written under the [Apache License](http://www.apache.org/licenses/LICENSE-2.0.html) - Enjoy!
