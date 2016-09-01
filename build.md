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
$ messageformat
usage: messageformat [-l lc] [-n ns] [-p] input

Parses the input JSON file(s) of MessageFormat strings into a JS module of
corresponding hierarchical functions, written to stdout. Directories are
recursively scanned for all .json files.

  -l lc, --locale=lc
        The locale(s) lc to include; if multiple, selected by matching
        message key. [default: en]

  -n ns, --namespace=ns
        The global object or modules format for the output JS. If ns does not
        contain a '.', the output follows an UMD pattern. For module support,
        the values 'export default' (ES6), 'exports' (CommonJS), and
        'module.exports' (node.js) are special. [default: module.exports]

  -p, --disable-plural-key-checks
        By default, messageformat.js throws an error when a statement uses a
        non-numerical key that will never be matched as a pluralization
        category for the current locale. Use this argument to disable the
        validation and allow unused plural keys. [default: false]
```


## Using compiled messageformat.js output

With default options, compiled messageformat functions are available through `module.exports`. However, using e.g. `-n i18n`, an UMD pattern is used, falling back to a global `i18n` object, with each source json having a corresponding subobject. Hence the compiled function corresponding to the `test` message defined in [`example/en/sub/folder/plural.json`](https://github.com/messageformat/messageformat.js/tree/master/example/en/sub/folder/plural.json) is available as [`i18n.sub.folder.plural.test`](https://github.com/messageformat/messageformat.js/tree/master/example/en/i18n.js):

```html
<script src="path/to/messageformat/example/en/i18n.js"></script>
<script>
  console.log(i18n.sub.folder.plural.test({ NUM: 3 }));
</script>
```
will log `"Your 3 messages go here."`

A working example is available [here](/messageformat.js/example/index.html).
