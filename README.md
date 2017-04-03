# api documentation for  [ecstatic (v2.1.0)](https://github.com/jfhbrook/node-ecstatic)  [![npm package](https://img.shields.io/npm/v/npmdoc-ecstatic.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-ecstatic) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-ecstatic.svg)](https://travis-ci.org/npmdoc/node-npmdoc-ecstatic)
#### A simple static file server middleware that works with both Express and Flatiron

[![NPM](https://nodei.co/npm/ecstatic.png?downloads=true)](https://www.npmjs.com/package/ecstatic)

[![apidoc](https://npmdoc.github.io/node-npmdoc-ecstatic/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-ecstatic_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-ecstatic/build..beta..travis-ci.org/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-ecstatic/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-ecstatic/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Joshua Holbrook",
        "email": "josh@nodejitsu.com",
        "url": "http://jesusabdullah.net"
    },
    "bin": {
        "ecstatic": "./lib/ecstatic.js"
    },
    "bugs": {
        "url": "https://github.com/jfhbrook/node-ecstatic/issues"
    },
    "dependencies": {
        "he": "^0.5.0",
        "mime": "^1.2.11",
        "minimist": "^1.1.0",
        "url-join": "^1.0.0"
    },
    "description": "A simple static file server middleware that works with both Express and Flatiron",
    "devDependencies": {
        "codecov": "^1.0.1",
        "eol": "^0.2.0",
        "express": "^4.12.3",
        "mkdirp": "^0.5.0",
        "request": "^2.49.0",
        "tap": "^5.7.0"
    },
    "directories": {},
    "dist": {
        "shasum": "477a4a6b25cecb112f697dbd2c7fa354154ea6be",
        "tarball": "https://registry.npmjs.org/ecstatic/-/ecstatic-2.1.0.tgz"
    },
    "gitHead": "a5e646b1122e51d715d03f3373a3ff99de84a9fc",
    "homepage": "https://github.com/jfhbrook/node-ecstatic",
    "keywords": [
        "static",
        "web",
        "server",
        "files",
        "mime",
        "middleware"
    ],
    "license": "MIT",
    "main": "./lib/ecstatic.js",
    "maintainers": [
        {
            "name": "jesusabdullah",
            "email": "josh.holbrook@gmail.com"
        },
        {
            "name": "jfhbrook",
            "email": "josh.holbrook@gmail.com"
        }
    ],
    "name": "ecstatic",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+ssh://git@github.com/jfhbrook/node-ecstatic.git"
    },
    "scripts": {
        "posttest": "tap --coverage-report=text-lcov | codecov",
        "test": "tap --coverage test/*.js"
    },
    "version": "2.1.0"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module ecstatic](#apidoc.module.ecstatic)
1.  [function <span class="apidocSignatureSpan">ecstatic.</span>showDir (opts, stat)](#apidoc.element.ecstatic.showDir)
1.  string <span class="apidocSignatureSpan">ecstatic.</span>version



# <a name="apidoc.module.ecstatic"></a>[module ecstatic](#apidoc.module.ecstatic)

#### <a name="apidoc.element.ecstatic.showDir"></a>[function <span class="apidocSignatureSpan">ecstatic.</span>showDir (opts, stat)](#apidoc.element.ecstatic.showDir)
- description and source-code
```javascript
showDir = function (opts, stat) {
  // opts are parsed by opts.js, defaults already applied
  var cache = opts.cache,
      root = path.resolve(opts.root),
      baseDir = opts.baseDir,
      humanReadable = opts.humanReadable,
      handleError = opts.handleError,
      showDotfiles = opts.showDotfiles,
      si = opts.si,
      weakEtags = opts.weakEtags;

  return function middleware (req, res, next) {

    // Figure out the path for the file from the given url
    var parsed = url.parse(req.url),
        pathname = decodeURIComponent(parsed.pathname),
        dir = path.normalize(
          path.join(root,
            path.relative(
              path.join('/', baseDir),
              pathname
            )
          )
        );

    fs.stat(dir, function (err, stat) {
      if (err) {
        return handleError ? status[500](res, next, { error: err }) : next();
      }

      // files are the listing of dir
      fs.readdir(dir, function (err, files) {
        if (err) {
          return handleError ? status[500](res, next, { error: err }) : next();
        }

        // Optionally exclude dotfiles from directory listing.
        if (!showDotfiles) {
          files = files.filter(function(filename){
            return filename.slice(0,1) !== '.';
          });
        }

        res.setHeader('content-type', 'text/html');
        res.setHeader('etag', etag(stat, weakEtags));
        res.setHeader('last-modified', (new Date(stat.mtime)).toUTCString());
        res.setHeader('cache-control', cache);

        sortFiles(dir, files, function (lolwuts, dirs, files) {
          // It's possible to get stat errors for all sorts of reasons here.
          // Unfortunately, our two choices are to either bail completely,
          // or just truck along as though everything's cool. In this case,
          // I decided to just tack them on as "??!?" items along with dirs
          // and files.
          //
          // Whatever.

          // if it makes sense to, add a .. link
          if (path.resolve(dir, '..').slice(0, root.length) == root) {
            return fs.stat(path.join(dir, '..'), function (err, s) {
              if (err) {
                return handleError ? status[500](res, next, { error: err }) : next();
              }
              dirs.unshift([ '..', s ]);
              render(dirs, files, lolwuts);
            });
          }
          render(dirs, files, lolwuts);
        });

        function render(dirs, files, lolwuts) {
          // each entry in the array is a [name, stat] tuple

          var html = [
            '<!doctype html>',
            '<html>',
            '  <head>',
            '    <meta charset="utf-8">',
            '    <meta name="viewport" content="width=device-width">',
            '    <title>Index of ' + he.encode(pathname) +'</title>',
            '    <style type="text/css">' + css + '</style>',
            '  </head>',
            '  <body>',
            '<h1>Index of ' + he.encode(pathname) + '</h1>'
          ].join('\n') + '\n';

          html += '<table>';

          var failed = false;
          var writeRow = function (file, i) {
            // render a row given a [name, stat] tuple
            var isDir = file[1].isDirectory && file[1].isDirectory();
            var href = parsed.pathname.replace(/\/$/, '') + '/' + encodeURIComponent(file[0]);

            // append trailing slash and query for dir entry
            if (isDir) {
              href += '/' + he.encode((parsed.search)? parsed.search:'');
            }

            var displayName = he.encode(file[0]) + ((isDir)? '/':'');

            var ext = file[0].split('.').pop();

            var iconClass = 'icon-' + (isDir ? '_blank' : (supportedIcons[ext] ? ext : '_page'));

            // TODO: use stylessheets?
            html += '<tr>' +
              '<td class="icon-parent"><i class="' + iconClass + '"></i></td>' +
              '<td class="perms"><code>(' + permsToString(file[1]) + ')</code></td>' +
              '<td class="file-size"><code>' + sizeToString(file[1], humanReadable, si) + '</code></td>' + ...
```
- example usage
```shell
...

Defaults to **false**.

## middleware(req, res, next);

This works more or less as you'd expect.

### ecstatic.showDir(folder);

This returns another middleware which will attempt to show a directory view. Turning on auto-indexing is roughly equivalent to adding
 this middleware after an ecstatic middleware with autoindexing disabled.

### 'ecstatic' command

to start a standalone static http server,
run 'npm install -g ecstatic' and then run 'ecstatic [dir?] [options] --port PORT'
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
