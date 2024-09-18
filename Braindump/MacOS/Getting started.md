#mac #os

As I am transitioning from Fedora to MacOS, some of the pain points will be written down here.

> [!info]
> The MacOS specific 'command' key is referenced as ⌘.

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
* Rebind the 'Command' key to 'Control'
* Be happy that you can now copy via 'Control' + 'c'
* Be frustrated that you now have to use 'Command' + 'c' if you want to mimic the 'Control' + 'c' functionality in your terminal
* Revert your changes and just deal with the 'Command' bullshit and get used to it

![[Pasted image 20240918112216.png]]

# My Changes to Configuration File for Application X Are Ignored

No, it is not ignored. Let's call it a feature and not a bug. If you press the the big CROSS icon on the left of your application window, you would assume that you are closing the application. But where is the fun in that, it is too logical so it is a no-go for MacOS.

MacOS prefers to minimize your application to the tray if you click the CLOSE button. Why? Because why not? The more complex, the better, right Siri?

So nope, your application is not ignoring your changes in your configuration files. You are just not restarting your application...

You can close an application using '⌘' + 'q'.

# Why Are My HOME and END Keys not Working?

Why would you use a single key to move to the beginning or end of a line? That is way to logical and thus by design a no-go for MacOS.

You know what would be way more illogical? To use TWO keys instead of one. You know what would even be more stupid? TO use TWO keys on the complete opposite of the keyboard. That way we annoy our users the most. Fantastic idea, let's implement that!

In MacOS we use the ⌘ key combination with the arrow keys.

**⌘ + ←**: go to beginning of line (HOME)
**⌘ + →**: go to end of line (END)

If you really want to you can tinker with a file named `DefaultKeyBinding.dict`. But there is even a better way! Stop using HOME or END and use 'ctrl' + 'a' and 'ctrl' + 'e' instead.

Reference: <https://www.reddit.com/r/MacOS/comments/pz9vnu/behavior_of_the_home_and_end_keys/>