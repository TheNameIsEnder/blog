+++

author = "Jacob Hell"
title = "Eclipse versus IntelliJ"
date = "2020-05-09"
description = "Eclipse versus IntelliJ"
tags = [
    "java",
    "ide"

]

+++

Let's compare these two IDEs.

<!--more-->

# General Info

### Eclipse

Eclipse was created by IBM and released in 2001. Eclipse natively supports Java, but has plugins for a ton of other languages. Eclipse is free and has no paid version.

### IntelliJ

IntelliJ was released in 2001 by JetBrains. IntelliJ natively supports Java, Groovy, and Kotlin, but has plugins for a ton of other languages. In the paid version of IntelliJ, JavaScript, SQL, Ruby, and a few other languages are supported.

# Debugging

### Eclipse

In Eclipse, to obtain the values of variables in scope requires you to either:

* Mouse over the variables and see the value
* Add the variable name to the "expressions" window.

That's too much manual work for this guy.

### IntelliJ

In IntelliJ, all variables in Scope are already present in the "Variables" window. No manual work is required by me!

# Refactoring

### Eclipse

Eclipse has refactoring for renaming files and packages (and updating the name of the references in other files), and extracting code into super classes. It doesn't offer much for refactoring the lines of code itself though. 

It can also suggest things like unused variables, infinite loops, and unreachable code.

### IntelliJ

IntelliJ offers much more options for creating super classes, even going into the member and inline code level.

IntelliJ offers suggestions for almost any line of code. 

# Windows

### Eclipse

The biggest problem I have with Eclipse is something I call "Window Fatigue". There are SO many windows in Eclipse, and they always end up in strange places. The concept of "Perspectives" makes this even more confusing!

For example, I will have the console window open while coding. Then when I run the program and start debugging, Eclipse will switch to the "debug" perspective. This opens and changes the position and size of all windows that are defined by the debug perspective. Once I'm done debugging, they all go back to the normal perspective.

It might just be me, but I continuously find myself moving stuff around when using Eclipse. And it really wears on me.

### IntelliJ

In IntelliJ it's harder to change the window positions. I haven't found myself trying to move stuff around when coding. This is a good thing for me. I can't be trusted with my constant window moving.

# Usability

### Eclipse

For whatever reason, loading and closing a project in Eclipse can cause problems. For example, sometimes after reopening a project, files will no longer be found. Or there will be random errors when trying to start a program, only fixed by removing the project and re importing it.

I'm not sure why this is. I have spent many hours trying to figure out issues with code, only for it to be an Eclipse problem.

### IntelliJ

Never had a random problem like this in IntelliJ. However, due to Eclipse being the most used IDE for some time, figuring out a problem is relatively easy. Since there is a good chance that someone else has had the same problem and posted their issue on the internet.

# Conclusion

I didn't intend for this post to be dropping bombs on Eclipse. IntelliJ is just a lot better in my opinion. The only major drawback is that if you are not doing desktop Java development, you need to buy the paid version to assist with your coding.

