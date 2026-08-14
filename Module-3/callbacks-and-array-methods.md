# Extra lesson: Taking callbacks and array methods apart

**Level:** intermediate, no frameworks
**Estimated time:** about 2 hours
**Prerequisites:** Module 2 (arrays, objects, functions, arrow functions), and Lessons 3.1 and 3.2. You should have met `forEach`, `map`, `filter` and `find` at least once before starting, because this lesson rebuilds them rather than introducing them.

## How this lesson is different

The regular lessons introduce each array method one at a time: here is `.map()`, here is what it does, now try it. That works, and you should still do those lessons.

This one goes the other way round. Instead of learning the methods as a list of features to memorise, you are going to **build them yourself** first. By the end of the first hour you will have written your own working versions of `forEach`, `map`, `filter` and `find`, using nothing but a `for` loop and a function parameter.

Why bother, when JavaScript already has them? Because once you have written `map` yourself, you will never again wonder why your callback needs a `return`, why `forEach` cannot be chained, or what those mysterious second and third parameters are. The methods stop being magic and become something you could have written on a slow afternoon. That is a much sturdier kind of understanding than remembering which method returns what.

We then spend the second hour on the things the regular lessons do not cover: the wider toolkit (`some`, `every`, `findIndex`, `sort`), chaining methods into a pipeline, and a hard look at the mistakes that catch nearly everybody.

### Learning goals

By the end of this lesson you should be able to:

- Explain what it means for a function to be a value, and predict the difference between passing `doThing` and `doThing()`.
- Implement your own versions of `forEach`, `map`, `filter`, `find`, `some` and `every` with a `for` loop.
- Choose the right built-in method for a task and justify the choice.
- Chain methods into a readable data pipeline.
- Recognise and fix the six most common callback and array method bugs.

---

## Part 1: A function is just a value

Everything in this lesson rests on one idea. In JavaScript, a function is a value like any other. You can store it in a variable, put it in an array, hand it to another function, or get one back as a return value.

```js
// A function stored in a variable. Nothing unusual here.
const shout = function (text) {
  return text.toUpperCase() + '!';
};

console.log(typeof shout); // "function"
console.log(shout('hello')); // "HELLO!"

// A function stored in an array. Reach into the array, then call what you find.
const tools = [shout, (text) => text.trim()];

console.log(tools[0]('hello')); // "HELLO!"
console.log(tools[1]('  spaced out  ')); // "spaced out"

// A function stored as an object property.
const helpers = {
  shout: shout,
  quiet: (text) => text.toLowerCase(),
};

console.log(helpers.shout('hello')); // "HELLO!"
console.log(helpers.quiet('HELLO')); // "hello"
```

The last one should look familiar. `helpers.quiet('HELLO')` has exactly the same shape as `text.toUpperCase()`: a property lookup, then a call. A method is nothing more than a function stored on an object, which is why `sightings.map(...)` later in this lesson is not a special language feature. It is a function living on the array, being called like any other.

### The single most important distinction

`shout` is the function itself. `shout('hello')` is the *result* of running it.

```js
console.log(shout);          // the function object itself
console.log(shout('hello')); // "HELLO!"
```

When you pass a function to another function, you almost always want the first form: the function itself, with no parentheses. The parentheses mean "run this now, and use whatever comes back".

```js
// Correct: hand setTimeout the function, and let it run it in 1 second.
setTimeout(shout, 1000);

// Wrong: this runs shout immediately, and hands setTimeout the string "UNDEFINED!"
// (or in many cases it crashes). setTimeout then has nothing useful to call.
setTimeout(shout('hi'), 1000);
```

If you take one thing from this section, take this: **a callback is passed without parentheses, because you are handing over the recipe, not the meal.**

### Writing a function that takes a function

A function that accepts another function as a parameter is nothing special. The parameter is just a name, and you call it with `()` like any other function.

```js
// 'formatter' is a parameter that happens to hold a function.
function announce(name, formatter) {
  const line = 'Now arriving: ' + name;
  return formatter(line);
}

console.log(announce('platform 3 service', shout));
// "NOW ARRIVING: PLATFORM 3 SERVICE!"

console.log(announce('platform 3 service', (text) => text.toLowerCase()));
// "now arriving: platform 3 service"

// That inline arrow function is the same function we stored on 'helpers'
// earlier, so we could just as well hand over the one already sitting there.
console.log(announce('platform 3 service', helpers.quiet));
// "now arriving: platform 3 service"
```

Those last two calls are interchangeable. Whether you write the function on the spot or fetch a ready-made one out of an object makes no difference to `announce`, because either way it receives a function value and calls it. Note the missing parentheses on `helpers.quiet` once again: we want the function, not the result of running it.

The function that gets passed in (`shout`, or that inline arrow function) is the **callback**. `announce` is sometimes called the *higher-order function*, because it takes a function as an argument.

Notice what this buys us: `announce` decides *when* and *with what* the callback runs. The caller decides *what happens*. That split is the whole point, and it is exactly how every array method in this lesson works.

### Warm-up exercise (about 15 minutes)

Create a file `functions-as-values.js`.

1. Write three small functions: `toUpper(text)`, `toLower(text)` and `reverse(text)`. The last one should return the string backwards. Hint: `text.split('').reverse().join('')`.
2. Write a function `applyAll(text, transformers)` where `transformers` is an **array of functions**. It should run each one in turn on the text and log the result of each.
3. Call it: `applyAll('Field notes', [toUpper, toLower, reverse]);`
4. Now call it again with an inline arrow function added to the array, one that returns the length of the text instead.

<details>
<summary>Solution</summary>

```js
function toUpper(text) {
  return text.toUpperCase();
}

function toLower(text) {
  return text.toLowerCase();
}

function reverse(text) {
  return text.split('').reverse().join('');
}

function applyAll(text, transformers) {
  for (let i = 0; i < transformers.length; i++) {
    // transformers[i] is a function, so we call it with ()
    const result = transformers[i](text);
    console.log(result);
  }
}

applyAll('Field notes', [toUpper, toLower, reverse]);
// FIELD NOTES
// field notes
// seton dleiF

applyAll('Field notes', [toUpper, (text) => text.length]);
// FIELD NOTES
// 11
```

You have just written a higher-order function that takes a whole array of callbacks. If that made sense, the array methods will hold no surprises.

</details>

---

## Part 2: Build the methods yourself

Here is the data we will use for the rest of the lecture. It is a log from a bird observation hide: each entry records a species, where it was seen, how many individuals, and whether any of them carried a leg ring.

```js
const sightings = [
  { species: 'Fieldfare', site: 'North Hide', count: 12, ringed: false },
  { species: 'Curlew', site: 'Jetty', count: 3, ringed: true },
  { species: 'Fieldfare', site: 'Reed Bed', count: 40, ringed: false },
  { species: 'Bittern', site: 'Reed Bed', count: 1, ringed: true },
  { species: 'Curlew', site: 'North Hide', count: 7, ringed: false },
];
```

Copy it into a file called `build-your-own.js`. We will add to that file as we go.

### 2.1 Your own forEach

`forEach` does one thing: run a callback once per element. It hands the callback three arguments (value, index, whole array) and it throws away whatever the callback returns.

```js
function myForEach(array, callback) {
  for (let i = 0; i < array.length; i++) {
    callback(array[i], i, array);
  }
  // Note: nothing is returned. This is not an oversight.
}

myForEach(sightings, (sighting) => {
  console.log(sighting.species + ' at ' + sighting.site);
});
```

That is the entire implementation. Three lines of loop.

If that line in the middle, `callback(array[i], i, array);`, is doing something you could not explain to someone else, open the box below and we will take it apart piece by piece. If it already reads clearly, skip straight on to the two consequences.

<details>
<summary>Reading <code>callback(array[i], i, array)</code> slowly</summary>

Everything interesting happens on that one line, so it is worth taking apart.

**`callback` is a parameter holding a function.** When we called `myForEach(sightings, (sighting) => {...})`, JavaScript bound the two arguments to the two parameter names, exactly as it would with numbers or strings:

```js
// array    is now the sightings array
// callback is now the arrow function we wrote at the call site
```

Nothing about it is special because it holds a function. Writing `callback(...)` runs whatever function is currently sitting in that variable. On the next call to `myForEach` it might be a completely different function, and the loop neither knows nor cares.

**The parentheses run it, and the values inside are the arguments.** So on the very first pass of the loop, with `i` at 0, that one line is equivalent to writing:

```js
callback(sightings[0], 0, sightings);
```

which is the same as:

```js
callback({ species: 'Fieldfare', site: 'North Hide', count: 12, ringed: false }, 0, sightings);
```

**Those three arguments then land in the callback's parameters, by position.** Our callback declared only one parameter:

```js
(sighting) => {
  console.log(sighting.species + ' at ' + sighting.site);
}
```

So `sighting` receives the first argument, the object. The other two arguments are still sent, but since the callback declared no names for them, they are quietly ignored. JavaScript does not object to being handed more arguments than a function asks for.

That is why the parameter name is entirely yours to choose. These three callbacks behave identically, because position is what matters, not the name:

```js
myForEach(sightings, (sighting) => console.log(sighting.count));
myForEach(sightings, (item) => console.log(item.count));
myForEach(sightings, (x) => console.log(x.count));
```

And it is why you opt in to the extra information simply by declaring more parameters:

```js
myForEach(sightings, (sighting, index) => {
  console.log(index + ': ' + sighting.species);
});
// 0: Fieldfare
// 1: Curlew
// 2: Fieldfare
// 3: Bittern
// 4: Curlew
```

The third argument, the array itself, is rarely needed, but it is there for callbacks that want to compare an element against its neighbours or against the whole collection. You will use it in Exercise 4 to strip duplicates.

**The loop controls the timing.** Write out what the five iterations do and the picture is complete:

```
i = 0  ->  callback(sightings[0], 0, sightings)   logs "Fieldfare at North Hide"
i = 1  ->  callback(sightings[1], 1, sightings)   logs "Curlew at Jetty"
i = 2  ->  callback(sightings[2], 2, sightings)   logs "Fieldfare at Reed Bed"
i = 3  ->  callback(sightings[3], 3, sightings)   logs "Bittern at Reed Bed"
i = 4  ->  callback(sightings[4], 4, sightings)   logs "Curlew at North Hide"
```

Five separate calls to the same function, each with different arguments. This is the division of labour from Part 1 in its final form: `myForEach` owns the *when* and the *with what*, and your callback owns the *what happens*. Neither half needs to know anything about the other.

One last detail, easy to miss. The callback runs five times, which means five separate function calls, each with its own scope. A variable declared inside the callback is created fresh on every pass and is gone by the next one:

```js
myForEach(sightings, (sighting) => {
  const label = sighting.species; // a brand new 'label' each time
  console.log(label);
});
// console.log(label); // ReferenceError: label is not defined out here
```

If you need something to survive across iterations, it has to be declared *outside* the callback, in the surrounding scope. We use that trick in Part 3 to add up totals.

</details>

### Two consequences

Two things fall straight out of the implementation:

- **`forEach` returns `undefined`.** There is no `return` in the loop. That is why you can never chain anything onto the end of a `forEach`.
- **You cannot `break` out of it.** `break` only works inside a loop, and by the time your callback runs you are inside a *function*, not inside the loop. A `return` in your callback just ends that one call and the loop carries on to the next item.

The real method is written as `array.forEach(callback)` rather than `myForEach(array, callback)`, but the behaviour is what you see above.

### 2.2 Your own map

`map` is `forEach` with one change: it keeps what the callback returns, and gives you a new array of those results.

```js
function myMap(array, callback) {
  const results = [];
  for (let i = 0; i < array.length; i++) {
    results.push(callback(array[i], i, array));
  }
  return results;
}

const labels = myMap(sightings, (sighting) => {
  return sighting.species + ' x' + sighting.count;
});

console.log(labels);
// [ 'Fieldfare x12', 'Curlew x3', 'Fieldfare x40', 'Bittern x1', 'Curlew x7' ]
```

Look at the `push` line. It pushes *whatever the callback returned*. If your callback returns nothing, it pushes `undefined`. This is the single most common `map` bug in existence, and now you can see exactly why it happens:

```js
// The braces make a function body, and there is no return in it.
const broken = myMap(sightings, (sighting) => {
  sighting.species; // this value goes nowhere
});
console.log(broken); // [ undefined, undefined, undefined, undefined, undefined ]
```

Also note that the output array always has the same length as the input. `map` transforms, it never removes. If your result is shorter, you wanted `filter`.

### 2.3 Your own filter

`filter` keeps the *original* elements, but only the ones for which the callback returned something truthy.

```js
function myFilter(array, callback) {
  const results = [];
  for (let i = 0; i < array.length; i++) {
    if (callback(array[i], i, array)) {
      results.push(array[i]); // note: the element, not the callback's result
    }
  }
  return results;
}

const flocks = myFilter(sightings, (sighting) => sighting.count >= 5);
console.log(flocks.length); // 3
```

Compare the `push` line with the one in `myMap`. In `map` we push the callback's return value. In `filter` we push the element and only *test* the return value. That one difference is the whole distinction between the two methods, and it explains why a `filter` callback should return a boolean while a `map` callback should return the new value.

The output array of `filter` is between zero and the original length. Never longer.

### 2.4 Your own find

`find` is `filter` that gives up early and returns a single element instead of an array.

```js
function myFind(array, callback) {
  for (let i = 0; i < array.length; i++) {
    if (callback(array[i], i, array)) {
      return array[i]; // leaves the whole function immediately
    }
  }
  return undefined; // we got to the end without a match
}

const firstRinged = myFind(sightings, (sighting) => sighting.ringed);
console.log(firstRinged); // { species: 'Curlew', site: 'Jetty', count: 3, ringed: true }

const osprey = myFind(sightings, (sighting) => sighting.species === 'Osprey');
console.log(osprey); // undefined
```

That trailing `return undefined` is why `find` gives you `undefined` on a miss, and why you must check the result before using it. It is also why `find` is faster than `filter` on a big array when you only want one match: it stops walking as soon as it has an answer.

### Exercise 1: implement the toolkit (about 30 minutes)

In `build-your-own.js`, write these four functions from scratch. Do not look at the code above while you write them, and test each one against the `sightings` array.

1. `myEvery(array, callback)` - returns `true` only if the callback returns truthy for **every** element. It should stop early and return `false` the moment it finds one that fails.
2. `mySome(array, callback)` - returns `true` if the callback returns truthy for **at least one** element. It should stop early on the first success.
3. `myIndexWhere(array, callback)` - like `find`, but returns the *index* of the first match, or `-1` if there is none. (The real method is called `findIndex`.)
4. `myCountWhere(array, callback)` - returns how many elements pass the test, as a number.

Test with, at minimum:

```js
console.log(myEvery(sightings, (s) => s.count > 0));      // true
console.log(myEvery(sightings, (s) => s.ringed));         // false
console.log(mySome(sightings, (s) => s.count > 30));      // true
console.log(myIndexWhere(sightings, (s) => s.ringed));    // 1
console.log(myIndexWhere(sightings, (s) => s.count > 99)); // -1
console.log(myCountWhere(sightings, (s) => s.site === 'North Hide')); // 2
```

<details>
<summary>Solution</summary>

```js
function myEvery(array, callback) {
  for (let i = 0; i < array.length; i++) {
    if (!callback(array[i], i, array)) {
      return false; // one failure is enough, stop here
    }
  }
  return true; // nothing failed
}

function mySome(array, callback) {
  for (let i = 0; i < array.length; i++) {
    if (callback(array[i], i, array)) {
      return true; // one success is enough
    }
  }
  return false;
}

function myIndexWhere(array, callback) {
  for (let i = 0; i < array.length; i++) {
    if (callback(array[i], i, array)) {
      return i; // return the position, not the element
    }
  }
  return -1;
}

function myCountWhere(array, callback) {
  let total = 0;
  for (let i = 0; i < array.length; i++) {
    if (callback(array[i], i, array)) {
      total = total + 1;
    }
  }
  return total;
}
```

Worth noticing: `myEvery` on an empty array returns `true`, and `mySome` returns `false`. That is not a bug, it matches the real methods, and it follows directly from where the `return` statements sit.

</details>

### Exercise 2: transform, then select (about 20 minutes)

Your six functions each do one job. Real questions usually need two or three of them working together, and the interesting part is deciding what order to put them in.

Write these, using only the functions you have built yourself. Do not use the built-in methods yet.

1. `siteNames` - an array of just the site names from `sightings`, with each one appearing only once. Hint: build the full list first, then keep an entry only when it is the first time that name has been seen. Your `myIndexWhere` can tell you where a name first appears.
2. `flockLabels` - labels like `'Fieldfare x40'`, but only for sightings of five or more birds. Do the selecting before the transforming, so you are not building labels you are about to discard.
3. `busiestSite` - a function that takes a site name and returns the number of sightings recorded there. Test it with `'Reed Bed'` (the answer is 2).
4. `describe` - a function that takes the array and returns a single readable string, such as `'5 sightings, 2 of them ringed'`. Use `myCountWhere` for the second number.

<details>
<summary>Solution</summary>

```js
// 1
const allSites = myMap(sightings, (sighting) => sighting.site);
const siteNames = myFilter(allSites, (name, index) => {
  // keep this name only if this is the first position it appears in
  return myIndexWhere(allSites, (other) => other === name) === index;
});
console.log(siteNames); // [ 'North Hide', 'Jetty', 'Reed Bed' ]

// 2
const flocks = myFilter(sightings, (sighting) => sighting.count >= 5);
const flockLabels = myMap(flocks, (sighting) => sighting.species + ' x' + sighting.count);
console.log(flockLabels); // [ 'Fieldfare x12', 'Fieldfare x40', 'Curlew x7' ]

// 3
function busiestSite(siteName) {
  return myCountWhere(sightings, (sighting) => sighting.site === siteName);
}
console.log(busiestSite('Reed Bed')); // 2

// 4
function describe(list) {
  const ringedCount = myCountWhere(list, (sighting) => sighting.ringed);
  return list.length + ' sightings, ' + ringedCount + ' of them ringed';
}
console.log(describe(sightings)); // "5 sightings, 2 of them ringed"
```

Task 1 is the fiddliest, and it is worth sitting with. The callback receives the index as its second argument, so it can ask: is the position I am at right now the same as the position where this name first appeared? If yes, this is the original and we keep it. If no, we have seen it before and we drop it. You will meet this exact pattern again with the built-in methods in Exercise 4.

Notice that tasks 3 and 4 are ordinary functions that happen to use your higher-order functions inside them. Nothing exotic is going on, and that is rather the point: these are just tools now.

</details>

---

## Part 3: The real methods, and the rest of the toolkit

Now switch to the built-in versions. They behave exactly like the ones you wrote, with two differences in shape: you call them **on** the array with a dot, and you do not pass the array as the first argument.

```js
sightings.forEach((sighting) => console.log(sighting.species));
const labels = sightings.map((sighting) => sighting.species);
const flocks = sightings.filter((sighting) => sighting.count >= 5);
const firstRinged = sightings.find((sighting) => sighting.ringed);
```

Here is the wider toolkit, with the question each method answers. Learning them by the *question* rather than the name is much more reliable when you are stuck.

| Question you are asking | Method | Gives you back |
| --- | --- | --- |
| Do something for each item | `forEach` | `undefined` |
| Turn each item into something else | `map` | new array, same length |
| Keep only the items that match | `filter` | new array, shorter or equal |
| Get the first item that matches | `find` | one element, or `undefined` |
| Where is the first match | `findIndex` | a number, or `-1` |
| Does at least one match | `some` | `true` or `false` |
| Do they all match | `every` | `true` or `false` |
| Is this exact value in here | `includes` | `true` or `false` |
| Put them in order | `sort` | the **same** array, reordered |
| Join them into a string | `join` | a string |

Four of these deserve a closer look.

### some and every

These read almost like English and they are badly underused. Reach for them whenever you are tempted to write a loop with a flag variable.

```js
console.log(sightings.some((s) => s.count > 30));   // true
console.log(sightings.every((s) => s.count > 0));   // true
console.log(sightings.every((s) => s.ringed));      // false
```

Compare that with the version people often write instead:

```js
// Do not do this
let foundBigFlock = false;
sightings.forEach((s) => {
  if (s.count > 30) {
    foundBigFlock = true;
  }
});
```

Five lines, a mutable flag, and it checks every element even after the answer is settled. `some` does the same job in one line and stops early.

### Totals and grouping: when forEach is the right answer

`map`, `filter` and `find` all hand you a new array. But plenty of questions have a single answer: a total, a count, an average, a lookup table. For those, use `forEach` with a variable declared **outside** the callback.

This is the exception to the rule that `forEach` is the least useful of the methods. When you are accumulating rather than transforming, it is exactly right.

```js
// A running total. 'total' lives outside, so it survives every iteration.
let total = 0;
sightings.forEach((sighting) => {
  total = total + sighting.count;
});
console.log(total); // 63
```

Remember the scope point from Part 2: the callback runs five separate times, and anything declared *inside* it is gone by the next call. The accumulator has to sit outside for the additions to add up.

The same shape builds a lookup table. Here we total the birds seen at each site:

```js
const perSite = {};

sightings.forEach((sighting) => {
  const current = perSite[sighting.site] || 0;
  perSite[sighting.site] = current + sighting.count;
});

console.log(perSite); // { 'North Hide': 19, Jetty: 3, 'Reed Bed': 41 }
```

Read the middle line carefully, because it is the crux. `perSite[sighting.site]` is `undefined` the first time we meet a site, and `undefined || 0` gives `0`. So the first sighting for a site starts its total from zero, and every later one adds to what is already there.

Two habits worth forming:

1. Declare the accumulator with `let` if it is a number you reassign, and `const` if it is an object or array you add to. Writing `const total = 0` and then `total = total + 1` inside the callback throws a `TypeError`.
2. Declare it immediately above the `forEach`, never inside. If you find yourself hunting for where a total was declared, it is too far away.

One last thing, so it does not surprise you later. There is a built-in method called `reduce` that does this accumulating job in a single call, without the variable outside. You will meet it in code you read long before you need to write it yourself, and it is worth recognising the shape then: if a chain ends in `.reduce(...)`, the array is being boiled down to one value. The `forEach` pattern above does the same work and is easier to read while you are learning, so stick with it for now.

### sort changes the original array

This is the one method in the list that does **not** leave your data alone. It sorts in place and returns a reference to the same array.

```js
const counts = [12, 3, 40, 1, 7];
counts.sort();
console.log(counts); // [ 1, 12, 3, 40, 7 ]
```

That is not a bug. With no callback, `sort` converts each element to a string and sorts alphabetically, and `'12'` really does come before `'3'`. For numbers you must supply a comparator: return a negative number to put `a` first, a positive number to put `b` first, and `0` to leave them level.

```js
const safeCopy = [...counts]; // copy first, so the original is untouched
safeCopy.sort((a, b) => a - b);
console.log(safeCopy); // [ 1, 3, 7, 12, 40 ]

safeCopy.sort((a, b) => b - a);
console.log(safeCopy); // [ 40, 12, 7, 3, 1 ]
```

For strings, use `localeCompare`:

```js
const species = ['Fieldfare', 'Bittern', 'Curlew'];
console.log([...species].sort((a, b) => a.localeCompare(b)));
// [ 'Bittern', 'Curlew', 'Fieldfare' ]
```

Get into the habit of copying with `[...array]` before sorting. Sorting the array that something else on the page is also using is a genuinely nasty bug to track down.

### Chaining: the pipeline

Because `map`, `filter` and `sort` all return arrays, you can hang the next method straight onto the end of the previous one. Each step takes the output of the one before it.

```js
const report = sightings
  .filter((sighting) => sighting.count >= 5 && !sighting.ringed)
  .sort((a, b) => b.count - a.count)
  .map((sighting) => sighting.species + ' x' + sighting.count);

console.log(report);
// [ 'Fieldfare x40', 'Fieldfare x12', 'Curlew x7' ]
```

Read it top to bottom as a sentence: take the sightings, keep the unringed flocks of five or more, put the biggest first, then turn each one into a label.

Three habits that keep chains readable:

- **Filter before you map.** There is no point transforming items you are about to throw away.
- **One idea per step.** If a single callback is doing two unrelated things, split it into two steps.
- **Break the chain when it stops reading well.** Three or four steps is usually the limit before a named intermediate variable is clearer. Chaining is a readability tool, not a scoring system.

And one warning: `forEach` cannot appear anywhere except the very end of a chain, because it returns `undefined`. If you find yourself wanting to chain after a `forEach`, you wanted `map`.

---

## Part 4: The six bugs everybody writes

Rather than describing these, here they are as broken code. Work out what each one prints and why *before* you run it, then fix it. This is the single most useful 20 minutes in the lesson, because these are the bugs you will actually hit next week.

Create `debug-me.js` with the `sightings` array at the top.

```js
// Bug 1
const speciesList = sightings.map((sighting) => {
  sighting.species;
});
console.log(speciesList);

// Bug 2
const bigOnes = sightings.forEach((sighting) => sighting.count > 10);
console.log(bigOnes);

// Bug 3
const osprey = sightings.find((sighting) => sighting.species === 'Osprey');
console.log('Seen ' + osprey.count + ' ospreys');

// Bug 4
sightings.forEach((sighting) => {
  let total = 0;
  total = total + sighting.count;
  console.log(total);
});

// Bug 5
const sortedCounts = [12, 3, 40, 1, 7].sort();
console.log(sortedCounts);

// Bug 6
const numbers = ['10', '10', '10'].map(parseInt);
console.log(numbers);
```

<details>
<summary>Solutions and explanations</summary>

**Bug 1** prints `[ undefined, undefined, undefined, undefined, undefined ]`. The braces create a function body, and a function body with no `return` returns `undefined`. `map` dutifully collects five `undefined` values.

Fix by returning, or by dropping the braces so the arrow function returns implicitly:

```js
const speciesList = sightings.map((sighting) => sighting.species);
```

**Bug 2** prints `undefined`. `forEach` always returns `undefined`, no matter what the callback returns. The callback here computes a boolean that is thrown away immediately. This person wanted `filter`:

```js
const bigOnes = sightings.filter((sighting) => sighting.count > 10);
```

**Bug 3** crashes with `TypeError: Cannot read properties of undefined (reading 'count')`. There is no Osprey in the array, so `find` returned `undefined`, and `undefined.count` is an error. Always guard the result:

```js
const osprey = sightings.find((sighting) => sighting.species === 'Osprey');
if (osprey) {
  console.log('Seen ' + osprey.count + ' ospreys');
} else {
  console.log('No ospreys recorded');
}
```

**Bug 4** prints `12`, `3`, `40`, `1`, `7` rather than a running total ending at 63. `total` is declared *inside* the callback, so it is created fresh and reset to `0` on every one of the five calls. Nothing accumulates. Move the declaration outside, where it survives:

```js
let total = 0;
sightings.forEach((sighting) => {
  total = total + sighting.count;
});
console.log(total); // 63
```

Watch the keyword when you move it. `const total = 0` followed by `total = total + ...` throws `TypeError: Assignment to constant variable`, which is the very next thing most people hit.

**Bug 5** prints `[ 1, 12, 3, 40, 7 ]`. The default sort compares strings. Pass a comparator:

```js
const sortedCounts = [12, 3, 40, 1, 7].sort((a, b) => a - b);
```

**Bug 6** prints `[ 10, NaN, 2 ]`, which looks like nonsense until you remember that `map` passes **three** arguments to the callback, and `parseInt` takes **two**: the text and the radix (the number base). So the calls are really `parseInt('10', 0)`, `parseInt('10', 1)` and `parseInt('10', 2)`, which give 10, `NaN` and 2. Wrap it so only the argument you want gets through:

```js
const numbers = ['10', '10', '10'].map((text) => parseInt(text, 10)); // [10, 10, 10]
```

This last one is the reason to know that the index is passed at all. It bites people who pass a named function straight to `map` without checking how many parameters that function accepts.

</details>

---

## Part 5: Practice exercises

New dataset. This is the job log from a community repair workshop, where volunteers help people mend things instead of binning them.

```js
const repairs = [
  { id: 'r1', item: 'Toaster', volunteer: 'Iris', minutes: 45, fixed: true, parts: ['fuse'] },
  { id: 'r2', item: 'Desk lamp', volunteer: 'Omar', minutes: 20, fixed: true, parts: [] },
  { id: 'r3', item: 'Bicycle', volunteer: 'Iris', minutes: 90, fixed: false, parts: ['brake cable', 'inner tube'] },
  { id: 'r4', item: 'Kettle', volunteer: 'Sam', minutes: 30, fixed: true, parts: ['element'] },
  { id: 'r5', item: 'Radio', volunteer: 'Omar', minutes: 75, fixed: false, parts: ['capacitor'] },
  { id: 'r6', item: 'Sewing machine', volunteer: 'Sam', minutes: 60, fixed: true, parts: ['belt', 'needle'] },
];
```

Put it in `repairs-practice.js`. For every task, work out which question you are asking before you pick a method, and use the built-in methods from here on.

### Exercise 3: single method queries (about 25 minutes)

1. Log a line for each repair in the form `Toaster - Iris - 45 min`.
2. Build an array `itemNames` containing only the item names.
3. Build an array `unfixed` containing the full objects for repairs that were not fixed.
4. Find the repair with the id `r4` and log its item name.
5. Find the repair for an `item` of `'Hairdryer'` and log a sensible message when it is not there.
6. Answer with a boolean: did any repair take more than 80 minutes?
7. Answer with a boolean: did every repair need at least one part?
8. Count how many repairs Iris handled.

<details>
<summary>Solution</summary>

```js
// 1
repairs.forEach((repair) => {
  console.log(repair.item + ' - ' + repair.volunteer + ' - ' + repair.minutes + ' min');
});

// 2
const itemNames = repairs.map((repair) => repair.item);
console.log(itemNames);
// [ 'Toaster', 'Desk lamp', 'Bicycle', 'Kettle', 'Radio', 'Sewing machine' ]

// 3
const unfixed = repairs.filter((repair) => !repair.fixed);
console.log(unfixed.length); // 2

// 4
const r4 = repairs.find((repair) => repair.id === 'r4');
console.log(r4.item); // Kettle

// 5
const hairdryer = repairs.find((repair) => repair.item === 'Hairdryer');
if (hairdryer) {
  console.log('Found: ' + hairdryer.id);
} else {
  console.log('No hairdryer in the log');
}

// 6
console.log(repairs.some((repair) => repair.minutes > 80)); // true

// 7
console.log(repairs.every((repair) => repair.parts.length > 0)); // false

// 8
const irisJobs = repairs.filter((repair) => repair.volunteer === 'Iris');
console.log(irisJobs.length); // 2
```

On task 8: `filter().length` is perfectly good and reads clearly. There is no need for a counter variable and a loop when the array can just tell you how long it is.

</details>

### Exercise 4: chains (about 25 minutes)

Each of these should be a single chain. Filter before you map.

1. `fixedLabels`: the item names of the successful repairs only, in the form `'Toaster (45 min)'`.
2. `quickWins`: the item names of repairs that were fixed **and** took under 50 minutes.
3. `longestFirst`: all repairs sorted longest to shortest, as strings like `'Bicycle: 90'`. Do not modify the original `repairs` array. Verify that you have not by logging `repairs[0].item` afterwards and checking it is still `'Toaster'`.
4. `volunteersOnFailures`: the names of volunteers who had at least one repair that failed, with no duplicates. Hint: build the list first, then remove duplicates with `filter` and `indexOf`.

<details>
<summary>Solution</summary>

```js
// 1
const fixedLabels = repairs
  .filter((repair) => repair.fixed)
  .map((repair) => repair.item + ' (' + repair.minutes + ' min)');
console.log(fixedLabels);
// [ 'Toaster (45 min)', 'Desk lamp (20 min)', 'Kettle (30 min)', 'Sewing machine (60 min)' ]

// 2
const quickWins = repairs
  .filter((repair) => repair.fixed && repair.minutes < 50)
  .map((repair) => repair.item);
console.log(quickWins); // [ 'Toaster', 'Desk lamp', 'Kettle' ]

// 3
const longestFirst = [...repairs]
  .sort((a, b) => b.minutes - a.minutes)
  .map((repair) => repair.item + ': ' + repair.minutes);
console.log(longestFirst);
// [ 'Bicycle: 90', 'Radio: 75', 'Sewing machine: 60', 'Toaster: 45', 'Kettle: 30', 'Desk lamp: 20' ]
console.log(repairs[0].item); // Toaster, so the original order survived

// 4
const volunteersOnFailures = repairs
  .filter((repair) => !repair.fixed)
  .map((repair) => repair.volunteer)
  .filter((name, index, all) => all.indexOf(name) === index);
console.log(volunteersOnFailures); // [ 'Iris', 'Omar' ]
```

That duplicate removal in task 4 is worth reading twice. `all.indexOf(name)` gives the position of the *first* time this name appears. If that is not the position we are currently at, we have seen it before, so we drop it. This is one of the few places where the index and array parameters really earn their keep.

</details>

### Exercise 5: totals and grouping (about 20 minutes)

These all have a single value as their answer, so reach for `forEach` and an accumulator declared outside the callback. Some of them combine that with a `filter` first.

1. `totalMinutes`: the total time spent across all repairs. (Expected: 320.)
2. `fixedMinutes`: the total time spent on repairs that succeeded. (Expected: 155.)
3. `minutesByVolunteer`: an object mapping each volunteer to their total minutes. (Expected: `{ Iris: 135, Omar: 95, Sam: 90 }`.)
4. `allParts`: a single flat array of every part used across all repairs. (Expected: 7 entries.)
5. `busiest`: the repair object with the most minutes. Do it with a loop and a running champion, not with `sort`.
6. `averageMinutes`: the mean time per repair, rounded to one decimal place. Use `totalMinutes` and `.toFixed(1)` from Lesson 3.1.

<details>
<summary>Solution</summary>

```js
// 1
let totalMinutes = 0;
repairs.forEach((repair) => {
  totalMinutes = totalMinutes + repair.minutes;
});
console.log(totalMinutes); // 320

// 2
let fixedMinutes = 0;
repairs
  .filter((repair) => repair.fixed)
  .forEach((repair) => {
    fixedMinutes = fixedMinutes + repair.minutes;
  });
console.log(fixedMinutes); // 155

// 3
const minutesByVolunteer = {};
repairs.forEach((repair) => {
  const current = minutesByVolunteer[repair.volunteer] || 0;
  minutesByVolunteer[repair.volunteer] = current + repair.minutes;
});
console.log(minutesByVolunteer); // { Iris: 135, Omar: 95, Sam: 90 }

// 4
const allParts = [];
repairs.forEach((repair) => {
  repair.parts.forEach((part) => {
    allParts.push(part);
  });
});
console.log(allParts);
// [ 'fuse', 'brake cable', 'inner tube', 'element', 'capacitor', 'belt', 'needle' ]

// 5
let busiest = repairs[0];
repairs.forEach((repair) => {
  if (repair.minutes > busiest.minutes) {
    busiest = repair;
  }
});
console.log(busiest.item); // Bicycle

// 6
const averageMinutes = (totalMinutes / repairs.length).toFixed(1);
console.log(averageMinutes); // "53.3"
```

Two things to notice. In task 2, the `forEach` sits at the end of a chain, which is fine because nothing follows it. That is the only position it can ever occupy.

In task 4, the callback contains another `forEach` over `repair.parts`. Nesting them is allowed and often the clearest way to walk an array of arrays. Just keep the nesting to one level, and give the two parameters different names so you can always tell which loop you are in.

Task 6 returns a string, because `.toFixed()` always does. Wrap it in `Number()` if you need to do further sums with it.

</details>

---

## Before you move on

Whatever you write from here on, check it against this list. Every item is a mistake from Part 4 turned into a habit.

- Every callback that needs to return a value does return one.
- Every `find` result is checked before it is used.
- Every accumulator is declared outside its `forEach`, with `let` for numbers and `const` for objects and arrays.
- Nothing is chained after a `forEach`.
- Anything you `sort` is a copy, and numbers get a comparator.
- No chain runs longer than four steps without a named intermediate variable.

---

## Quick reference

```js
// Do something with each item. Returns undefined. Cannot be chained.
items.forEach((item) => console.log(item));

// Transform each item. Always same length out.
const changed = items.map((item) => item.name);

// Keep matching items. Never longer than the input.
const matching = items.filter((item) => item.active);

// First match, or undefined. Always check the result.
const one = items.find((item) => item.id === wanted);

// Position of first match, or -1.
const at = items.findIndex((item) => item.id === wanted);

// Booleans. Both stop early.
const any = items.some((item) => item.active);
const all = items.every((item) => item.active);

// Accumulate a single answer. Declare the variable outside the callback.
let total = 0;
items.forEach((item) => {
  total = total + item.price;
});

// Reorder. Mutates, so copy first, and always pass a comparator for numbers.
const ordered = [...items].sort((a, b) => a.price - b.price);
```

## References

All of these are on MDN Web Docs, which is the reference every working JavaScript developer keeps open.

1. [Callback function](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function) - the glossary definition, worth reading once you have built your own.
2. [First-class function](https://developer.mozilla.org/en-US/docs/Glossary/First-class_Function) - the idea behind all of Part 1.
3. [Arrow function expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) - in particular the section on implicit returns.
4. [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) - the full list of methods, far longer than the ones here.
5. [Array.prototype.forEach()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)
6. [Array.prototype.map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
7. [Array.prototype.filter()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)
8. [Array.prototype.find()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find)
9. [Array.prototype.findIndex()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex)
10. [Array.prototype.some()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/some)
11. [Array.prototype.every()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every)
12. [Array.prototype.sort()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) - read the section on the comparator before you use it in anger.

A tip on reading MDN: skip past the syntax block to the Examples section first. The examples are usually clearer than the prose above them, and you can work backwards to the formal description once you have the shape of it.
