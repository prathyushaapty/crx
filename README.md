# crx [![Build Status](https://secure.travis-ci.org/oncletom/crx.svg)](http://travis-ci.org/oncletom/crx)

crx is a [pure node.js](http://nodejs.org/) command line app for packing Google Chrome extensions. **No OpenSSL required**!

If you'd like to integrate it into your [grunt](http://gruntjs.com/) workflow, give [grunt-crx](https://github.com/oncletom/grunt-crx) a spin.

Massive hat tip to the [node-rsa project](https://github.com/rzcoder/node-rsa)!

## Install

```bash
$ npm install crx
```

## Module API

Asynchronous functions returns an [ES6 Promise](https://github.com/jakearchibald/es6-promise).

### ChromeExtension = require("crx")
### crx = new ChromeExtension(attrs)

This module exports the `ChromeExtension` constructor directly, which can take an optional attribute object, which is used to extend the instance.

### crx.load(path)

Prepares the temporary workspace for the Chrome Extension located at `path`.

```js
crx.load('/path/to/extension').then(crx => {
  // ...
});
```

### crx.pack()

Packs the Chrome Extension and resolves the promise with a Buffer containing the `.crx` file.

```js
crx.load('/path/to/extension')
  .then(crx => crx.pack())
  .then(crxBuffer => {
    fs.writeFile('/tmp/foobar.crx', crxBuffer);
  });
```

### crx.generateUpdateXML()

Returns a Buffer containing the update.xml file used for `autoupdate`, as specified for `update_url` in the manifest. In this case, the instance must have a property called `codebase`.

```js
const crx = new ChromeExtension({ ..., codebase: 'https://autoupdateserver.com/myFirstExtension.crx' });

crx.load('/path/to/extension')
  .then(crx => crx.pack())
  .then(crxBuffer => {
    // ...
    const xmlBuffer = crx.generateUpdateXML();
    fs.writeFile('/foo/bar/update.xml', xmlBuffer);
  });
```

## Module example

```javascript
const fs = require("fs"),
const ChromeExtension = require("crx"),
const { join } = require("path"),
const crx = new ChromeExtension(
  codebase: "http://localhost:8000/myFirstExtension.crx",
  privateKey: fs.readFileSync(join(__dirname, "key.pem"))
});

crx.load(join(__dirname, "myFirstExtension"))
  .then(crx => crx.pack())
  .then(crxBuffer => {
    const updateXML = crx.generateUpdateXML()

    fs.writeFile(join(__dirname, "..", "update.xml"), updateXML);
    fs.writeFile(join(__dirname, "..", "myFirstExtension.crx"), crxBuffer);
  });
```

## CLI API

### crx pack [directory] [-o file] [--zip-output file] [-p private-key]

Pack the specified directory into a .crx package, and output it to stdout. If no directory is specified, the current working directory is used.

Use the `-o` option to write the signed extension to a file instead of stdout.

Use the `--zip-output` option to write the unsigned extension to a file.

Use the `-p` option to specify an external private key. If this is not used, `key.pem` is used from within the directory. If this option is not used and no `key.pem` file exists, one will be generated automatically.

Use the `-b` option to specify the maximum buffer allowed to generate extension. By default, will rely on `node` internal setting (~200KB).

### crx keygen [directory]

Generate a 2048-bit RSA private key within the directory. This is called automatically if a key is not specified, and `key.pem` does not exist.

Use the `--force` option to overwrite an existing private key located in the same given folder.

### crx -h

Show information about using this utility, generated by [commander](https://github.com/visionmedia/commander.js).

## CLI example

Given the following directory structure:

    └─┬ myFirstExtension
      ├── manifest.json
      └── icon.png

run this:

    cd myFirstExtension
    crx pack -o

to generate this:

    ├─┬ myFirstExtension
    │ ├── manifest.json
    │ ├── icon.png
    │ └── key.pem
    └── myFirstExtension.crx

You can also name the output file like this:

    cd myFirstExtension
    crx pack -o myFirstExtension.crx

to get the same results, or also pipe to the file manually like this.

    cd myFirstExtension
    crx pack > ../myFirstExtension.crx

As you can see a key is generated for you at `key.pem` if none exists. You can also specify an external key. So if you have this:

    ├─┬ myFirstExtension
    │ ├── manifest.json
    │ └── icon.png
    └── myPrivateKey.pem

you can run this:

    crx pack myFirstExtension -p myPrivateKey.pem -o

to sign your package without keeping the key in the directory.

Copyright
---------

    Copyright (c) 2014 Jed Schmidt, Thomas Parisot

    Permission is hereby granted, free of charge, to any person obtaining
    a copy of this software and associated documentation files (the
    "Software"), to deal in the Software without restriction, including
    without limitation the rights to use, copy, modify, merge, publish,
    distribute, sublicense, and/or sell copies of the Software, and to
    permit persons to whom the Software is furnished to do so, subject to
    the following conditions:

    The above copyright notice and this permission notice shall be
    included in all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
    EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
    MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
    NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
    LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
    OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
    WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
