# ExtraCatalogs

![Maintenance Status](https://img.shields.io/badge/maintenance-active-green.svg)

Alternative MessageCatalog for Visualworks Smalltalk that supports
[XLIFF](http://en.wikipedia.org/wiki/XLIFF)-format files.

ExtraCatalogs is licensed under the MIT license.  See the Copyright
tab in the RB, the 'notice' property of this package, or the LICENSE
file on GitHub.

ExtraCatalogs' primary home is the
[Cincom Public Store Repository](http://www.cincomsmalltalk.com/CincomSmalltalkWiki/Public+Store+Repository).
Check there for the latest version.  It is also on
[GitHub](https://github.com/randycoulman/ExtraCatalogs).

ExtraCatalogs is compatible with VW 7.7 and later.  If you find any
incompatibilities with VW
7.7 or later, let me know (see below for contact information) or file
an issue on GitHub.

# Introduction

ExtraCatalogs provides `SimpleMessageCatalog`, an alternative
`MessageCatalog` for Visualworks Smalltalk.  A `SimpleMessageCatalog`
is an in-memory dictionary of translated strings that works with the
normal translation lookup in Visualworks.  It doesn't replace the
existing system; rather it works alongside it as an alternative method
of resolving `UserMessage`s.

ExtraCatalogs consists of three packages:

* ExtraCatalogs: The base package containing all you need to use
  `SimpleMessageCatalog`s in your application.

* ExtraCatalogsTools: Contains `UserMessageExtractor`, a handy tool
  for extracting `UserMessage`s from your application in order to
  generate an XLIFF file for your translators.

* ExtraCatalogsTests: The unit tests for ExtraCatalogs.

## Using Simple Message Catalogs

Assuming you have one or more XLIFF files containing the translations
for your application (see below), you can load the XLIFF file into a
`SimpleMessageCatalog`:

```
catalog := SimpleMessageCatalog fromXLIFF: xliffFilename
```

Once you've loaded a catalog, you then need to register it with a
Locale.  You can either register it with the current locale:

```
catalog register
```

or with a different locale:

```
catalog registerWith: anotherLocale
```

Once the catalog is registered, there is no need for your application
to retain a reference to it; it is maintained in a registry and will
now participate in `UserMessage` lookup using the normal Visualworks
mechanisms.

When your application closes, you can release all of the loaded
catalogs by sending:

```
SimpleMessageCatalog flush
```

## Generating Simple Message Catalogs

Where do you get an XLIFF file to start with?  Most translation
services work with this format, so it is possible that you can already
get one from your translator.

If not, `UserMessageExtractor` from ExtraCatalogsTools can provide
some rudimentary help.  Using the built-in facilities in Visualworks,
iterate all of the methods in your application that might contain
`UserMessage`s that need to be translated.

One way to do this is to start with `SystemUtils>>allBehaviorsDo:` and
filter the results based on whether the class might contain
`UserMessage`s that you care about.

For each method:

```
tree := aCompiledMethod mclass parseTreeFor: aCompiledMethod selector.
visitor := UserMessageExtractor new.
[visitor visitNode: tree] on: InformationSignal
    do: [:ex | "Found a UserMessage that couldn't be extracted"
               ex resume].
^visitor userMessages
```

Once you've collected all of the `UserMessage`s, you can write an XLIFF
file containing them:

```
catalog := (SimpleMessageCatalog id: #myCatalogID)
    addAll: collectedUserMessages;
    yourself.

catalog writeXLIFFFile
```

This will create a file named `myCatalogID.xliff`.  You can use
`writeXLIFFFile:` and provide your own filename if you prefer.

Once you have your XLIFF file(s), you can send them to your translator
to be translated.  When you get them back, include them with your
application and you're ready to go.

## Maintaining Simple Message Catalogs

`SimpleMessageCatalog` contains methods that are useful in maintaining
message catalogs as they evolve over time.

* `additions: aCatalog`: Assume `aCatalog` is an updated version of
  `self` and answer a new catalog that contains any new
  `UserMessage`s.

* `removals: aCatalog`: Assume `aCatalog` is an updated version of
  `self` and answer a new catalog that contains any removed
  `UserMessage`s.

* `conflicts: aCatalog`: Assume `aCatalog` is an updated version of
  `self` and answer a new catalog that contains any `UserMessage`s
  that have conflicting default strings.

* `upgrade: aCatalog`: Add or update all `UserMessage`s in `aCatalog`
  to `self`.

Using these messages, it is possible to generate a new version of a
catalog, create a new "delta" catalog containing the additions and conflicts
between the new catalog and the previous version.  This delta catalog
is usually much smaller than the original and can be translated more
inexpensively.  When the translated delta catalog is received back
from the translator, the new catalog can be upgraded with the
translated delta catalog.

## Understanding the Code

The two main classes of interest are `SimpleMessageCatalog` and
`UserMessageExtractor`.  Start from the class comments of those
classes.

# Contributing

I'm happy to receive bug fixes and improvements to this package.  If
you'd like to contribute, please publish your changes as a "branch"
(non-integer) version in the Public Store Repository and contact me as
outlined below to let me know.  I will merge your changes back into
the "trunk" as soon as I can review them.

# Credits

ExtraCatalogs was originally written by Travis Griggs, who has
graciously allowed me to take over maintenance of this package.

# Contact Information

If you have any questions about this package and how to use it, feel free to contact me.

* Web site: http://randycoulman.com
* Blog: Courageous Software (http://randycoulman.com/blog)
* E-mail: randy _at_ randycoulman _dot_ com
* Twitter: @randycoulman
* GitHub: randycoulman
