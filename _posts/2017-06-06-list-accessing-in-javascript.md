---
layout: post
title: "List accessing in Javascript"
categories: javascript clojure clojurescript arrays
---

Javascript's syntax for accessing specific indexes in arrays is simple and effective however it has meant that the language has developed without forming a pragmatic approach to accessing an array.

```js
const testList = ["a", "b", "c", "d", "e"];

testList[0] // get first element
testList[1] // get second element
testList[testList.length - 1] // last element
testList.slice(-1)[0]         // last element
testList.slice(1) // get rest of element
```

This is clearly not concise and it is not obvious to beginners why we use certain approaches for some things and other approaches for a seemingly identical task. Of course as you learn about mutability you discover why we dont use `.shift()` or `.pop()` to get the first or last elements.

In Clojure(script) there is no *extra* syntax for accessing any of our data structures. Therefore the default way to access an element in a list or array is to use a function `nth` and provide the list and the index, as soon as you begin to use one side-effect free function it becomes much more straightforward and clear what you are really asking for. Then sprinkle in a few extra helpful aliases `first`, `second` and `last` that use `nth` with fixed parameters.

```clj
(def test-list [:a :b :c :d :e])

(first test-list)     ;; :a
(second test-list)    ;; :b
(last test-list)      ;; :e
(rest test-list)      ;; '(:b :c :d :e)
(nth test-list 3)     ;; :d
```

Here it is clear what we are asking for in `test-list` and it is no surprise as to what is returned. I suspect that even if you are not familiar with lisp or Clojure you understand what this code means due to the verboseness of the keywords.

So I have suggested that Javascript needs to have some side-effect free method to access elements of an array, well `.slice` fits the bill. So how do we create a function that allows us to create our numerous *aliases*. One answer I have found is to use a curried function (a function that returns a function). This allows us to assign the function to a keyword such as `first` and call then call our first as a function `fn(arg1)(arg2)`.

```js
const nthItems = (...limits) => arr => {
  const items = arr.slice(...limits);
  return items.length === 1 ? items[0] : items;
}

const first = nthItems(0, 1);
const second = nthItems(1, 2);
const last = nthItems(-1);
const rest = nthItems(1);
const nth = (arr, n) => nthItems(n, n + 1)(arr);
```

So we have created out nthItems function to first take limits that will define the various alias functions which can then be passed an array and will finally return our indexed element. For clarity `first([0, 1, 2])` is the same as `nthItems(0, 1)([0, 1, 2])` but I think that is enough on currying. So above we see the various *aliases* and we see that they are each defined in the same way with the exception of `nth`. Our main `nthItems` functions is fairly simple it just slices the input array and returns it as a single element if necessary. The `nth` function is slightly different in that it takes its own index argument and will slice from that index to the next. So lets see if we have cleaned up array accessing a little bit.

```js
// Now we can use our functions in a more lispy way
const testList = ["a", "b", "c", "d", "e"];

first(testList)     // "a"
second(testList)    // "b"
last(testList)      // "e"
rest(testList)      // ["b", "c", "d", "e"]
nth(testList, 3)    // "d"
```

Now we can see that this is looking much cleaner that our original JavaScript snippet and its looking much more readable. We have approached the problem in a functional way by making a function that is reused, if we needed the second to last element, we can use a function called `butLast` it would be simple to extend this method.

```js
const butLast = nthItems(-2, -1);
butLast(["a", "b", "c"])    // => "b"
butLast([0, 1, 2, 3, 4])    // => 3
```
