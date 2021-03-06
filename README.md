# funstream

Funstream gives you iteratorish methods on your streams.

```
const fun = require('funstream')

fun.ary([1, 2, 3, 4, 5]).map(n => n + 1).filter(n => n % 2).map(n => `${n}\n`).pipe(process.stdout)
// prints lines with 3 and 5
fun.ary([1, 2, 3, 4, 5]).map(n => n + 1).filter(n => n % 2).reduce((a, b) => a + b).then(console.log)
// prints 8
```

Funstream makes object streams better.

## Funstream constructors

### fun.ify(stream[, opts]) → FunStream

This is probably what you want.

Makes an existing stream a funstream!  Has the advantage over `fun()` of
handling error propagation for you.

`opts` is an optional options object.  The only option currently is `async`
which let's you explicitly tell Funstream if your callbacks are sync or
async. If you don't include this we'll detect which you're using by looking
at the number of arguments your callback takes. Because promises and sync functions
take the same number of arguments, if you're using promise returning callcks you'll need to
explicitly pass in `async: true`.

### fun.ary(array[,opts]) → FunStream

Returns a funstream that will receive entries from the array one at a time
while respecting back pressure.

### fun([opts]) → FunStream

Make a passthrough Funstream.  You can pipe into this to get access to our
handy methods.  You'll want to do something about errors in the source
stream because `pipe` pretends they aren't a thing.

## Funstream methods

This is the good stuff.  All callbacks can be sync or async.  You can
indicate this by setting the `async` property on the opts object either when
calling the method below or when constructing the objects to start with.
Values of the `async` property progogate down the chain, for example:

`.map(…, {async: true}).map(…)`

The second map callback will also be assume do to be async.

Multiple sync functions of the same time will be automatically aggregated
without constructing additional streams, so:

`.filter(n => n < 23).filter(n => n > 5)`

The second `filter` call actually returns the same stream object.  This does
mean that if you try to fork the streams inbetween it won't work. Sorry.

### .filter(filterWith[, opts]) → FunStream

Filter the stream! 

* `filterWith(data) → Boolean` (can throw)
* `filterWith(data, cb)` (and `cb(err, shouldInclude)`)
* `filterWith(data) → Promise(Boolean)

If `filterWith` returns true, we include the value in the output stream,
otherwise not.

### .map(mapWith[, opts]) → FunStream

Transform the stream! 

* `mapWith(data) → newData` (can throw)
* `mapWith(data, cb)` (and `cb(err, newData)`)
* `mapWith(data) → Promise(newData)

`data` is replaced with `newData` from `mapWith` in the output stream.

### .reduce(reduceWith[, initial[, opts]]) → Promise

Promise the result of computing everything.

* `reduceWith(a, b) → newData` (can throw)
* `reduceWith(a, b, cb)` (and `cb(err, newData)`)
* `reduceWith(a, b) → Promise(newData)

Concat a stream:
```
fun.ify(stream)
  .reduce((a, b) => a + b)
  .then(wholeThing => { … })
```

### .forEach(consumeWith[, opts]) → Promise

Run some code for every chunk, promise that the stream is done.

Example, print each line:
```
fin.ify(stream)
  .forEach(chunk => console.log(chunk)
  .then(() => console.log('Done!'))
```