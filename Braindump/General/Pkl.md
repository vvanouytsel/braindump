#language

Pkl (pronounced as 'pickle') is a configuration language that aims to be programmable, scalable and safe.

Reference: [Basic Configuration :: Pkl Docs](https://pkl-lang.org/main/current/language-tutorial/01_basic_config.html)

## VSCode

A VSCode extension for `pkl` exists but is not available via the official repositories. You can install it by downloading it via GitHub.

* Download the latest release `vsix` file.
* Install the extension via the CLI.

```bash
code --install-extension ~/Downloads/pkl-vscode-0.17.0.vsix
```

## Commands

Evaluate a `pkl` file.

```
pkl eval myfile.pkl

# Print in json format
pkl eval -f json myfile.pkl
```

## Structure

Typically a configuration contains a hierarchical structure. Pkl provides immutable objects for this purpose.

An object has three kinds of members:
* Properties
* Elements
* Entries

### Properties

```pkl
bird {  #1
  name = "Common wood pigeon" #2
  diet = "Seeds"
  taxonomy { #3
    species = "Columba palumbus"
  }
}
```

| Item | Description                                                        |
| ---- | ------------------------------------------------------------------ |
| 1    | This _defines_ `bird` to be an object                              |
| 2    | For primitive values, Pkl has the `=` syntax (more on this later). |
| 3    | Just like `bird {`, but to show that objects can be nested.        |

### Elements

You can think of an object that only contains elements as an array. Elements are indexed by an integer. To access an element, you use brackets. For example, myObject[42].

```pkl
exampleObjectWithJustIntElements {
  100 #1
  42
}

exampleObjectWithMixedElements {
  "Bird Breeder Conference"
  (2000 + 23) #2
  exampleObjectWithJustIntElements #3
}
```

| Item | Description                                                                    |
| ---- | ------------------------------------------------------------------------------ |
| 1    | When you write only the value (without a name), you describe an _element_.     |
| 2    | Elements don’t have to be literal values; they can be arbitrary _expressions_. |
| 3    | Elements can really be _any_ value, not just primitive values.                 |

### Entries

An entry is similar to a property as they are both 'named'. However for an entry the name (key) does not need to be known at declaration time. The syntax used for the key of an entry is brackets with a value in the middle.  For example `["something"]`, or `[anotherObject]`. As you can see, names do not have to be strings.

```pkl
pigeonShelter {
  ["bird"] { #1
    name = "Common wood pigeon"
    diet = "Seeds"
    taxonomy {
      species = "Columba palumbus"
    }
  }
  ["address"] = "355 Bird St." #2
}

birdCount {
  [pigeonShelter] = 42 #3
}
```

| Item | Description                                                                  |
| ---- | ---------------------------------------------------------------------------- |
| 1    | The difference with properties is the notation of the key: `[<expression>]`. |
| 2    | As with properties, entries can be primitive values or objects.              |
| 3    | Any object can be used as a key for an entry.                                |

## Collections

You can mix and match all of the above members of an object. However this can quickly become confusing and could provide issues when using target formatters, as they are more restrictive.

```pkl
mixedObject {
  name = "Pigeon"
  lifespan = 8
  "wing"
  "claw"
  ["wing"] = "Not related to the _element_ \"wing\""
  42
  extinct = false
  [false] {
    description = "Construed object example"
  }
}
```

Evaluating the above object with the json formatter will result in errors.

```bash
$ pkl eval -f json /Users/me/tutorial/mixedObject.pkl
–– Pkl Error ––
Cannot render object with both properties/entries and elements as JSON.
Object: "Pigeon"

89 | text = renderer.renderDocument(value)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
at pkl.base#Module.output.text (https://github.com/apple/pkl/blob/0.24.0/stdlib/base.pkl#L90)
```

To work around that Pkl has two special types of objects:
* **Listings**: these objects exclusively contain elements
* **Mappings**: these objects exclusively contain entries

These special types are simply just objects and do not require custom syntax.

```pkl
birds { #1
  "Pigeon"
  "Parrot"
  "Barn owl"
  "Falcon"
}

habitats { #2
  ["Pigeon"] = "Streets"
  ["Parrot"] = "Parks"
  ["Barn owl"] = "Forests"
  ["Falcon"] = "Mountains"
}
```

| Item | Description                         |
| ---- | ----------------------------------- |
| 1    | A listing containing four elements. |
| 2    | A mapping containing four entries.  |

You can reference the values of your properties, elements and entries using the correct syntax.

```pkl
birds { 
  "Pigeon"
  "Parrot"
  "Barn owl"
  "Falcon"
}

habitats { 
  ["Pigeon"] = "Streets"
  ["Parrot"] = "Parks"
  ["Barn owl"] = "Forests"
  ["Falcon"] = "Mountains"
}

collar {
  color = "green" // pun intended!
  size = "large"
}

awesomeBird = birds[3] // elements
awesomeHabitat = habitats["Falcon"] // entries
awesomeCollar = collar.color // properties

```