+++
date = "2015-02-04T23:03:54Z"
draft = false
title = "Diving into golang: 1: An opinionated introduction"
tags = ["golang", "programming", "vim", "tutorial"]
series = ["Diving into golang"]
+++
In  the past six months, i’ve been exclusively doing most of my backend programming work from a programming language known as Go or golang.

The features of the language that appeal to me are:
   * Its speed, which is really close to that of C/C++.
   * Its simplicity. You can keep the whole language syntax in your head, and not have to consult the documentation for every line of you write. 
   * Its concurency. In case you don’t know, concurrency has to do with being able to run parts of the code simultaneously. In this our generation of multicore device processors, your application can fully maximise the number cores of the device, without unnecessary complexity in implementation.
   * Completeness. With a full featured standard library, we dont need to spend all our time searching for third party libraries or implementing low level features our selves.

These are just a few of the features of golang, as that really isnt the purpose of this post.

Back to “diving into go” First, head over to the golang official website and follow the instructions to set up Go on your development environment, whether on windows, linux or mac.

For me, my development environment is a laptop running Ubuntu 14.04 LTS, a linux operating system. I use VIM as my promary text editor, due to its simplicity. If you use linux and would prefer to use vim, you can visit this github page on how to set up VIM for Go.If you’re very new to vim, you can download and install vim from your software repository, to get through the basics of vim in a few minutes, head to the linux terminal, then type and run vimtutor.

Irrespective of your operating system and text editor (on windows, you can simply use notepad++ ), the next steps you take are what determine how much of agolang programmer you’d become.

For me, my journey was simple, I started by taking the go tour, which is in my opinion the fastest way of learing Go. The tour is available online at tour.golang.org, but you can download it and run it offline, if you’ve already set up Go on your environment. Run 
        
        go get code.google.com/p/go-tour/gotour

Then in your terminal, run the executable by typing and running

        gotour
        
Learn the basics of everything, variables, control structures, loops, etc, but nothing more, as fast as you can. The trick is learning just what you need at first, and not trying to keep everything in your head.

After going through the tour, the next steop is to come up with a project and work on it. You can get project ideas at https://github.com/karan/Projects.

This step is what makes you really matters. Each project you complete boosts your moral and gives you moe confidence in your skills. Remember to use maximal use of google, stackoverflow, and github source codes, to solve any hitches you experience. The major traits of a professional programmer, is not being able to code, but being able to solve problems and find solutions.

When you feel comfortable in your skills, go ahead to contribute to any open source project on github. Again, you can read this blog post “getting started with open source contributions” if you are new to open source contributions.
