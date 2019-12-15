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

If you are intrested in poking in the script it's hosted over [in the pytomatic repo.](https://github.com/N0K0/pytomatic/blob/master/samples/BF4/_old_autoit3/BF%20commander.au3
)

After finishing up the script in AutoIT3 i figured this ought to be possible to do with Python!
What i found for the job was the PyAutoGUI framework, which is supernice but with one big flaw i can't look past.
Namely that you have to use the API which moves the mouse on the screen. Effectively locking usage of the computer to the application your are automating.
Personally i try to use the Windows message API to send mouse events directly to applications so that i can still use the real mouse for whatever i feel like.
