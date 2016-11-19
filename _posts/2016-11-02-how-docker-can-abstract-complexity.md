---
layout: post
section-type: post
title: How Docker can abstract complexity
category: Docker
tags: [ 'Docker' ]
---

Github is a wonderful magic land until you want to run a project that is written in some language that you just have no idea about. In my case, this would be something like C++. I've never worked with C++ nor would I have any idea how to get started. This is where Docker comes in.

If you're just new to Docker then this will hopefully get you interested in some really cool open source projects. After all, Docker itself is open source and hosted on Github.

*Let's take an example:*

### OCR (Optical character recognition)

Have you ever wanted to just throw images into the command line and have the text abstracted? maybe you can think of some lucrative application that reads images and does some magic with the content. Maybe you search around for an open source tool to do this sort of thing and end up running into [tesseract](https://github.com/tesseract-ocr/tesseract).

![Ocr](https://bundyfx.github.io/bundyfx-blog/img/posts/2016-11-18-how-docker-can-abstract-complexity/1.png)

Awesome, but how the hell do I use this thing?

Enter Docker, Well more so the Dockerfile. If you look through the repository you may notice that there is a Dockerfile included.

![Ocr](https://bundyfx.github.io/bundyfx-blog/img/posts/2016-11-18-how-docker-can-abstract-complexity/2.png)

This is always awesome to see because it allows people to use the project whilst abstracting some of the complicated aspects of running C++ projects on different operating systems.

So how do we run this in Docker? Firstly we need to create an image from the project. This is essentially the purpose of the Dockerfile. All we need to do in docker is run:

`docker build . -t ocr`

What we're saying here is that we want to build a new image in our current working directory (indicated by the '.', assuming you're in the project root) and we want to tag our new image with the name 'ocr' (can be anything).

Once we've done that Docker will read in the file and create a new image that's ready to use.

Now we've got our image created let's just try the software. According to the documentation, it says you can just run tesseract from the command line and pass in some params.

Well, let's get a picture to test with.

For this test, I'll go with something pretty straight forward designed to test OCR functionality.

![Ocr](https://bundyfx.github.io/bundyfx-blog/img/posts/2016-11-18-how-docker-can-abstract-complexity/3.jpg)

Firstly we will copy the image to our container then run it against OCR and see how it goes.

![Ocr](https://bundyfx.github.io/bundyfx-blog/img/posts/2016-11-18-how-docker-can-abstract-complexity/4.png)

How cool is that? And since its being returned to stdout and we're in PowerShell we can very much so pipe that information on to whatever we want just like any other object.

Take a look around Github and challenge yourself to try running projects you find in Docker.
