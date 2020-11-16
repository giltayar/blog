---
title: Backend Devs, don't upgrade to the Apple Silicon Macs... Yet!
description: Why you probably won't want to upgrad to the new Apple Silicon Macs in 2020 or 2021
date: 2020-11-14
tags:
  - docker
  - backend
  - mac
layout: layouts/post.njk
---
The [new Apple Macs](https://www.apple.com/apple-events/november-2020/) are a wonder.
As the now famous [AnandTech graph](https://www.anandtech.com/show/16226/apple-silicon-m1-a14-deep-dive/4)
shows us, Apple's ARM processors have been disrupting Intel,
and have now finally surpassed them not just in performance-per-watt,
but now also in _absolute_ performance:

![Apple ARM vs x86](/img/anandtech-apple-arm-vs-x86.png)

The above graphs show the Apple ARM A12 as being more performant than high-end x86 chips.
And to think that the M1 chips are even faster! Apple's bold claims in its event are probably
valid, and the [benchmarks that are coming in](https://browser.geekbench.com/mac-benchmarks)
lay truth that claim.

At the heart of this transition is not just a new CPU, but a new CPU _architecture_,
the ARM architecture, that comes with a different instruction set than the one we're used to:
if the previous CPUs that were in the Mac, the x86 CPUs from Intel used the x86 instruction set,
then the new CPU (the M1) uses the ARM instruction set. Thi
s means that programs compiled to the x86 instruction set will not work with the new CPU,
and so won't work on the new Macs. All Mac programs need to be recompiled to the
ARM instruction set to work on the new ARN Macs.

And yet Apple says that the new Macs are backward compatible with all our old programs. In this
blog post I'll first explain why this is mostly true, but is _not_ true for some programs, and
specifically Docker. After explaining the ramifications for Docker, we'll be close to understanding
why, for backend developers, the new Macs just aren't there yet, and what will happen in the coming
years to make them OK even for backend developers.

## How backward compatibility works on the new Macs

How do the new Macs preserve backward compatibility with old programs that use the x86 instruction
set? The simplest way to do that would have been CPU emulation, but Apple chose a different, and
much more performant way. Let's first understand what CPU emulation is, as it will help us understand
what Apple is doing, and will also help us understand later why Apple's new mechanism will
_not_ help with Docker.

CPU emulation is used by Virtual Machine software (e.g. VMWare) when the target VM CPU is different
than the host VM CPU. We're not used to this kind of Virtual Machine because for the past 20 years
we've been using just one family of CPUs: the x86 family. This means that when running a VM,
both the host machine (the actual physical machine) and the virtual machine use the same CPU.
The Virtual Machine software has it easy.

But what if we want to run an ARM virtual machine on top of an x86 physical one? Virtual machine
software knows how to do that by _emulating_ the ARM CPU: when running a block
of ARM code, it translates that block of ARM code into the x86 instruction set, and does it on
the fly. From the operating system's perspective, the machine is running ARM code, whereas
in fact it's running x86. Transparently.

This works, and has been used multiple times in the past, although as we said, in the last 30
years, this isn't used much (although I believe Android emulation when developing uses exactly
that technique). As you can surmise, this technique is _slow_.
A huge slice of CPU time is dedicated to the emulation itself,
and doesn't leave a lot to the actual program running. This technique works, and works
well, but the programs run very slowly.

This significant performance penalty (I've read somewhere that it's on the order of 50-75% slowdown,
but I can't seem to find the reference to that) is why Apple chose not to use this technique which
would have guaranteed total compatibilty, but at the cost of performance. Instead, Apple chose a more
difficult and less backward compatible way: they used a "translator", which they called "Rosetta".

Rosetta is an extension of the operating system that runs
whenever your run a program on the new Mac. Rosetta intercepts the execution of the program,
understands that the program is compiled to x86, translates
the binary code into ARM, and _then_ runs the program. Three things make translation faster than
emulation:

1. The translation is a "whole program" translation, which can probably make things faster
   than code block by code block (This is just my intuition. not sure how much this is true)

2. The translation result can be cached, so the next time the program is run, it can run
   immediately without the need for the expensive translation.

3. The operating system itself (MacOS Big Sur) runs ARM code natively, and does
   not need to be translated, which is _not_ true when running under emulation. This also gives
   a significant performance boost, because a significant amount of CPU time is in the operating
   system itself, and not just in the program being run.

This is why Apple claims _practical_ and almost full backward compatibility. I believe that
claim as we have seen this same technique used in the last Apple CPU transition: [from the PowerPC
to the x86](https://en.wikipedia.org/wiki/Rosetta_(software)), with great success.

Rosetta is not as simple as that: there is a twist in the plot that is important
to understand. Some programs generate and execute x86 code on the fly.
The premiere example for this are language runtimes: most of them don't really interpret
the language code (as it once was done in the 90s), but rather compile them to x86 on the fly, for
better performance. This includes the JVM, the .NET runtime,
and all the JavaScript execution engines (e.g. v8) that are at the heart of all browsers.
So _browsers_ use this capability. Rosetta, it seems, handles
this with aplomb, by falling back to emulation: translating the new x86 blocks of code to ARM
on the fly. Obviously, this comes with a performance penalty, as this code cannot be cached
and cannot be globally optimized.

This isn't a real problem for browsers and language runtimes,
as it is probable that these will very quickly be compiled to the new
ARM instruction set and run natively on the new Macs.

## Why Rosetta works slowly for Docker on the Mac

> Note: at the time I'm writing this post, it seems that
[Docker doesn't run](https://github.com/docker/for-mac/issues/4733) _at all_ on the new Macs. But
given that a program like Docker is a complex beast that touches the internals of everything,
that is to be expected, and will probably change in the coming months. So let's assume for now
that Docker _does_ run.

Now that I've explained how Rosetta works to ensure backward compatibility, let's understand why
it will work very VERY slowly. To do that, we need to understand three things about Docker:

1. Docker is a _Linux_ technology, and because of that,
   part of it runs in a Virtual Machine when running under MacOS.
   This is done pretty transparently by the Docker Desktop software,
   so that we don't really know that the VM is there, but it _is_ there. All your docker
   containers are actually running inside this VM.
2. Docker is a runtime: it's a program that executes docker images, i.e. executes code on the fly.
3. Docker runs x86 images: this is actually _not_ true, but as a first approximation, it _is_ true,
   so let's accept it like that till we understand more. So Docker runs only images that have
   x86 binaries.

And because Docker is a runtime, and executes x86 code on the fly in a Virtual Machine,
Rosetta has no recourse but to go the slow emulation route, rather than the fast "translation"
route. Which means that when running docker containers based on x86 images,
those runs will be **painfully slow**.

So in a few months, once all the kinks are out, you will be able to run Docker on your machine,
but those `docker` `run`-s will run very slowly: they're running x86 code on the fly,
and thus triggering Rosetta's slow emulation layer and not its fast translation layer.

## What can be done to solve this

### (and why does it complicate things tremendously)

How can we solve this? Actually, it is easy to solve: Linux can run under ARM CPUs, and Docker
supports images that are ARM images, in addition to images that are x86 images. Moreover, Docker
is amazing in that the same image can have both the ARM and the x86 images, and Docker
will choose which to run based on what CPU is running currently. So you can, for example,
`docker` `run` `ubuntu`, and it will download and run the x86 image if you're running under x86,
and download and run the ARM image if you're running under ARM. That is assuming the `ubuntu` image
has ARM support.

And this is amazing: Docker has always supported multi-CPU images. Talk about forward thinking!
Moreover, it has [experimental technology](https://github.com/docker/buildx#building-multi-platform-images) that enables you to build
and run Docker images for another CPU architecture! In a nutshell, it has QEMU emulation support
built in, so it can _itself_ build or run an image of another CPU architecture. So an ARM docker
can build and run x86 images, and an x86 docker can build and run ARM images.

Docker as a technology has never ceased to amaze me, and this new capability is no exception.
But, remember: while it can build and run an image for another architecture, it does so _painfully_
slowly.

So, perfect! We buy the new ARM Macs, run the new Docker that runs ARM images, and create and run
Docker images for ARM. Problem solved!

It'll work, except for three complications:

1. Most base images in Docker do NOT yet support ARM.
2. The Docker ARM images you create can be used by your x86 colleagues, but they will be slow
   to run. This isn't a huge problem: after all, we're talking development here.
3. The docker ARM images you create cannot be used by your x86 production machines. Yes,
   theoretically they could (via emulation),
   but you don't want to run under emulation in production, do you?

Actually, it seems that the first point is already less of a problem. The [list of "official"
images that have ARM64 support](https://hub.docker.com/search?architecture=arm64&source=verified&type=image&image_filter=store%2Cofficial)
is pretty nice and probably includes most of what you need.

But those last two points are what will cause complications in the years to come: any x86 image
is unusable running under x86 in production, and difficult to use by our x86 colleagues.
And the other way around: any x86 image generated by our colleagues will be
painfully slow to run on my ARM Macs. Moreover, our x86 colleagues will need to generate just one
docker image for their services: the x86 image, which will be usable on their machine and on
the production machines. While we, with our new ARM Macs, will need to generate _two_ images:
one ARM for ourselves, and one for the production machines. Alternatively, we can generate
just one x86 image, but again, that will run very slowly on our machine.

And _this_ is, in the end, why the ARM machines are, in my opinion, a no go for backend developers:
the ecosystem is, and will be in the coming years, an x86 ecosystem, and an ARM Cac in this
ecosystem is a pain in the butt.

## Why things aren't as bad as they seem

That doesn't sound good, does it? While consumer electronics, and most consumer computers, are
quickly migrating to ARM, will backend developers be stuck in an x86 world,
just because the production machines are x86?

I believe not. I believe production will migrate to ARM slowly over the years, for the same reason
that consumers are migrating: better performance, and better performance per watt. Once that
happens, it is the x86 machines that will be at a disadvantage, and backend developers will finally
be able to buy ARM Macs.

When will that be? Two years? Five years? 10? I don't know, but I do know that once that happens,
you will see me being the first in line at the Apple store to buy
those shiny ARM Macs that everyone has.

## Reference

* [Docker multi-CPU architecture](https://docs.docker.com/docker-for-mac/multi-arch/)
* [The new Apple Macs](https://www.apple.com/apple-events/november-2020/)
* [Blog post on docker on ARM Macs](https://chriswarrick.com/blog/2020/06/22/what-an-arm-mac-means-for-developers-and-windows-users/)

## Thanks

I want to thank [Chris Warrick](https://chriswarrick.com/contact/) whose blog post (noted above
in the reference) was my main source of information on Docker for ARM Mac. If you want the real
deal, and more information on this important issue,
go read his post: https://chriswarrick.com/blog/2020/06/22/what-an-arm-mac-means-for-developers-and-windows-users/.
