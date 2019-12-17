---
title: "Game automation with Python. An introduction to the Pytomatic Framework"
date: 2019-12-15T20:00:00+01:00
---

*Lets start this post like every single food blog post ever.*

I love systems, and i love putting things in systems. I'm also an gigantic fan of automation.
When i first started programming it was with the AutoHotkey language back in 2008 or so. For those who don't know AutoHotKey
it's a scripting language which is made for macro creation and software automation. For my own part it all started with 
me looking into how to save myself for some of the grind in some old ass MMO and generally just make me not having to play games. 
Like for example [Dark Pirate](https://www.mmofacts.com/dark-pirates-2128) MMO. And that Fish tank game [Insaniquarium](https://en.wikipedia.org/wiki/Insaniquarium) which is waaay older than what i thought!
It's something incredibly satisfying to create a script that is stable enough to grind all night long and when you are in school.

Questions of morality aside, i've always loved that sort of challenge. I generally avoid doing this to games where is would affect the game notably.
It's always the online games that got a worst sort of grind, so that generally where i have been doing this sort of thing on a small scale.

Anyway, back to AHK. I felt already then that the syntax was quite ugly and strictly lack luster. After a year or so i discovered the language AutoIt3.
After some reading up on the differences it seems AHK is a fork of the open source AutoIT2 and AutoIT3 is a closed source language.
Not really something that affected 14 years old me...

Over the same period i also started touching in on the E2 language from the add-on to Garrys mod called [Wiremod](https://wiremod.com/)
and assembler via the Cheat Engine debugger. All in all it's not hard to see that my main motivation for learning to program
was a strong wish to make games to what i wanted them to do, not so much make my own things.

After an lull over several years i rediscovered my love for game automatization via the Battlefield 4 Commander mode again.
I created an okay enough AutoIT3 script which was able to play the game competently enough. And as an added bonus. I **actually** managed to save an copy of it!

![Class scores](/Pytomatic/battlelog_class_score.png)

After finishing up the script in AutoIT3 i figured this ought to be possible to do with Python!
What i found for the job was the PyAutoGUI framework, which is super nice but with one big flaw i can't look past.
Namely that you have to use the API which moves the mouse on the screen. Effectively locking usage of the computer to the application your are automating.
Personally i try to use the Windows message API to send mouse events directly to applications so that i can still use the real mouse for whatever i feel like.

Here is an example of PyAutoGUI by [u/AirHamyes](https://www.reddit.com/user/AirHamyes/)
<video controls width="100%"><source src="/Pytomatic/AirHamyes_paint.mp4" type="video/mp4"></video>

Note how the mouse pointer is bouncing around among all the different colors. This locks the mouse to the Paint app. Which is lame and can be avoided.

Other than that i also had issues finding any good modules for image analysis and the likes, so overall creating my own thing seemed like the only sane thing to do!

# What is the challenges that needs to be solved?

Let's look at the script for BF4!
[Github: BF commander.au](https://github.com/N0K0/pytomatic/blob/master/samples/BF4/_old_autoit3/BF%20commander.au3)

After about 40 lines with of definitions we find the [following](https://github.com/N0K0/pytomatic/blob/master/samples/BF4/_old_autoit3/BF%20commander.au3#L42-L49)

{{< highlight vbs "linenos=table,linenostart=42" >}}
WinWait ($winTitle)
$winHandle = WinGetHandle($winTitle,"")
if($winHandle = 0) then
   Exit(1)
EndIf

WinSetState($winHandle,"",@SW_SHOW)
WinActivate($winHandle)
{{< / highlight >}}

```WinWait``` simply spinlocks the script until we find an window with the correct title, when that windows is found we fetch the handle with ```WinGetHandle```
After that we let our paranoia take overhand and doubly make sure the window we are looking for is visible and on top with ```WinSetState``` and ```WinActivate```.

Next up might look wierd, but what this line does is start the wait process for the enemy in an active game.
{{< highlight vbs "linenos=table,linenostart=50" >}}
getRedPos(1000000000000000000)
{{< / highlight >}}

After that we start our main command cycle:
{{< highlight vbs "linenos=table,linenostart=54" >}}
while true
   $iteration = $iteration + 1
   $activeWindow = WinGetHandle("[ACTIVE]")
   $mPos = MouseGetPos()
   WinActivate($winHandle)
   WinMove($winHandle,"",3,3,683,417)
   useUav()
   useSpec()

   if Mod($iteration,25) == 0 Then
	  orderSquads()
   EndIf

   Sleep(1000)
WEnd
{{< / highlight >}}
First thing first is making sure the BF4 window is still active and in the correct area of the screen.
Ofter that we will try to use our scouting drone and whatever other special ability we got on that map.
And last but not least we also try to order out different squads, but we only do this every 25 cycles so not to spam players with commands all the time.

The ```useUav()``` , ```useSpec()``` and ```orderSquads()``` methods are quite similar. With the main difference if the order of where we are going to click our buttons.

{{< highlight vbs "linenos=table,linenostart=111" >}}
func useSpec()
   $delay = 200

   $pos = getRedPos()
   click($pos,"right")
   Sleep($delay)
   offsetClick($pos,$specialCommands1)
   Sleep($delay)
EndFunc
{{< / highlight >}}

And the ```useUav()``` method:

{{< highlight vbs "linenos=table,linenostart=122" >}}
func useUav()
   $delay = 200
   $pos = getRedPos()

   click($pos,"right")
   Sleep($delay)
   offsetClick($pos,$scanOffset)

   Sleep($delay)

   $pos = getRedPos()
   click($pos,"right")
   Sleep($delay)
   offsetClick($pos,$empOffset)
   Sleep($delay)
EndFunc
{{< / highlight >}}

Mainly what we see here is the usage of the ```getRedPos()``` method and some different commands related to how to click
the screen based on the return from the previously mentioned method.

The ```getRedPos()``` is quite big but i can sum it up with that we are fetching images of the game window where we are looking for a shade of the color ```0xFCAA85```

As for the switch case at [line 158](https://github.com/N0K0/pytomatic/blob/master/samples/BF4/_old_autoit3/BF%20commander.au3#L157-L166). 
The reason we are using this sort of setup is because the pixel searcher function in AutoIT3 scans one line at the time. 
The default is left to right, top to bottom. This can be changed by swapping the X and Y params, which is what happens in the switch.
It's not perfect  but it works. An other issue with AutoIT3 is that the PixelSearch method returns the first pixel, without any possibility for filtering on size or returning multiple "spots" with one run over the screen.
The rest of the ```getRedPos()``` is simply a fallback if we are unable to find the color, so we end up using an avarage of earlier found points