---
layout: post
title: Function to check balanced parentheses
tags: [interview,algorithms]
---
I did a telephone interview with Amazon this morning, and this was one of the questions they asked:

> Given a string comprised of only 3 sorts of parentheses (round), [square] and {curly}, and no other
> characters, write a function to detect if its balanced.

Some examples of balanced input:
```
([{}])
([]){}
[()]{}{[()()]()}
```
Some examples of unbalanced input:
```
[(])
([]){}(
```

My first thought was to use a Stack, but of course, my mind went blank and I started seeing
problems with that approach that weren't there. I guess that's interviews for you!

Anyway, a few hours later I decided to attempt it again on my own. It was actually a pretty
simple function to write, and I got it done in a few minutes. I'm a little disappointed that
I couldn't figure it out properly during the interview, but that's ok. It was a great
experience anyway (my first proper interview in about 20 years!), the interviewer was very
helpful, and I've a better idea what to practice for the next time :-)

Check out the code in [Java](https://github.com/mcmullm2-dcu/balanced-parentheses/blob/master/java/Brackets.java)
or [C#](https://github.com/mcmullm2-dcu/balanced-parentheses/blob/master/csharp/Brackets.cs).
