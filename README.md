[cover]: https://codecov.io/gh/rollup/stream/branch/master/graph/badge.svg
[cover-url]: https://codecov.io/gh/rollup/stream
[tests]: https://img.shields.io/circleci/project/github/rollup/stream.svg
[tests-url]: https://circleci.com/gh/rollup/stream
[npm]: https://img.shields.io/npm/v/@rollup/stream
[npm-url]: https://www.npmjs.com/package/@rollup/stream
[size]: https://packagephobia.now.sh/badge?p=@rollup/stream
[size-url]: https://packagephobia.now.sh/result?p=@rollup/stream

[![tests][tests]][tests-url]
[![cover][cover]][cover-url]
[![npm][npm]][npm-url]
[![size][size]][size-url]
[![libera manifesto](https://img.shields.io/badge/libera-manifesto-lightgrey.svg)](https://liberamanifesto.com)

# @rollup/stream

🍣 Stream Rollup build results

This package exists to provide a streaming interface for Rollup builds. This is useful in situations where a build system is working with [vinyl files](https://github.com/gulpjs/vinyl), such as [`gulp.js`](https://gulpjs.com/).

## Requirements

This plugin requires an [LTS](https://github.com/nodejs/Release) Node version (v8.0.0+) and Rollup v1.20.0+.

## Install

Using npm:

```console
npm install @rollup/stream --save-dev
```

## Usage

Assume a `src/index.js` file exists and contains code like the following:

```js
export default 'jingle bells, batman smells';
```

We can bundle `src/index.js` using streams such like:

```js
import rollupStream from '@rollup/stream';

const { log } = console;
const options = {
  input: 'src/index.js',
  output: { format: 'cjs' }
};
const stream = rollupStream(options);
let bundle = '';

stream.on('data', (data) => bundle += data);
stream.on('end', () => log(bundle));
```

The preceding code will concatenate each chunk (or asset) and output the entire bundle's content when Rollup has completed bundling and the stream has ended.

### Usage with Gulp

Using Gulp requires piping. Suppose one wanted to take the bundle content and run it through a minifier, such as [`terser`](https://www.npmjs.com/package/terser):

```js
import rollupStream from '@rollup/stream';
import gulp from 'gulp';
import terser from 'gulp-terser';
import source from 'vinyl-source-stream';

gulp.task('rollup', () => {
  const options = { input: 'src/index.js' };
  return rollupStream(options)
    .pipe(source('bundle.js'))
    .pipe(terser({ keep_fnames: true, mangle: false }))
    .pipe(gulp.dest('dist'));
});
```

### Using Sourcemaps

Rollup can produce source maps by specifying the `sourcemap` output option. To use the generated sourcemaps with Gulp:

```js
import rollupStream from '@rollup/stream';
import buffer from 'vinyl-buffer';
import gulp from 'gulp';
import sourcemaps from 'gulp-sourcemaps';
import source from 'vinyl-source-stream';

gulp.task('rollup', () => {
  const options = { input: 'src/index.js', output: { sourcemap: true } };
  return rollupStream(options)
    .pipe(source('bundle.js'))
    .pipe(buffer())
    .pipe(sourcemaps.init({loadMaps: true}))
    .pipe(sourcemaps.write('dist'))
    .pipe(gulp.dest('dist'));
});
```

### Watching for changes

To optimize build times, you will need to make use of the `cache` option when watching for file changes:

```js
import rollupStream from '@rollup/stream';
import buffer from 'vinyl-buffer';
import gulp from 'gulp';
import source from 'vinyl-source-stream';

// cache variable needs to be stored outside the gulp task
let cache;

gulp.task('rollup', () => {
  const options = {
    input: 'src/index.js',
    // Apply cache in the options object
    cache: cache,
  };
  return rollupStream(options)
    .on('bundle', bundle => {
      // Update the cache data after every new bundle is created
      cache = bundle;
    })
    .pipe(source('bundle.js'))
    .pipe(buffer())
    .pipe(gulp.dest('dist'));
});

gulp.task('watch', done => {

  // Gulp 4
  gulp.watch('./src/**/*.js', gulp.series('rollup'));

  // Gulp 3
  gulp.watch('./src/**/*.js', ['rollup']);

  done();
});

```

_(Example reproduced from [Rollup-Stream readme](https://github.com/Permutatrix/rollup-stream#readme))_

## Options

All [Rollup options](https://www.rollupjs.org/guide/en/#configuration-files) are valid to pass as options to `@rollup/stream`.

## Meta

[CONTRIBUTING](/.github/CONTRIBUTING.md)

[LICENSE (MIT)](/LICENSE)
