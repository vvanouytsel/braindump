#mac #os

As I am transitioning from Fedora to MacOS, some of the pain points will be written down here.

# Gnome like Workflow

I was used to the GNOME workflow on my Fedora. To get the same functionality on MacOS, the following has to be done. 

- Open Mission Control by swiping up with three fingers, pressing `F3` or  `CMD` + `Arrow up`.
- Add new Spaces by clicking the "+" button at the top.
- Open the application you want to assign.
- Right-click its Dock icon, hover over "Options," and select “This Desktop” to keep it in that Space.
- Set custom keyboard shortcuts in **System Preferences > Keyboard > Shortcuts** under “Mission Control.”
	- Switch to Desktop 1
	- Switch to Desktop 2
	- ...
- Use your shortcuts to switch to your desired Desktop.

So far so good, until I had the following question.

'Looks cool, but how can I close my virtual desktops from this mission control screen?'

At this point I was already tired of MacOS (literally 15 minutes into the transition). You can't. Like seriously, you just can't. Unless of course you pull up your VISA card to download some external tooling that adds basic functionality. I am starting to see pattern with Microsoft here...

* <https://www.fadel.io/missioncontrolplus>

# Hold Key and Print Multiple Characters

I can't even believe I have to write this. If you have read my frustration above, I just needed to ventilate a bit. So I went to my terminal and started to type 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA' as that often helps to ease my frustration.

But guess what, Apple told me to go fuck myself and showed my some weird menu of special characters related to the key that I was holding down.

![[Pasted image 20240918111038.png]]

I already took out my VISA card and was ready to swipe it some more to unlock even more basic functionality, but it looks this time it was fixable with some configuration.

```bash
# Disable keyboard pressandhold
defaults write -g ApplePressAndHoldEnabled -bool false 
```

After logging out and in again, I was able to ventilate.

Reference: <https://macos-defaults.com/keyboard/applepressandholdenabled.html>

# What the Hell is a 'command' Key

> Why would we use USB ports if we can annoy people and use our custom 'Lightning' implementation. Heck, why should we even use the standard 'control' key? We can annoy our users even more by specifying our own standard. Let's create a 'command' key. Which is basically the same as 'control', but not really.
> 
> - Steve Jobs, when designing the MacOS system

Yeah, I am not going to even bother explaining but basically there is a 'command' key that takes over the 'control' key that you are so used to for working with computers in the last 30 years.

Either suck it up and get used to it, or whine about it and remap the 'command' key to the 'control' key.

I tried to adapt for 30 minutes, but since my warranty would not cover 'broken laptop after smashing against the wall' I had decided to simply rebind those keys.

* Navigate to **System Settings** > **Keyboard** > **Keyboard shortcuts** > **Modifier Keys** 
* Rebind the 'Control' key to 'Command'

![[Pasted image 20240918112216.png]]