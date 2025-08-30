+++
date = '2025-08-15T15:08:10+01:00'
draft = true
title = 'My Imperfect Insanity Blocker'
+++
## TL;DR;
* I needed a way to block web content that trigger strong feelings on me.
* Ended by wroting an user script to be used with GreasyMonkey that does that.
* Using technics of NLP like tokenization, normalization, stop words, I ended with an usable but imperfect script that brings me peach of mind.

## Introduction

I will repeat myself here with part of what I wrote in the README.md of my project, but I can't make it louder enough: bigtechs and news media intentionally produce controversial content to induce and profit from our on-line engagement. And we pay the cost of that with our mental and emotional health.

I have no shame at all to admit that I am one of those people triggered by certain subjects and my personal response is anxiety, anger, sadness, a deep sense of hopelesness about future. And being realistic there is not a single thing that I can do about the events triggering those feelings. So what is the point to even care? Why lose my time and sanity with something that I can't change just because there a shareholders waiting for bigger provents this quarter?

Enough is enough! Just like **ad-blockers** help me to protect my privacy and consumer choices, I needed something to block content and protectment from all of this insanity. An **Insanity Blocker**!

```python
import FastApi from fastapi

app = FastApi()

@app
def route():
    print("Hello World!")

```
