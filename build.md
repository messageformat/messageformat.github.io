---
layout: page
title: Build-time Compilation
---

For a significant decrease in filesize and execution time, you should precompile your messages to JavaScript during your build phase. It works like this:

```js
> var mf = new MessageFormat('en');
> var messages = {
    simple: 'A simple message.',
    var: 'Message with {X}.',
    plural: 'You have {N, plural, =0{no messages} one{1 message} other{# messages}}.',
    select: '{GENDER, select, male{He has} female{She has} other{They have}} sent you a message.',
    ordinal: 'The {N, selectordinal, one{1st} two{2nd} few{3rd} other{#th}} message.' };

> var mfunc = mf.compile(messages);
> mfunc().ordinal({N:3})
'The 3rd message.'

> console.log(mfunc.toString())
function anonymous() {
var number = function (value, offset) {
  if (isNaN(value)) throw new Error("'" + value + "' isn't a number.");
  return value - (offset || 0);
};
var plural = function (value, offset, lcfunc, data, isOrdinal) {
  if ({}.hasOwnProperty.call(data, value)) return data[value]();
  if (offset) value -= offset;
  var key = lcfunc(value, isOrdinal);
  if (key in data) return data[key]();
  return data.other();
};
var select = function (value, data) {
  if ({}.hasOwnProperty.call(data, value)) return data[value]();
  return data.other()
};
var pluralFuncs = {
  en: function (n, ord) {
    var s = String(n).split('.'), v0 = !s[1], t0 = Number(s[0]) == n,
        n10 = t0 && s[0].slice(-1), n100 = t0 && s[0].slice(-2);
    if (ord) return (n10 == 1 && n100 != 11) ? 'one'
        : (n10 == 2 && n100 != 12) ? 'two'
        : (n10 == 3 && n100 != 13) ? 'few'
        : 'other';
    return (n == 1 && v0) ? 'one' : 'other';
  }
};
var fmt = {};

return {
  simple: function(d) { return "A simple message."; },
  var: function(d) { return "Message with " + d.X + "."; },
  plural: function(d) { return "You have " + plural(d.N, 0, pluralFuncs.en, { 0: function() { return "no messages";}, one: function() { return "1 message";}, other: function() { return number(d.N) + " messages";} }) + "."; },
  select: function(d) { return select(d.GENDER, { male: function() { return "He has";}, female: function() { return "She has";}, other: function() { return "They have";} }) + " sent you a message."; },
  ordinal: function(d) { return "The " + plural(d.N, 0, pluralFuncs.en, { one: function() { return "1st";}, two: function() { return "2nd";}, few: function() { return "3rd";}, other: function() { return number(d.N) + "th";} }, 1) + " message."; }
}
}
```


## CLI Compiler

A [CLI compiler](https://github.com/SlexAxton/messageformat.js/tree/master/bin/messageformat.js) is also included, available as `./node_modules/.bin/messageformat` or just `messageformat` when installed with `npm install -g`.

```
$ messageformat --help
Usage: messageformat -l [locale] [OPTIONS] [INPUT_DIR] [OUTPUT_DIR]

Available options:
   -l,	--locale	locale(s) to use [mandatory]
   -i,	--inputdir	directory containing messageformat files to compile
   -o,	--output	output where messageformat will be compiled
   -ns,	--namespace	global object in the output containing the templates
   -I,	--include	glob patterns for files to include from `inputdir`
   -m,	--module	create a commonJS module, instead of a global window variable
   -s,	--stdout	print the result in stdout instead of writing in a file
   -w,	--watch  	watch `inputdir` for changes
   -v,	--verbose	print logs for debug
```

`messageformat` will recursively read all JSON files in `inputdir` and compile them to `output`.

When using the CLI, the following commands will each produce the same results as included in the [examples](https://github.com/SlexAxton/messageformat.js/tree/master/example/en):

```
$ messageformat --locale en --namespace i18n --inputdir ./example/en --output ./i18n.js
$ messageformat --locale en ./example/en ./i18n.js
$ messageformat -l en ./example/en
```


## Using compiled messageformat.js output

With default options, compiled messageformat functions are available through the `i18n` global object, with each source json having a corresponding subobject. Hence the compiled function corresponding to the `test` message defined in [`example/en/sub/folder/plural.json`](https://github.com/SlexAxton/messageformat.js/tree/master/example/en/sub/folder/plural.json) is available as [`i18n['sub/folder/plural'].plural`](https://github.com/SlexAxton/messageformat.js/tree/master/example/en/i18n.js):

```html
<script src="path/to/bower_components/messageformat/example/en/i18n.js"></script>
<script>
  console.log(i18n['sub/folder/plural'].plural({NUM: 3}));
</script>
```
will log `"Your 3 messages go here."`

A working example is available [here](https://github.com/SlexAxton/messageformat.js/tree/master/example).
