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

## Amending

Expressing a part of the configuration in ters of another is called amending.

```pkl
bird {
  name = "Pigeon"
  diet = "Seeds"
  taxonomy {
    kingdom = "Animalia"
    clade = "Dinosauria"
    order = "Columbiformes"
  }
}

parrot = (bird) {
  name = "Parrot"
  diet = "Berries"
  taxonomy {
    order = "Psittaciformes"
  }
}
```

The `bird` object is being amended in this example and leads to the output below.

```pkl
bird {
  name = "Pigeon"
  diet = "Seeds"
  taxonomy {
    kingdom = "Animalia"
    clade = "Dinosauria"
    order = "Columbiformes"
  }
}
parrot {
  name = "Parrot"
  diet = "Berries"
  taxonomy {
    kingdom = "Animalia"
    clade = "Dinosauria"
    order = "Psittaciformes"
  }
}
```

## Modules

A `pkl` file describes a module. Modules are objects that can be referred to from other modules.

```pkl
// pigeon.pkl
name = "Common wood pigeon"
diet = "Seeds"
taxonomy {
  kingdom = "Animalia"
  clade = "Dinosauria"
  species = "Columba palumbus"
}
```

The module above can be imported using the `import` statement.

```pkl
// parrot.pkl
import "pigeon.pkl" 

parrot = (pigeon) {
  name = "Great green macaw"
  diet = "Berries"
  taxonomy {
    species = "Ara ambiguus"
  }
}
```

If you run `Pkl` on `parrot.pkl` you will notice that the object `parrot` is printed as well.

```pkl
❯ pkl eval parrot.pkl
parrot {
  name = "Great green macaw"
  diet = "Berries"
  taxonomy {
    kingdom = "Animalia"
    clade = "Dinosauria"
    species = "Ara ambiguus"
  }
}
```

If you want to define that the module itself is an object, which is amended from the `pigeon.pkl` module, you use the `amends` keyword.

```pkl
// dog.pkl
amends "pigeon.pkl"
diet = "Meat"
taxonomy {
  kingdom = "Dogania"
}
```

By using `amends` the `pigeon` amended module is no longer printed.

```
❯ pkl eval dog.pkl
name = "Common wood pigeon"
diet = "Meat"
taxonomy {
  kingdom = "Dogania"
  clade = "Dinosauria"
  species = "Columba palumbus"
}
```

## Templates

A `Pkl` file is either a  'normal' module or a template. The only difference between them is the intended use of the module.

### Types

`Pkl` supports the following basic types:

```
name: String = "Writing a Template"
part: Int = 3
hasExercises: Boolean = true
amountLearned: Float = 13.37
duration: Duration = 30.min
bandwidthRequirementPerSecond: DataSize = 52.4288.mb
```

By default the output of `Pkl` is `pcf` which is a subset of `Pkl`. Because `pcf` does not have type signatures, running `Pkl` against this module removes the type signatures in the output.

### Typed objects

You can define types for objects by creating classes.

```pkl
class Language { // 1
  name: String = "ThisIsADefaultValue"
}

bestForConfig: Language = new { // 2
  name = "Pkl"
}
```

| Item | Description                                        |
| ---- | -------------------------------------------------- |
| 1    | A class definition.                                |
| 2    | A property definition, using the `Language` class. |

### Creating a template

The easiest approach is to write your configuration in a `Pkl` file. Once that is done, you can create your template based on your configuration.

Let' say that this is your typical configuration you want to create a template for.

```pkl
// workshop2024.pkl
title = "Pkl: Configure your Systems in New Ways"
interactive = true
seats = 100
occupancy = 0.85
duration = 1.5.h
`abstract` = """
  With more systems to configure, the software industry is drowning in repetitive and brittle configuration files.
  YAML and other configuration formats have been turned into programming languages against their will.
  Unsurprisingly, they don’t live up to the task.
  Pkl puts you back in control.
  """

event {
  name = "Migrating Birds between hemispheres"
  year = 2024
}

instructors {
  "Kate Sparrow"
  "Jerome Owl"
}

sessions {
  new {
    date = "2/1/2024"
    time = 30.min
  }
  new {
    date = "2/1/2024"
    time = 30.min
  }
}

assistants {
  ["kevin"] = "Kevin Parrot"
  ["betty"] = "Betty Harrier"
}

agenda {
  ["beginners"] {
    name = "Basic Configuration"
    part = 1
    duration = 45.min
  }
  ["intermediates"] {
    name = "Filling out a Template"
    part = 2
    duration = 45.min
  }
  ["experts"] {
    name = "Writing a Template"
    part = 3
    duration = 45.min
  }
}
```

For best practices you should start your template with a capital letter.

```pkl
// Workshop.pkl
module Workshop

// You can create the class directly in the file
// But this shows you that you can also import it into a template.
import "TutorialPart.pkl"

title: String
interactive: Boolean
seats: Int
occupancy: Float
duration: Duration
`abstract`: String

// event is an object
// we create a class for it allowing us to type it properly
class Event {
  name: String
  year: Int
}

event: Event
instructors: Listing<String>

class Session {
  time: Duration
  date: String
}

sessions: Listing<Session>
assistants: Mapping<String, String>

agenda: Mapping<String, TutorialPart>
```

```pkl
// TutorialPart.pkl
class TutorialPart {
  name: String
  part: Int
  hasExercises: Boolean
  amountLearned: Float
  duration: Duration
  bandwidthRequirementPerSecond: DataSize
}
```