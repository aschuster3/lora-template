---
layout: post
title: "The birth of Slackbotbot"
permalink: the-birth-of-slackbotbot
date: 2017-01-22 18:00:06
comments: true
description: "The birth of Slackbotbot"
keywords: "slack robot side project"
categories:

tags:

---

So this story starts late 2015, when somebody shows me [this video](https://www.youtube.com/watch?v=kvctQKIXM-o) and tells me we need to make a
Slackbotbot for the office. I agree and begin scouring the internet hoping somebody has done all the work for me with some script. No dice, so I
begin my perilous journey into Slackbotbot.

Some time passes (read: like, a year) and after a number of false starts I finally get some momentum going with the project.  Admittedly, the
turning point was someone writing a [Python API for the Create 2](https://github.com/simondlevy/BreezyCreate2).

For starters, here's what I gathered to begin work. Most of this I had lying around from other hardware projects that I never did:

**Raspberry Pi 2B** | This probably works with other versions, but this is what I had.
**iRobot Create 2** | Surprisingly, there already existed a maker Roomba! Your mileage may vary with a standard Roomba. It comes with a serial to USB.
**Reachargable Portable battery** | I bought this when PokémonGo got popular, so now I've repurposed it here.
**McRoboface (optional)** | [Couldn't resist](https://www.kickstarter.com/projects/4tronix/mcroboface)
**SunFounder Project Super Starter Kit** | Apparently they still sell these.  Honestly, any sort of prototyping kit will do.

I had the latest version of Rasbian Jesse installed and updated everything before starting. Hooking up the Pi to the Roomba was as simple
as plugging in a USB. I had to remove the faceplate on the Roomba to reach the serial port.

Step one is to crack into the Python API and see what kind of limitations we can expect to see from the Roomba. Some exploration showed that
the API doesn't natively expose as much as I would like, but it doesn't take much work to coax out what we want. This gives us our first snippet!

{% highlight python %}
from breezycreate2 import Robot
import time

# This is what the API actually exposes
outerBot = Robot()

# This is an inner reference that allows me to do
# some more fine tuned work
bot = outerBot.robot

# The commands are non-blocking, so I have to
# block manually with some well placed sleeps
bot.turn_clockwise(100)
time.sleep(1)
bot.turn_clockwise(0)
time.sleep(0.1)
bot.drive_straight(300)
time.sleep(3.3) # Gotta last as long as the action
bot.drive_straight(0)
time.sleep(0.1)

# Close the connection because we're not heathens
outerBot.close()
{% endhighlight %}

Success! The Roomba fires to life, does a quick turn, and lurches forward!  I had to put it back on the charger myself, but hey,
progress. Before closing out my work, I went ahead and (with the help of a friend better at this kind of stuff than I am) made
some songs to play through the Roomba. Without explaining the whole Open Interface, the Roomba can store 4 "songs" of a certain
length, and can play them back. After writing a quick helper to form and save the songs, we've got a singing vacuum.

{% highlight python %}
from breezycreate2 import Robot
import time

# Helper to form the notes the way the API wants them
def addNote(note, duration, eighths=12):
    return note + "," + str(int(duration * eighths))

# A classic
def createOdeToJoy():
    ode_to_joy = ""
    ode_to_joy += addNote('E4', 2) + ","
    ode_to_joy += addNote('E4', 2) + ","
    ode_to_joy += addNote('F4', 2) + ","
    ode_to_joy += addNote('G4', 2) + ","
    ode_to_joy += addNote('G4', 2) + ","
    ode_to_joy += addNote('F4', 2) + ","
    ode_to_joy += addNote('E4', 2) + ","
    ode_to_joy += addNote('D4', 2) + ","
    ode_to_joy += addNote('C4', 2) + ","
    ode_to_joy += addNote('C4', 2) + ","
    ode_to_joy += addNote('D4', 2) + ","
    ode_to_joy += addNote('E4', 2) + ","
    ode_to_joy += addNote('E4', 3) + ","
    ode_to_joy += addNote('D4', 1) + ","
    ode_to_joy += addNote('D4', 4)
    return ode_to_joy

# Set everything like last time
outerBot = Robot()
bot = outerBot.robot

# In the background, this puts the song into
# one of the Roomba's "registers" and then
# calls it up, but for our purposes it just
# plays the notes
bot.play_song(1, createOdeToJoy())

outerBot.close()
{% endhighlight %}

Some final notes: A problem that came up consistently at this stage was that the Roomba did some weird things if you
didn't time the sleeps properly, but fortunately we can read the state of the Roomba to find out if it's busy. Going
forward, some wrappers around the existing methods will be helpful so that we don't have to worry about timing every
time we make a call to do something.

For the code here with a few other goodies and another song, I've added the code to GitHub [here](https://github.com/aschuster3/slackbotbot-source/tree/master/the-birth-of-slackbotbot).
