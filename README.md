# Sprig: Template functions for Go templates

The Go language comes with a [built-in template
language](http://golang.org/pkg/text/template/), but not
very many template functions. This library provides a group of commonly
used template functions.

It is inspired by the template functions found in
[Twig](http://twig.sensiolabs.org/documentation).

[![Build Status](https://travis-ci.org/Masterminds/sprig.svg?branch=master)](https://travis-ci.org/Masterminds/sprig)

## Usage

API documentation is available [at GoDoc.org](http://godoc.org/github.com/Masterminds/sprig), but
read on for standard usage.

### Load the Sprig library

To load the Sprig `FuncMap`:

```go

import (
  "github.com/Masterminds/sprig"
  "html/template"
)

// This example illustrates that the FuncMap *must* be set before the
// templates themselves are loaded.
tpl := template.Must(
  template.New("base").Funcs(sprig.FuncMap()).ParseGlob("*.html")
)


```

### Call the functions inside of templates

By convention, all functions are lowercase. This seems to follow the Go
idiom for template functions (as opposed to template methods, which are
TitleCase).


Example:

```
{{ "hello!" | upper | repeat 5 }}
```

Produces:

```
HELLO!HELLO!HELLO!HELLO!HELLO!
```

## Functions

### Date Functions

- date: Format a date, where a date is an integer type or a time.Time type, and
  format is a time.Format formatting string.
- date_modify: Given a date, modify it with a duration: `date_modify "-1.5h" now`. If the duration doesn't
parse, it returns the time unaltered. See `time.ParseDuration` for info on duration strings.
- now: Current time.Time, for feeding into date-related functions.

### String Functions

- trim: strings.TrimSpace
- trimall: strings.Trim, but with the argument order reversed `trimall "$" "$5.00"` or `"$5.00 | trimall "$"`
- upper: strings.ToUpper
- lower: strings.ToLower
- title: strings.Title
- repeat: strings.Repeat, but with the arguments switched: `repeat count str`. (This simplifies common pipelines)
- substr: Given string, start, and length, return a substr.
- nospace: Remove all spaces from a string. `h e l l o` becomes
  `hello`.
- abbrev: Truncate a string with ellipses
- abbrevboth: Truncate both sides of a string with ellipses
- untitle: Remove title case
- intials: Given multiple words, return the first letter of each
  word
- randAlphaNum: Generate a random alpha-numeric string
- randAlpha: Generate a random alphabetic string
- randAscii: Generate a random ASCII string, including symbols
- randNumeric: Generate a random numeric string
- wrap: Wrap text at the given column count
- wrapWith: Wrap text at the given column count, and with the given
  string for a line terminator: `wrap 50 "\n\t" $string`
- contains: strings.Contains, but with the arguments switched: `contains substr str`. (This simplifies common pipelines)
- quote: Wrap strings in double quotes. `quote "a" "b"` returns `"a"
  "b"`
- squote: Wrap strings in single quotes.

### String Slice Functions:

- join: strings.Join, but as `join SEP SLICE`
- split: strings.Split, but as `split SEP STRING`. The results are returned
  as a map with the indexes set to _N, where N is an integer starting from 0.
  Use it like this: `{{$v := "foo/bar/baz" | split "/"}}{{$v._0}}` (Prints `foo`)

### Conversions:

- atoi: Convert a string to an integer. 0 if the integer could not be parsed.

### Defaults:

- default: Give a default value. Used like this: {{trim "   "| default "empty"}}.
  Since trim produces an empty string, the default value is returned. For
  things with a length (strings, slices, maps), len(0) will trigger the default.
  For numbers, the value 0 will trigger the default. For booleans, false will
  trigger the default. For structs, the default is never returned (there is
  no clear empty condition). For everything else, nil value triggers a default.

### OS:

- env: Read an environment variable.
- expandenv: Expand all environment variables in a string.

### Encoding:

- b32enc: Encode a string into a Base32 string
- b32dec: Decode a string from a Base32 string
- b64enc: Encode a string into a Base64 string
- b64dec: Decode a string from a Base64 string

### Data Structures:

- tuple: A sequence of related objects. It is implemented as a
  `[]interface{}`, where each item can be accessed using `index`.

```
{{$t := tuple 1 "a" "foo"}}
{{index $t 2}}{{index $t 0 }}{{index $t 1}}
{{/* Prints foo1a *}}
```

### Reflection:

- typeOf: Takes an interface and returns a string representation of the type.
  For pointers, this will return a type prefixed with an asterisk(`*`). So
  a pointer to type `Foo` will be `*Foo`.
- typeIs: Compares an interface with a string name, and returns true if they match.
  Note that a pointer will not match a reference. For example `*Foo` will not
  match `Foo`.
- typeIsLike: returns true if the interface is of the given type, or
  is a pointer to the given type.
- kindOf: Takes an interface and returns a string representation of its kind.
- kindIs: Returns true if the given string matches the kind of the given interface.

	Note: None of these can test whether or not something implements a given
	interface, since doing so would require compiling the interface in ahead of
	time.


### Math Functions:

- add1: Increment an integer by 1
- add: Sum integers. `add 1 2 3` renders `6`
- sub: Subtract the second integer from the first
- div: Divide the first integer by the second
- mod: Module of first integer divided by second
- mul: Multiply two integers
- biggest: Return the biggest of a series of integers. `biggest 1 2 3`
  returns `3`.


## Principles:

The following principles were used in deciding on which functions to add, and
determining how to implement them.

- Template functions should be used to build layout. Therefore, the following
  types of operations are within the domain of template functions:
  - Formatting
  - Layout
  - Simple type conversions
  - Utilities that assist in handling common formatting and layout needs (e.g. arithmetic)
- Template functions should not return errors unless there is no way to print
  a sensible value. For example, converting a string to an integer should not
  produce an error if conversion fails. Instead, it should display a default
  value that can be displayed.
- Simple math is necessary for grid layouts, pagers, and so on. Complex math
  (anything other than arithmetic) should be done outside of templates.
- Template functions only deal with the data passed into them. They never retrieve
  data from a source.
- Finally, do not override core Go template functions.
