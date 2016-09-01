---
layout: default
title: Home
---

The experience and subtlety of your program's text can be important. MessageFormat is a mechanism for handling both **pluralization** and **gender** in your applications. It can also lead to much better translations, as it's designed to support [all the languages](http://www.unicode.org/cldr/charts/latest/supplemental/language_plural_rules.html) included in the [Unicode CLDR](http://cldr.unicode.org/).

The ICU has an [official guide](http://userguide.icu-project.org/formatparse/messages) for the format. Messageformat.js supports and extends all parts of the [standard](http://icu-project.org/apiref/icu4j/com/ibm/icu/text/MessageFormat.html), with the exception of the deprecated ChoiceFormat.

There is a good slide-deck on [Plural and Gender in Translated Messages](https://docs.google.com/presentation/d/1ZyN8-0VXmod5hbHveq-M1AeQ61Ga3BmVuahZjbmbBxo/pub?start=false&loop=false&delayms=3000#slide=id.g1bc43a82_2_14) by Markus Scherer and Mark Davis. But, again, remember that many of these problems apply even if you're only outputting english.


## What problems does it solve?

Using messageformat.js, you can separate your code from your text formatting, while enabling much more humane expressions. In other words, you won't need to see this anymore in your output:

> There are 1 results.  
> There are 1 result(s).  
> Number of results: 5.


## What does it look like?

With this message:

```js
var msg =
  '{GENDER, select, male{He} female{She} other{They} }' +
  ' found ' +
  '{RES, plural, =0{no results} one{1 result} other{# results} }' +
  ' in the ' +
  '{CAT, selectordinal, one{#st} two{#nd} few{#rd} other{#th} }' +
  ' category.';
```

You'll get these results:

```js
var mfunc = new MessageFormat('en').compile(msg);

mfunc({ GENDER: 'male', RES: 1, CAT: 2 })
// 'He found 1 result in the 2nd category.'

mfunc({ GENDER: 'female', RES: 1, CAT: 2 })
// 'She found 1 result in the 2nd category.'

mfunc({ GENDER: 'male', RES: 2, CAT: 1 })
// 'He found 2 results in the 1st category.'

mfunc({ RES: 2, CAT: 2 })
// 'They found 2 results in the 2nd category.'
```


## Features

* Handles arbitrary nesting of pluralization and select rules
* Supports all ~466 languages included in the Unicode CLDR
* Works on the server and the client
* Remarkably useful even for single-language use
* Speed & efficiency: Can pre-compile messages to JavaScript code
  * Great for speed: message formatting is just string concatenation
* Compatible with other MessageFormat implementations
* Extendable with custom formatting functions
* Very whitespace tolerant
* Supports Unicode


## Installation

### Node
```
npm install messageformat
```

```js
var MessageFormat = require('messageformat');
var mf = new MessageFormat('en');
```

### Bower
```
bower install messageformat
```

```html
<script src="path/to/bower_components/messageformat/messageformat.js"></script>
<script>
  var mf = new MessageFormat('en');
</script>
```
