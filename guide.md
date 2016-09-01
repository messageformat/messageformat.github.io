---
layout: page
title: Format Guide
---

## Contents
* [Variables](#variables)
* [SelectFormat](#selectFormat)
* [PluralFormat](#pluralFormat)
* [Nesting](#nesting)
* [Escaping](#escaping)


The simplest case of MessageFormat involves no formatting, just a string passthrough. This may sound silly, but often it's nice to always use the same message formatting system when doing translations, and not everything requires variables.

```javascript
var mf = new MessageFormat('en');
var message = mf.compile('This is a message.');

message();
  // "This is a message."
```

NOTE: if a message _does_ require data to be passed in, an error is thrown if you do not.


## Variables

The second most simple way to use MessageFormat is for simple variable replacement. MessageFormat may look odd at first, but it's actually fairly simple. One way to think about the `{` and `}` is that every level of them bring you into and out-of `literal` and `code` mode.

By default (like in the previous example), you are just writing a literal. Then the first level of brackets brings you into one of several data-driven situations. The most simple is variable replacement.

Simply putting a variable name in between `{` and `}` will place that variable there in the output.

```javascript
var mf = new MessageFormat('en');
var varMessage = mf.compile('His name is {NAME}.');

varMessage({ NAME : "Jed" });
  // "His name is Jed."
```


## SelectFormat

`SelectFormat` is a lot like a switch statement for your messages. Often it's used to select gender in a string. The format of the statement is `{varname, select, thisvalue{...} thatvalue{...} other{...}}`, where `varname` matches a key in the data you give to the resulting function, and `'thisvalue'` & `'thatvalue'` are some of the string-equivalent values that it may have. The `other` key is required, and is selected if no other case matches.

Note that comparison is made using the JavaScript `==` operator, so if a key is left out of the input data, the case `undefined{...}` would match that.

```javascript
var mf = new MesssageFormat('en');
var selectMessage = mf.compile(
  '{GENDER, select, male{He} female{She} other{They}} liked this.'
);

selectMessage({ GENDER: 'male' });
  // "He liked this."

selectMessage({ GENDER: 'female' });
  // "She liked this."

selectMessage({});
  // "They liked this."
```


## PluralFormat

`PluralFormat` is a similar mechanism to `SelectFormat`, but specific to numerical values. The key that is chosen is generated from the specified input variable by a locale-specific _plural function_.

The numeric input is mapped to a plural category, some subset of `zero`, `one`, `two`, `few`, `many`, and `other` depending on the locale and the type of plural. English, for instance, uses `one` and `other` for cardinal plurals (one result, many results) and `one`, `two`, `few`, and `other` for ordinal plurals (1st result, 2nd result, etc). For information on which keys are used by your locale, please refer to the [CLDR table of plural rules](http://www.unicode.org/cldr/charts/latest/supplemental/language_plural_rules.html).

Matches for exact values are available with the `=` prefix.

The keyword for cardinal plurals is `plural`, and for ordinal plurals is `selectordinal`.

Within a plural statement, `#` will be replaced by the variable value.

```javascript
var mf = new MessageFormat('en');
var pluralMessage = mf.compile(
  'There {NUM_RESULTS, plural, =0{are no results} one{is one result} other{are # results}}.'
);

pluralMessage({ NUM_RESULTS: 0 });
  // "There are no results."

pluralMessage({ NUM_RESULTS: 1 });
  // "There is one result."

pluralMessage({ NUM_RESULTS: 100 });
  // "There are 100 results."
```


#### Offset extension

To generate sentences such as "You and 4 others added this to their profiles.", PluralFormat supports adding an `offset` to the variable value before determining its plural category. Literal/exact matches are tested before applying the offset.

```javascript
var mf = new MessageFormat('en');

var offsetMessage = mf.compile(
  'You {NUM_ADDS, plural, offset:1' +
    '=0{did not add this}' +
    '=1{added this}' +
    'one{and one other person added this}' +
    'other{and # others added this}' +
  '}.'
);

offsetMessage({ NUM_ADDS: 0 });
  // "You did not add this."

offsetMessage({ NUM_ADDS: 1 });
  // "You added this."

offsetMessage({ NUM_ADDS: 2 });
  // "You and one other person added this."

offsetMessage({ NUM_ADDS: 3 });
  // "You and 2 others added this."
```


## Nesting

All types of messageformat statements may be nested within each other, to unlimited depth:

```text
{SEL1, select,
  other {
    {PLUR1, plural,
      one {1}
      other {
        {SEL2, select,
          other {Deep in the heart.}
        }
      }
    }
  }
}
```


## Escaping

The characters `{` and `}` must be escaped with a `\` to be included in the output as literal characters. Within plural statements, `#` must also be similarly escaped. Keep in mind that you'll need to double-escape with `\\` within e.g. JavaScript and JSON strings.

```javascript
var mf = new MessageFormat('en');
var escMessage = mf.compile('\\{ {S, plural, other{# is a \\#}} \\}');

escMessage({ S: 5 });
  // "{ 5 is a # }"
```
