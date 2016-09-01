---
layout: page
title: Build-time Compilation
---

For a significant decrease in filesize and execution time, you should precompile your messages to JavaScript during your build phase. It works like this:

```js
var mf = new MessageFormat('en');
var messages = {
  simple: 'A simple message.',
  var: 'Message with {X}.',
  plural: 'You have {N, plural, =0{no messages} one{1 message} other{# messages}}.',
  select: '{GENDER, select, male{He has} female{She has} other{They have}} sent you a message.',
  ordinal: 'The {N, selectordinal, one{1st} two{2nd} few{3rd} other{#th}} message.'
};

var mfunc = mf.compile(messages);
mfunc().ordinal({ N: 1 })
  // "The 1st message."

var efunc = new Function('return (' + mfunc.toString() + ')()');

efunc();
// { simple: [Function],
//   var: [Function],
//   plural: [Function],
//   select: [Function],
//   ordinal: [Function] }

efunc().ordinal({ N: 2 });
  // "The 2nd message."
```

Note that as `efunc` is defined as a `new Function()`, it has no access to the surrounding scope. This means that the output of `mfunc().toString()` can be saved as a file and later included with `require()` or `<script src=...>`, providing access to the compiled functions completely independently of messageformat.js, or any other dependencies.


## CLI Compiler

A [CLI compiler](https://github.com/messageformat/messageformat.js/tree/master/bin/messageformat.js) is also included, available as `./node_modules/.bin/messageformat` or just `messageformat` when installed with `npm install -g`.

```text
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

When using the CLI, the following commands will each produce the same results as included in the [examples](https://github.com/messageformat/messageformat.js/tree/master/example/en):

```
$ messageformat --locale en --namespace i18n --inputdir ./example/en --output ./i18n.js
$ messageformat --locale en ./example/en ./i18n.js
$ messageformat -l en ./example/en
```


## Using compiled messageformat.js output

With default options, compiled messageformat functions are available through the `i18n` global object, with each source json having a corresponding subobject. Hence the compiled function corresponding to the `test` message defined in [`example/en/sub/folder/plural.json`](https://github.com/messageformat/messageformat.js/tree/master/example/en/sub/folder/plural.json) is available as [`i18n['sub/folder/plural'].plural`](https://github.com/messageformat/messageformat.js/tree/master/example/en/i18n.js):

```html
<script src="path/to/messageformat/example/en/i18n.js"></script>
<script>
  console.log(i18n['sub/folder/plural'].plural({NUM: 3}));
</script>
```
will log `"Your 3 messages go here."`

A working example is available [here](/messageformat.js/example/index.html).
