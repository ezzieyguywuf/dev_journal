2021-05-27
==========

First entry. I have no real plan for how I want this structured or laid out.
I'll start by listing the current "active" projects that I'm working on, in
order of activity:

- package Unreal-Engine in gentoo (i.e. make an ebuild)
- finish becoming an official Gentoo Dev (I need to schedule time for an
  interview)
- A bunch of gentoo-related stuff: rstudio rev dump, dwarf-fortress rev-bump...
  I don't think I'll list them all
- mycad

Next, here's some stuff that I want to be working on but don't currently have
time (or one of the ones above needs to be finished first):

- make a simple game in Unreal Engine
- make a small project to learn the C++20 stuff
- rewrite mycad (AGAIN!) in C++ (20?): I think this is the best bet if I ever
  want anyone other than myself to contribute

So, let's talk about what's going on with Unreal Engine ebuild

- I downloaded the source (yay!) - this wasn't easy, but I guess I 'get it',
  they put a lot of work in to it then make it free, so they sort of ask you to
  sign your life away, nbd.
- I made a very bare-bones ebuild that asks you to download the dist file
- I also cloned the repository and I'm trying to build it "locally" (i.e.
  outside of portage) first
    - first attempt failed - I followed the instructions to downloads
      dependencies and generate a cmake file. Then I did `mkdir build; cd build;
      cmake ..; make -4` but it crashed
    - the docs mention something abaut `make StandardSet`, which is _supposed_
      to be the default. Anywho, I tried that and now it's running, we'll see
      how that goes.

I can predict that the following will be things I need to resolve for the
ebuild:

- fetch all the dependencies manually in `SRC_URI` - this looks like it could be
  QUITE the headache, as there are TONS of files, like windows DLL's and stuff.
  I think these may be needed so the thing can build cross-platform games?
- figure out what the cmake generator is doing..and just write our own cmake
  file
- Figure out what "StandardSet" means - and then whether we want to offer an
  ever barer-bones version of the ebuild
    - There's other make targets that the docs discuss, add these as USE-flags
    - Or don't - does the "StandardSet" provide anything that's worthwhile or
      useful on its own?

_later that same day_

Alright, reading through some of the build output, catching some notes here:

- I think "StandardSet" includes the following:
    - UnrealLightmass
    - CrashReportClientEditor
    - ShaderCompileWorker
    - CrashReportClient
    - I think this because of the following output I saw:
```sh
Running command : Engine/Binaries/DotNET/UnrealBuildTool.exe UnrealLightmass Linux Development
Running command : Engine/Binaries/DotNET/UnrealBuildTool.exe CrashReportClientEditor Linux Shipping
Running command : Engine/Binaries/DotNET/UnrealBuildTool.exe ShaderCompileWorker Linux Development
Running command : Engine/Binaries/DotNET/UnrealBuildTool.exe CrashReportClient Linux Shipping
```
- k, so it's definitely using it's own (downloaded) clang compiler. ugh - but
  let's just build it this way for now, get something up and running. But it'll
  be real nice to try to get it to use my system clang
    - based on output `using toolchain located at
      '/path/to/my/git/clone/Engine/Extras....etc<some clang stuff>'`
- ugh, bundled libc++ and standard c++ library. w/e, stay the course
- Ok, I guess things build in this order:
    1. UnrealHeaderTool
    2. CrashReportClientEditor
    3. UnrealLightmass
    4. ShaderCompilerWorker
    5. CrashReportClient
    6. UnrealFrontend (this one is honking big)
    7. UnrealInsights
    8. UE4Editor (nah, this is in honking HUGE)

So right now, UE4Editor is still cranking. What I'm wondering though is do I
absolutely **need** the editor in order to make games? Or is it just a super
nice IDE? We'll see.
