WeakAuaras(WA) is an addon which provides a framework to create widgets and UI elements in World of Warcraft.

# Using a custom frame to hide weakauras

Personally I do not like my class weakauras to be visible whenever I am out of combat. However I do want them to show up if I have an enemy target selected.

One way to build this is via a custom frame which only activates when one of your provided conditionals are true. For example, have a valid target, be in a dungeon, cast a spell or be in combat. Set the parent frame of your weakauras to your custom frame and enable 'set parent to anchor'.

* Create a new aura of type `Empty Base Region`. Name it `MyAnchor`.
* Create a trigger for each condition you want.
	* Trigger 1: Player/Unit Info > Unit Characteristics > Unit = Target
	* Trigger 2: Player/Unit Info > Conditions > In Combat
* Set the activation to `Any Triggers` at the top of the triggers tab.
* Navigate to the `Group` tab of your class weakauras and the `Position and Size` settings.
	* Specify `Select frame` in the  `Anchored to` option and specify `WeakAuras:MyAnchor`.
	* Make sure `Set parent to Anchor` is ticked.
* Your class auras now only show up if any of the provided conditions are true.
