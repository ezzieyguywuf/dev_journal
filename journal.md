2021-05-29
==========

Ok, not a ton of time for dev today (it's a saturday and my daugther is still
here [whoot!]) but I did want to record that I want to try a different strategy
with the unreal engine ebuild.

Till now, I've been trying to use their build tools. It essentially goes
something like this:

1. run a `GetDependencies.sh` script (I'm paraphrasing) to go out onto the web
   and download stuff - this won't work in portage due to the network sandbox
2. run some sort of `Setup.sh` script that...sets things up. Even this seems to
   call some `exe` executables that download `dll`s from...somewhere. Again,
   that won't fly in portage
3. Run a `GenerateBuildFiles.sh` script (again, paraphrasing) to
   generate...build files. Like, `CMakeLists.txt`

I've looked at these `CMakeLists.txt` files, they're kind of wonky and use a
bunch of `add_custom_target` calls, and then they execute a bunch of build
scripts.

So, here's the new approach: just skip all of that. Instead, I'm going to take a
stab at just writing my own `CMakeLists.txt` from scratch to build the source
directly.

I highly doubt it will be succesful. But we'll see.

_not that long later_

k, so `find . -name "CMakeLists.txt"` in a "vanilla" project dir versus one
that's run the `GenerateBuildFiles.sh` actually results in only one difference -
a top-level `CMakeLists.txt`. It seems that all the sub-folders already contain
`CMakeLists.txt`, which could actually make our crazy plan somewhat less
crazy...

_after the kids are in bed_

I started learning a bit about C++20 - wow it seems cool! Between modules and
ranges (and probs some other features added since C++11 that I wasn't aware of),
it seems like I may be able to use some of my functional, haskell workflows in
C++20. The `co_yield` thing seems rather pythoney, but that's not bad - I guess
they're trying to modernize the language? or maybe python just did it better,
and so they're copying?

Whatever, it's neat. I should do a project to get my feet wet with these
features. Oh, concepts, yea those are cool!

2021-05-28
==========

Whoot, UE4Editor finished overnight. I haven't cranked it up yet, though,
because I'm busy udating my windows 10 vm so my daugther (I guess I'll keep her
name out of  this) can play some minecraft bedrock edition later.

I know what you're thinking "But wolf, you can play minecraft in linux natively
if you use the java version." Yes, this is true, but whenever she's not here,
she plays on her nintendo switch, which only has the bedrock edition. So if we
want to play together, then I have to crank up windows 10. Even though she'll be
playing alone today, the world we've built is all on the windows 10 vm, so there
you have it, let's go do 2 hrs worth of windws updates.

That being said, I've got some more things to add to my list of  things I want
to work on but currently don't have time to (should I keep all these in a
separate document? I'm thinking yes...)

- Figure out how to adjust which drive SeaBios will try to boot from first
    - this is a problem for me because I pass through two drives, and the one
      that SeaBios tries to boot is NOT the one with the OS on it, so it fails
    - My previous solution was to just his `<Esc>` to manually select the
      correct drive from the menu
    - for some reason, SeaBios is not recognizing my keyboard. Leading too...
- Figure out why SeaBios is not recognizing my keyboard
    - It doesn't appear to be a qemu problem, b/c once Windows 10 boots up it
      sees the keyboard just fine
- Explore virgl: someone said I could probably get very good graphics
  acceleration with this, and **not** have to forward my graphics card to the VM

_later that day_

Sweet, windows 10 update finished! However, the audio is all garbled, ugh! I
think last time it had something to do with "MSI Interrupt" I think, so I'll
google around and see if I can re-do it. It doesn't seem to bother my daughter,
so I'll just let her enjoy her game-time for now.

_right before bed_

dang - packaging unreal engine is turning in to a real rabit hole.

so i've deciphered most of the setup scripts, but now I'm stuck because they
ship this GitDependencies.exe executable that magically fetches a bunch of dlls
from...somewhere. I dunno.

i may have to abandon this cause...I mean, I can gathwr and self-host tbe stuff
that GitDepemdencies.exe fetches, but i have no way of knowing if that violates
any copyrights or licenses.

I'll press on for noq but it's not lookimg good...

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
