---
layout: post
title: "Using JavaScript's Map and Reduce to implement common Lodash functions"
date: 2017-01-28T14:21:45-06:00
---

Recently, I needed to use a few functions commonly available in the [Lodash](https://lodash.com) and [Underscore.js](http://underscorejs.org) libraries. I wondered if I could implement my own version of these,
and sure enough, with JavaScript's [`reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)
and [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) it was fairly simple.

### zip([arrays])

`zip()` is a handy function that takes two arrays and returns an object whose keys are from one array and values are from the other.

```
zip(['foo', 'bar'], ['baz', 'biz']);
// => { 'foo': 'baz', 'bar': 'biz' }
```

With just a few lines we can easily implement this with JavaScript's `reduce()`.

```
function zip (keys, values) {
  return keys.reduce((result, currentKey, currentIndex) => {
    result[currentKey] = values[currentIndex];
    return result;
  }, {});
}
```

`reduce()` takes a callback, executed for each value in the array, and an initial value, in this case the empty object `{}`. The callback itself can take 4 arguments; an accumulator, the current value of the array, the current index of the array, and the array itself.

Above, we pass in an empty object to be assigned keys and values. Then, on each iteration through the `keys` array, we set the value of that key in our `result` object to be the element with the corresponding index in the `values` array. Finally, we must return our `result` so it becomes the accumulator in the next iteration.

### Object.values([object])

```
values({ a: 3, b: 5, c: 7 });
// => [3, 5, 7]
```

`Object.values()` is currently in the ECMAScript2017 Draft and, as of this writing, is supported by Chrome(51.0) and Firefox(47). However, it is not supported in Node and I actually needed to use it the other day. Both Lodash and Underscore have support for this but I figured it could be easily implemented with `reduce()`. Sure enough, here's what I came up with:

```
function values(obj) {
  return Object.keys(obj).reduce((vals, key) => vals.concat(obj[key]), []);
}
```

Here we reduce on the keys of the object, and our accumulator is an array, initialized by `[]` as the 2nd argument, that is being populated with the value of each key from the object. Since the return value of `foo.concat(bar)` is an array containing all of the original elements in `foo` in addition to `bar`, our accumulator in the next iteration is exactly what we want.


### filterWithKeys([arrays])

For lack of a better name, `filterWithKeys()` takes two arrays, the first being objects and the second being an array of keys we wish to filter for.

```
const objects = [{ a: 1, b: 3, c: 5, d: 7 }, { a: 9, b: 11, c: 13, d: 17 }];
const keys = ['a', 'c'];
filterWithKeys(objects, keys);
// => [{ a: 1, c: 5 }, { a: 9, c: 13 }]
```

I actually came across a pretty slick one liner solution given you know which fixed keys you want and are using JS in an environment that supports [object destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Object_destructuring).

```
const filterForAB = ({ a, b }) => ({ a, b });
filterForAB({ a: 1, b: 2, c: 3 });
// => { a: 1, b: 2 }
```

Obviously you can't dynamically choose which keys you would like to filter for but if you don't need to change these filters on the fly, this solution is slick. Essentially object destructuring is used in the argument to assign local variable `a` and `b` to `arg[a]` and `arg[b]`, respectively, from the object passed in. Then, a new object is created and returned with only the keys and values for `a` and `b` using [object shorthand](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#New_notations_in_ECMAScript_2015). This function could be combined with `map()` to filter the objects' keys in a collection, like so:

```
const objects = [{ a: 1, b: 3, c: 5, d: 7 }, { a: 9, b: 11, c: 13, d: 17 }];
objects.map(obj => filterForAB(obj));
// // => [{ a: 1, b: 3 }, { a: 9, b: 11 }]
```

However, if you would like a function that can accept which keys to filter for we have to try a different approach. Here is my solution,

```
function filterWithKeys(object, keys) {
  return keys.reduce((result, key) => {
    result[key] = object[key];
    return result;
  }, {});
}
```
Again, we pass in an empty object as the `initialValue` and `reduce()` the keys we wish to keep into a new object with values from our object being passed in. We can combine this with `map()` to filter all of the objects in a collection, like so:

```
const objects = [{ a: 1, b: 3, c: 5, d: 7 }, { a: 9, b: 11, c: 13, d: 17 }];
objects.map(obj => filterWithKeys(obj, ['c', 'd']));
// => [{ c: 5, d: 7 }, { c: 13, d: 17 }]
```

JavaScript's [`reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)
and [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
are really powerful when it comes to function composition. The next time you find yourself reaching for Lodash or Underscore, consider implementing your own version of that function, even if it's just for fun :)
