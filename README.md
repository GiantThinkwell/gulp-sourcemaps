## gulp-sourcemaps  [![NPM version][npm-image]][npm-url] [![build status][travis-image]][travis-url] [![Test coverage][coveralls-image]][coveralls-url]

### Usage

#### Inline maps
Inline maps are embedded in the source file.

```javascript
var gulp = require('gulp');
var concat = require('gulp-concat');
var uglify = require('gulp-uglify');
var sourcemaps = require('gulp-sourcemaps');

gulp.task('javascript', function() {
  gulp.src('src/**/*.js')
    .pipe(sourcemaps.init())
      .pipe(concat('all.js'))
      .pipe(uglify())
    .pipe(sourcemaps.write())
    .pipe(gulp.dest('dist'));
});
```

All plugins between `sourcemaps.init()` and `sourcemaps.write()` need to support source maps.

Warning: the current NPM version of `gulp-concat` doesn't support source maps yet. You can use https://github.com/floridoo/gulp-concat/tree/sourcemap_pipe2 for now.

#### External source map files

To write external source map files, pass a path relative to the destination to `sourcemaps.write()`.

Example:
```javascript
var gulp = require('gulp');
var concat = require('gulp-concat');
var uglify = require('gulp-uglify');
var sourcemaps = require('gulp-sourcemaps');

gulp.task('javascript', function() {
  gulp.src('src/**/*.js')
    .pipe(sourcemaps.init())
      .pipe(concat('all.js'))
      .pipe(uglify())
    .pipe(sourcemaps.write('../maps'))
    .pipe(gulp.dest('dist'));
});
```


### Options

- `addComment`

  By default a comment containing / referencing the source map is added. Set this to `false` to disable the comment (e.g. if you want to load the source maps by header).

  Example:
  ```javascript
  gulp.task('javascript', function() {
    var stream = gulp.src('src/**/*.js')
      .pipe(sourcemaps.init())
        .pipe(concat('all.js'))
        .pipe(uglify())
      .pipe(sourcemaps.write('../maps', {addComment: false}))
      .pipe(gulp.dest('dist'));
  });
  ```

- `includeContent`

  By default the source maps include the source code. Pass `false` to use the original files.

- `sourceRoot`

  Set the path where the source files are hosted (use this when `includeContent` is set to `false`).

  Example:
  ```javascript
  gulp.task('javascript', function() {
    var stream = gulp.src('src/**/*.js')
      .pipe(sourcemaps.init())
        .pipe(concat('all.js'))
        .pipe(uglify())
      .pipe(sourcemaps.write({includeContent: false, sourceRoot: '/src'}))
      .pipe(gulp.dest('dist'));
  });
  ```

### Plugin developers only: How to add source map support to plugins

- Generate a source map for the transformation the plugin is applying
- Apply this source map to the vinyl `file`. E.g. by using [vinyl-sourcemaps-apply](https://github.com/floridoo/vinyl-sourcemaps-apply).
  This combines the source map of this plugin with the source maps coming from plugins further up the chain.

#### Example:

```javascript
var through = require('through2');
var applySourceMap = require('vinyl-sourcemaps-apply');
var myTransform = require('myTransform');

module.exports = function(options) {

  function transform(file, encoding, callback) {
    // generate source maps if plugin source-map present
    if (file.sourceMap) {
      options.makeSourceMaps = true;
    }

    // do normal plugin logic
    var result = myTransform(file.contents, options);
    file.contents = new Buffer(result.code);

    // apply source map to the chain
    if (file.sourceMap) {
      applySourceMap(file, result.map);
    }

    this.push(file);
    callback();
  }

  return through.obj(transform);
};
```

[npm-image]: https://img.shields.io/npm/v/gulp-sourcemaps.svg?style=flat
[npm-url]: https://npmjs.org/package/gulp-sourcemaps
[travis-image]: https://img.shields.io/travis/floridoo/gulp-sourcemaps.svg?style=flat
[travis-url]: https://travis-ci.org/floridoo/gulp-sourcemaps
[coveralls-image]: https://img.shields.io/coveralls/floridoo/gulp-sourcemaps.svg?style=flat
[coveralls-url]: https://coveralls.io/r/floridoo/gulp-sourcemaps?branch=master
