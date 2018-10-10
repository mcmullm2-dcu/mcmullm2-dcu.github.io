---
layout: post
title: Reflections on SDA Assignment 1
tags: [sql,d3,data visualisation]
---
Our first assignment in SDA (Mobile Application Development) was to create
a simple Android app that displayed a bit of text towards the top of the
screen and add a button below that displayed a 'Toast' pop-up when pressed.
It was a simple app, but a great way of getting our feet wet with Android
development.

## Challenges
The biggest challenge for me was getting a working AVD running on my AMD laptop. Looking through StackOverflow, it seems to be a common issue, and most solutions didn't seem to work for me. Either the device would refuse to run, or else it would crash when trying to run the assignment. My backup plan was to use my work desktop (thanks GitHub!), which worked fine, but I'm glad I persevered: there's now a functional AVD up and running on my laptop. I think removing HAXM, turning on virtualisation in the BIOS, and a few other hacks seemed to create the right environment.

The only other real difficulty was trying to get the DatabaseDemo code running. This was an existing app that we had to clone from GitHub and run on our local device. The Saturday Workshop, along with a little StackOverflow, pointed me in the right direction, which led to editing the version number in `build.gradle` file and changing the project structure settings. The specific instructions online didn't work for me, but by comparing the settings with a working project, it came together in the end. I'm currently going through some Gradle tutorials outside of Android Studio at the moment to get a better understanding of it.

## Adding UI Elements
Everything else worked out well. Android Studio made it easy to quickly place UI components, but the assignment was structured so that delving into the layout XML was necessary (for example, to change to `RelativeLayout`). The graphic interface was useful as a reference though: the 'View All Attributes' link gave a useful list of what properties were available.

By default, all elements seemed to pile up on top of each other, so it was interesting to look through a few positioning options. There seemed to be options to position elements precisely, a bit like absolute positioning in CSS. However, positioning elements in relation to other elements, or the parent layout itself, seemed more flexible. For example, I aligned the button to the bottom-centre of the parent, and when I rotated the screen to landscape, it adjusted itself appropriately.

One interesting aspect was the idea of 'start' and 'end' instead of 'left' and 'right' -- this makes layouts flexible enough to accommodate right-to-left reading languages, which I hadn't considered previously until Android Studio suggested it.

Having separate XML resource files was interesting. It's clearly going to come in useful, especially as projects become more complex. I played around with a couple of other resource types, like dimensions (dimens.xml) and creating a custom icon.

## Debugging
Debugging was also worth playing around with. Logging messages seemed similar to how we would have worked in JavaScript. Also, the Android documentation gave a good tip about defining tags as constants to ensure consistency. Finally, setting up breakpoints and stepping through code line-by-line generally worked as expected. However, it didn't seem to attach immediately, so if I was looking at a button's click event handler, the breakpoint didn't kick in until I pressed the button a second time. I'm not sure if that's normal, or if my setup needs some tweaking.

## GitHub
Git/GitHub are extremely useful. It was great being able to build up the project bit by bit, and explain each step in commit messages. If something went drastically wrong, it was good to know that I could just go back to the last working commit, which in turn meant I could experiment a bit without worrying too much about making mistakes. I noticed that some files weren't being tracked -- some through `.gitignore`, and others that I must have missed along the way. Once I got everything I needed together on GitHub, I was able to clone the whole repository on a different computer and run it. It was reassuring to know that everything needed was backed up, and that I (or someone else) could make further changes on another machine if required.

## Conclusion
So all in all it was a good assignment, I learned a lot about the basics of Android development and finding my way around Android Studio. Plus it was fun seeing my app show up on my phone!
