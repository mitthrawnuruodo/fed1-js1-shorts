# Extra lesson: Taking callbacks and array methods apart

**Estimated time:** about 2 hours  
**You will need:** variables, `if`, the `for` loop, arrays, objects, functions and arrow functions. Nothing else.

## How this lesson is different

The regular lessons hand you the array methods one at a time: here is `.map()`, here is what it does, now try it. That is a sensible way to meet them, and you should still do those lessons.

This one goes the other way round. Instead of learning the methods as a list of features to memorise, you are going to **build them yourself** first, using nothing but a `for` loop from Module 1 and a function from Module 2.

Why bother, when JavaScript already has them? Because once you have written `map` yourself, you will never again wonder why your callback needs a `return`, or why `forEach` behaves differently from the others. The methods stop being magic and become something you could have written yourself on a slow afternoon. That is a much sturdier kind of understanding than remembering which method returns what.

Along the way there are a few boxes you can open if you want the longer explanation, and skip if you do not. They are optional.

### Learning goals

By the end of this lesson you should be able to:

- Explain what it means for a function to be a value, and predict the difference between passing `doThing` and `doThing()`.
- Write your own versions of `forEach`, `map`, `filter` and `find` with a `for` loop.
- Choose the right method for a task and say why.
- Combine `filter` and `map` into one readable line.
- Recognise and fix the five most common mistakes with these methods.

---

## Part 1: A function is just a value

Everything in this lesson rests on one idea. In JavaScript, a function is a value like any other. You can store it in a variable, put it in an array, or put it in an object.

```js
// A function stored in a variable. You have done this already with arrow functions.
const shout = function (text) {
  return text.toUpperCase() + '!';
};

console.log(typeof shout); // "function"
console.log(shout('hello')); // "HELLO!"

// A function stored in an array. Reach into the array, then call what you find.
const tools = [shout, (text) => text.toLowerCase()];

console.log(tools[0]('hello')); // "HELLO!"
console.log(tools[1]('HELLO')); // "hello"

// A function stored in an object.
const helpers = {
  shout: shout,
  quiet: (text) => text.toLowerCase(),
};

console.log(helpers.shout('hello')); // "HELLO!"
console.log(helpers.quiet('HELLO')); // "hello"
```

That last one should look familiar. `helpers.quiet('HELLO')` has exactly the same shape as `text.toUpperCase()` from Lesson 3.1: a dot, a name, then parentheses. A method is nothing more than a function stored on an object. That is worth remembering when we get to `sightings.map(...)` later, because it means `map` is not special syntax. It is a function that lives on the array.

### The most important distinction in this lesson

`shout` is the function itself. `shout('hello')` is the **result** of running it.

```js
console.log(shout);          // the function
console.log(shout('hello')); // "HELLO!"
```

The parentheses mean "run this now". Without them, you are talking about the function rather than running it.

When you hand a function to another function, you nearly always want it **without** parentheses, because you are handing over the recipe, not the meal. Someone else will do the cooking, later, and possibly several times.

### Writing a function that takes a function

A function that accepts another function is nothing special. The parameter is just a name, and you call it with `()` like anything else.

```js
// 'formatter' is a parameter that happens to hold a function.
function announce(name, formatter) {
  const line = 'Now arriving: ' + name;
  return formatter(line);
}

console.log(announce('the 3 o clock train', shout));
// "NOW ARRIVING: THE 3 O CLOCK TRAIN!"

console.log(announce('the 3 o clock train', helpers.quiet));
// "now arriving: the 3 o clock train"

// Or write the function on the spot, which is what you will see most often.
console.log(announce('the 3 o clock train', (text) => text + ' (delayed)'));
// "Now arriving: the 3 o clock train (delayed)"
```

Those three calls are interchangeable as far as `announce` is concerned. Whether the function was stored earlier or written on the spot makes no difference. Either way `announce` receives a function and calls it.

The function you pass in is the **callback**. That is the whole definition.

Notice what this buys us. `announce` decides *when* the callback runs and *what it gets*. You decide *what happens*. That split is the entire point, and it is exactly how the array methods work.

### Warm-up exercise (about 10 minutes)

Create a file `functions-as-values.js`.

1. Write two small functions, `addExclamation(text)` and `addQuestion(text)`, which return the text with `!` or `?` on the end.
2. Write a function `decorate(text, decorator)` that returns the result of calling `decorator` on `text`.
3. Call it with each of your two functions.
4. Call it once more with a function written on the spot, one that returns the text in brackets.

<details>
<summary>Solution</summary>

```js
function addExclamation(text) {
  return text + '!';
}

function addQuestion(text) {
  return text + '?';
}

function decorate(text, decorator) {
  return decorator(text);
}

console.log(decorate('Hello', addExclamation)); // Hello!
console.log(decorate('Hello', addQuestion)); // Hello?
console.log(decorate('Hello', (text) => '[' + text + ']')); // [Hello]
```

Check one thing before moving on: no parentheses after `addExclamation` when you pass it in. If you wrote `decorate('Hello', addExclamation())` you would be calling it too early, with no text, and handing `decorate` the result instead of the function.

</details>

---

## Part 2: Build the methods yourself

Here is the data for this part. It is a log from a bird watching hide: what was seen, where, how many, and whether any of the birds had a ring on its leg.

```js
const sightings = [
  { species: 'Fieldfare', site: 'North Hide', count: 12, ringed: false },
  { species: 'Curlew', site: 'Jetty', count: 3, ringed: true },
  { species: 'Fieldfare', site: 'Reed Bed', count: 40, ringed: false },
  { species: 'Bittern', site: 'Reed Bed', count: 1, ringed: true },
  { species: 'Curlew', site: 'North Hide', count: 7, ringed: false },
];
```

Copy it into a file called `build-your-own.js`. We add to that file as we go.

### 2.1 Your own forEach

`forEach` does one thing: run a callback once for each element. Here it is, written from scratch.

```js
function myForEach(array, callback) {
  for (let i = 0; i < array.length; i++) {
    callback(array[i], i);
  }
}

myForEach(sightings, (sighting) => {
  console.log(sighting.species + ' at ' + sighting.site);
});
```

That is the whole thing. A `for` loop you already know, and one line inside it.

<details>
<summary>Optional: reading <code>callback(array[i], i)</code> slowly</summary>

If that line is doing something you could not explain out loud, here it is in pieces.

**`callback` is a parameter holding a function.** When we called `myForEach(sightings, (sighting) => {...})`, JavaScript matched up the arguments with the parameter names in order, exactly as it would with numbers or strings. So inside the function, `array` is the sightings array and `callback` is the arrow function we wrote.

Nothing about it is special because it holds a function. Writing `callback(...)` runs whatever function happens to be in there right now.

**The parentheses run it, and what is inside them are the arguments.** On the first pass of the loop `i` is 0, so that line means:

```js
callback(sightings[0], 0);
```

which is the same as:

```js
callback({ species: 'Fieldfare', site: 'North Hide', count: 12, ringed: false }, 0);
```

**Those arguments land in the callback's parameters, by position.** Our callback declared one parameter:

```js
(sighting) => {
  console.log(sighting.species + ' at ' + sighting.site);
}
```

So `sighting` receives the first argument, the object. The second argument is still sent, but the callback gave it no name, so it is ignored. JavaScript does not complain about being handed more arguments than a function asks for.

This is why the parameter name is yours to choose. All three of these behave identically, because position is what matters, not the name:

```js
myForEach(sightings, (sighting) => console.log(sighting.count));
myForEach(sightings, (item) => console.log(item.count));
myForEach(sightings, (x) => console.log(x.count));
```

And it is why you get the index simply by declaring a second parameter:

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

**The loop controls the timing.** Written out, the five passes do this:

```
i = 0  ->  callback(sightings[0], 0)   logs "Fieldfare at North Hide"
i = 1  ->  callback(sightings[1], 1)   logs "Curlew at Jetty"
i = 2  ->  callback(sightings[2], 2)   logs "Fieldfare at Reed Bed"
i = 3  ->  callback(sightings[3], 3)   logs "Bittern at Reed Bed"
i = 4  ->  callback(sightings[4], 4)   logs "Curlew at North Hide"
```

Five separate calls to the same function, each with different arguments. `myForEach` owns the *when*, your callback owns the *what happens*, and neither needs to know anything about the other.

One last detail. Because the callback is a separate function call each time, a variable declared inside it is created fresh on every pass and is gone by the next one:

```js
myForEach(sightings, (sighting) => {
  const label = sighting.species; // a brand new 'label' every time
  console.log(label);
});
```

That is normal function scope from Module 2, nothing new. It matters in Part 4, where it causes one of the classic bugs.

</details>

Notice what is **not** in `myForEach`: there is no `return` anywhere. That is not an oversight. `forEach` gives you nothing back. It is for *doing* something, not for producing a value.

The real method is written `sightings.forEach(callback)` rather than `myForEach(sightings, callback)`, because it lives on the array, but the behaviour is exactly what you see above.

### 2.2 Your own map

`map` is `forEach` with one change: it keeps whatever the callback returns, and gives you a new array of those results.

```js
function myMap(array, callback) {
  const results = [];
  for (let i = 0; i < array.length; i++) {
    results.push(callback(array[i], i));
  }
  return results;
}

const labels = myMap(sightings, (sighting) => {
  return sighting.species + ' x' + sighting.count;
});

console.log(labels);
// [ 'Fieldfare x12', 'Curlew x3', 'Fieldfare x40', 'Bittern x1', 'Curlew x7' ]
```

Look closely at the `push` line. It pushes **whatever the callback returned**. So if your callback returns nothing, it pushes nothing, which in JavaScript means `undefined`:

```js
// The braces open a function body, and there is no return inside it.
const broken = myMap(sightings, (sighting) => {
  sighting.species; // this value goes nowhere
});
console.log(broken); // [ undefined, undefined, undefined, undefined, undefined ]
```

This is the most common `map` mistake there is, and you can now see exactly why it happens.

One more thing falls out of the code: the new array always has the same length as the old one. `map` changes elements, it never removes them. If you wanted fewer elements, you wanted `filter`.

### 2.3 Your own filter

`filter` keeps the **original** elements, but only the ones your callback approves of.

```js
function myFilter(array, callback) {
  const results = [];
  for (let i = 0; i < array.length; i++) {
    if (callback(array[i], i)) {
      results.push(array[i]); // the element, not the callback's result
    }
  }
  return results;
}

const flocks = myFilter(sightings, (sighting) => sighting.count >= 5);
console.log(flocks.length); // 3
```

Compare the two `push` lines:

- In `myMap` we push `callback(...)`, the callback's **return value**.
- In `myFilter` we push `array[i]`, the **element**, and only use the return value to decide `if`.

That single difference is the whole distinction between the two methods. It also explains the rule you were told in Lesson 3.2: a `filter` callback should return `true` or `false`, because its answer goes into an `if`, while a `map` callback should return the new value, because its answer goes into the array.

The result of `filter` is never longer than what you started with, and it can be empty.

### 2.4 Your own find

`find` is `filter` that stops at the first match and hands back one element instead of an array.

```js
function myFind(array, callback) {
  for (let i = 0; i < array.length; i++) {
    if (callback(array[i], i)) {
      return array[i]; // leaves the whole function immediately
    }
  }
  return undefined; // we reached the end without a match
}

const firstRinged = myFind(sightings, (sighting) => sighting.ringed);
console.log(firstRinged); // { species: 'Curlew', site: 'Jetty', count: 3, ringed: true }

const osprey = myFind(sightings, (sighting) => sighting.species === 'Osprey');
console.log(osprey); // undefined
```

That last line of the function is why `find` gives you `undefined` when nothing matches, and why you must check the result before using it. The early `return` is also why `find` does less work than `filter` on a long array: it stops as soon as it has an answer.

### Exercise 1: build three more (about 20 minutes)

In `build-your-own.js`, write these three yourself. Do not scroll back up while you write them. Test each one against `sightings`.

1. `myEvery(array, callback)` - returns `true` only if the callback returns `true` for **every** element. It should stop and return `false` as soon as one fails.
2. `mySome(array, callback)` - returns `true` if the callback returns `true` for **at least one** element. It should stop as soon as one succeeds.
3. `myCountWhere(array, callback)` - returns how many elements pass the test, as a number.

Test with at least these:

```js
console.log(myEvery(sightings, (s) => s.count > 0));                 // true
console.log(myEvery(sightings, (s) => s.ringed));                    // false
console.log(mySome(sightings, (s) => s.count > 30));                 // true
console.log(mySome(sightings, (s) => s.species === 'Osprey'));       // false
console.log(myCountWhere(sightings, (s) => s.site === 'North Hide')); // 2
```

<details>
<summary>Solution</summary>

```js
function myEvery(array, callback) {
  for (let i = 0; i < array.length; i++) {
    if (!callback(array[i], i)) {
      return false; // one failure is enough, no point continuing
    }
  }
  return true; // nothing failed
}

function mySome(array, callback) {
  for (let i = 0; i < array.length; i++) {
    if (callback(array[i], i)) {
      return true; // one success is enough
    }
  }
  return false;
}

function myCountWhere(array, callback) {
  let total = 0;
  for (let i = 0; i < array.length; i++) {
    if (callback(array[i], i)) {
      total = total + 1;
    }
  }
  return total;
}
```

`myEvery` and `mySome` are near mirror images. In `myEvery` we look for a reason to say no. In `mySome` we look for a reason to say yes. Everything else about them is the same.

If yours works but looks different from mine, that is fine. There is more than one correct loop.

</details>

---

## Part 3: The real methods

From here on, use the built-in versions. They work exactly like the ones you wrote, with one difference in shape: you call them **on** the array with a dot, and you do not pass the array in.

```js
sightings.forEach((sighting) => console.log(sighting.species));
const labels = sightings.map((sighting) => sighting.species);
const flocks = sightings.filter((sighting) => sighting.count >= 5);
const firstRinged = sightings.find((sighting) => sighting.ringed);
```

Here they are with the question each one answers. Learning them by the **question** rather than the name is far more reliable when you are stuck at midnight.

| Question you are asking | Method | What you get back |
| --- | --- | --- |
| Do something with each item | `forEach` | nothing |
| Turn each item into something else | `map` | a new array, same length |
| Keep only the items that match | `filter` | a new array, shorter or the same |
| Get the first item that matches | `find` | one element, or `undefined` |
| Does at least one match | `some` | `true` or `false` |
| Do they all match | `every` | `true` or `false` |

When you are unsure which to use, ask yourself what shape the answer should be. If the answer is a list, you want `map` or `filter`. If it is one thing, you want `find`. If it is a yes or no, you want `some` or `every`.

### some and every

You built these in Exercise 1, so the built-in ones need no explanation. They are worth pointing out because they are easy to forget and they read almost like English:

```js
console.log(sightings.some((s) => s.count > 30));  // true
console.log(sightings.every((s) => s.count > 0));  // true
console.log(sightings.every((s) => s.ringed));     // false
```

### Counting and totalling

Some questions do not have a list as their answer. "How many?" and "how much altogether?" both have a single number as the answer.

For counting, `filter` already does the work, and arrays can tell you their own length:

```js
const ringedCount = sightings.filter((sighting) => sighting.ringed).length;
console.log(ringedCount); // 2
```

For a total, use the `for` loop you already know:

```js
let totalBirds = 0;

for (let i = 0; i < sightings.length; i++) {
  totalBirds = totalBirds + sightings[i].count;
}

console.log(totalBirds); // 63
```

You can do the same with `forEach`, which some people find easier to read:

```js
let totalBirds = 0;

sightings.forEach((sighting) => {
  totalBirds = totalBirds + sighting.count;
});

console.log(totalBirds); // 63
```

Both are correct. Use whichever you find clearer.

The one thing to get right is **where `totalBirds` is declared**. It has to be outside, above the loop. If you declare it inside, it is created fresh on every pass and reset to zero, so nothing ever adds up. That is Bug 5 in the next part.

There is also a built-in method for this, called `reduce`. You will see it in code long before you need to write it, and it is worth recognising the shape: if a line ends in `.reduce(...)`, an array is being boiled down to a single value. The loop above does the same job and is easier to read while you are learning, so stay with it for now.

### Doing two things at once

`map` and `filter` both hand you back an array. So you can put a second method straight onto the end of the first, and it works on the result:

```js
const flockLabels = sightings
  .filter((sighting) => sighting.count >= 5)
  .map((sighting) => sighting.species + ' x' + sighting.count);

console.log(flockLabels); // [ 'Fieldfare x12', 'Fieldfare x40', 'Curlew x7' ]
```

Read it top to bottom as a sentence: take the sightings, keep the ones with five or more birds, then turn each of those into a label. This is called **chaining**.

Two rules, and one warning:

- **Filter first, then map.** There is no point building labels for sightings you are about to throw away.
- **Two steps is plenty** at this stage. If it needs more, use a variable in between and give it a name. Chaining is meant to make code easier to read, not to prove a point.
- **Nothing can follow `forEach`.** It gives you nothing back, so there is nothing for the next method to work on. If you want to carry on afterwards, you wanted `map` or `filter`.

---

## Part 4: The five mistakes everybody makes

These are not exotic. They are the ones you will actually make this month.

Work out what each one does **before** you run it, then fix it. Put them in a file called `debug-me.js` with the `sightings` array at the top.

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
console.log('We saw ' + osprey.count + ' ospreys');

// Bug 4
const bittern = sightings.filter((sighting) => sighting.species === 'Bittern');
console.log('The bittern was at ' + bittern.site);

// Bug 5
sightings.forEach((sighting) => {
  let total = 0;
  total = total + sighting.count;
  console.log(total);
});
```

<details>
<summary>Answers and explanations</summary>

**Bug 1** logs `[ undefined, undefined, undefined, undefined, undefined ]`.

The braces open a function body, and a function body with no `return` gives back `undefined`. `map` collects five of them. Either add the `return`, or drop the braces so the arrow function returns automatically:

```js
const speciesList = sightings.map((sighting) => sighting.species);
```

**Bug 2** logs `undefined`.

`forEach` gives nothing back, whatever the callback returns. The comparison is worked out and then thrown away. This person wanted `filter`:

```js
const bigOnes = sightings.filter((sighting) => sighting.count > 10);
```

**Bug 3** crashes: `TypeError: Cannot read properties of undefined (reading 'count')`.

There is no Osprey, so `find` gave back `undefined`, and `undefined` has no `.count`. Always check before you use the result:

```js
const osprey = sightings.find((sighting) => sighting.species === 'Osprey');

if (osprey) {
  console.log('We saw ' + osprey.count + ' ospreys');
} else {
  console.log('No ospreys today');
}
```

**Bug 4** logs `The bittern was at undefined`.

This one is sneaky because nothing crashes. `filter` always gives back an **array**, even when only one thing matched. An array has no `.site` property. You either want `find` instead:

```js
const bittern = sightings.find((sighting) => sighting.species === 'Bittern');
console.log('The bittern was at ' + bittern.site);
```

or, if you really do want the array, reach into it with `[0]`. Prefer `find`, since it says what you mean.

**Bug 5** logs `12`, `3`, `40`, `1`, `7` instead of a total.

`total` is declared **inside** the callback, so it is created fresh and set back to `0` on each of the five calls. Nothing accumulates. Move it outside:

```js
let total = 0;

sightings.forEach((sighting) => {
  total = total + sighting.count;
});

console.log(total); // 63
```

Watch the keyword when you move it. `const total = 0` followed by `total = total + ...` gives `TypeError: Assignment to constant variable`, which is usually the next thing people hit.

</details>

---

## Part 5: Practice

New data. This is the job log from a community repair workshop, where volunteers help people mend things instead of throwing them away.

```js
const repairs = [
  { id: 'r1', item: 'Toaster', volunteer: 'Iris', minutes: 45, fixed: true },
  { id: 'r2', item: 'Desk lamp', volunteer: 'Omar', minutes: 20, fixed: true },
  { id: 'r3', item: 'Bicycle', volunteer: 'Iris', minutes: 90, fixed: false },
  { id: 'r4', item: 'Kettle', volunteer: 'Sam', minutes: 30, fixed: true },
  { id: 'r5', item: 'Radio', volunteer: 'Omar', minutes: 75, fixed: false },
  { id: 'r6', item: 'Sewing machine', volunteer: 'Sam', minutes: 60, fixed: true },
];
```

Put it in `repairs-practice.js`. Before each task, decide what **shape** the answer should be. That tells you the method.

### Exercise 2: one method at a time (about 20 minutes)

1. Log a line for each repair, like `Toaster - Iris - 45 min`.
2. `itemNames` - an array of just the item names.
3. `unfixed` - the repairs that were not fixed.
4. Find the repair with the id `r4` and log its item name.
5. Find the repair for an item called `Hairdryer`, and log a sensible message when there is not one.
6. Did any repair take more than 80 minutes? Answer with `true` or `false`.
7. Did every repair take at least 15 minutes? Answer with `true` or `false`.
8. How many repairs did Iris do?

<details>
<summary>Solution</summary>

```js
// 1 - doing something with each one, so forEach
repairs.forEach((repair) => {
  console.log(repair.item + ' - ' + repair.volunteer + ' - ' + repair.minutes + ' min');
});

// 2 - a list of new values, so map
const itemNames = repairs.map((repair) => repair.item);
console.log(itemNames);
// [ 'Toaster', 'Desk lamp', 'Bicycle', 'Kettle', 'Radio', 'Sewing machine' ]

// 3 - a shorter list of the same things, so filter
const unfixed = repairs.filter((repair) => !repair.fixed);
console.log(unfixed.length); // 2

// 4 - one thing, so find
const r4 = repairs.find((repair) => repair.id === 'r4');
console.log(r4.item); // Kettle

// 5 - one thing that might not be there, so find plus a check
const hairdryer = repairs.find((repair) => repair.item === 'Hairdryer');

if (hairdryer) {
  console.log('Found: ' + hairdryer.id);
} else {
  console.log('No hairdryer in the log');
}

// 6 - a yes or no about at least one, so some
console.log(repairs.some((repair) => repair.minutes > 80)); // true

// 7 - a yes or no about all of them, so every
console.log(repairs.every((repair) => repair.minutes >= 15)); // true

// 8 - a number, so filter and then ask how long the result is
const irisCount = repairs.filter((repair) => repair.volunteer === 'Iris').length;
console.log(irisCount); // 2
```

Task 4 has no check on the result, because we know `r4` is in there. Task 5 has one, because we do not. In real code, where the data comes from somewhere else, always check.

</details>

### Exercise 3: two methods together (about 20 minutes)

Each of these is one `filter` followed by one `map`. Filter first.

1. `fixedLabels` - the successful repairs, as strings like `Toaster (45 min)`.
2. `quickWins` - the item names of repairs that were fixed **and** took under 50 minutes.
3. `longJobs` - the repairs of 60 minutes or more, as strings like `Bicycle: 90 min`.
4. `omarItems` - the item names of everything Omar worked on.

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
const longJobs = repairs
  .filter((repair) => repair.minutes >= 60)
  .map((repair) => repair.item + ': ' + repair.minutes + ' min');
console.log(longJobs); // [ 'Bicycle: 90 min', 'Radio: 75 min', 'Sewing machine: 60 min' ]

// 4
const omarItems = repairs
  .filter((repair) => repair.volunteer === 'Omar')
  .map((repair) => repair.item);
console.log(omarItems); // [ 'Desk lamp', 'Radio' ]
```

If a chain confuses you, split it in two and log the middle:

```js
const fixed = repairs.filter((repair) => repair.fixed);
console.log(fixed); // check this looks right before going further

const fixedLabels = fixed.map((repair) => repair.item + ' (' + repair.minutes + ' min)');
```

That is not a worse solution. It is often the better one, and it is much easier to debug.

</details>

### Exercise 4: counting and totalling (about 15 minutes)

These all have a single value as the answer.

1. `totalMinutes` - the total time across all repairs. (Expected: 320.)
2. `fixedCount` - how many repairs succeeded. (Expected: 4.)
3. `fixedMinutes` - the total time spent on the repairs that succeeded. (Expected: 155.)
4. `averageMinutes` - the average time per repair, to one decimal place. Use `.toFixed(1)` from Lesson 3.1. (Expected: 53.3.)
5. `busiest` - the repair that took the longest. Use a loop and a variable holding the winner so far.

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
const fixedCount = repairs.filter((repair) => repair.fixed).length;
console.log(fixedCount); // 4

// 3 - filter first, then total up what is left
let fixedMinutes = 0;

repairs
  .filter((repair) => repair.fixed)
  .forEach((repair) => {
    fixedMinutes = fixedMinutes + repair.minutes;
  });
console.log(fixedMinutes); // 155

// 4
const averageMinutes = (totalMinutes / repairs.length).toFixed(1);
console.log(averageMinutes); // "53.3"

// 5 - start with the first one as the winner, then look for a better one
let busiest = repairs[0];

repairs.forEach((repair) => {
  if (repair.minutes > busiest.minutes) {
    busiest = repair;
  }
});
console.log(busiest.item); // Bicycle
```

Task 3 puts a `forEach` on the end of a chain, which is allowed because nothing comes after it. That is the only place `forEach` can ever sit.

Task 4 gives back a string, because `.toFixed()` always does. That is fine for logging. If you need to do more sums with it, wrap it in `Number()`.

Task 5 is the same shape as a running total, except the variable holds an object instead of a number, and it only changes when we find something better.

</details>

---

## Before you move on

Whatever you write from here on, check it against this list. Every line is one of the five bugs turned into a habit.

- Every `map` callback returns something.
- Every `find` result is checked before it is used, unless you are certain it is there.
- Nothing is chained after a `forEach`.
- Totals and counters are declared above the loop, not inside it, and with `let`.
- When you meant one thing, you used `find` and not `filter`.

## Quick reference

```js
// Do something with each item. Gives nothing back.
items.forEach((item) => console.log(item));

// Turn each item into something else. Same length out.
const changed = items.map((item) => item.name);

// Keep the matching items. Shorter or the same.
const matching = items.filter((item) => item.active);

// The first match, or undefined. Check it before using it.
const one = items.find((item) => item.id === wanted);

// Yes or no.
const any = items.some((item) => item.active);
const all = items.every((item) => item.active);

// How many.
const howMany = items.filter((item) => item.active).length;

// A total. The variable goes above the loop.
let total = 0;
items.forEach((item) => {
  total = total + item.price;
});

// Two steps. Filter first.
const labels = items
  .filter((item) => item.active)
  .map((item) => item.name);
```

## References

All on MDN Web Docs, which is the reference working developers actually use. A tip on reading it: skip past the syntax block and go straight to the Examples, then work backwards.

1. [Callback function](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function) - the short glossary definition.
2. [First-class function](https://developer.mozilla.org/en-US/docs/Glossary/First-class_Function) - the idea behind all of Part 1.
3. [Arrow function expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) - especially the part about returning without braces.
4. [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) - every array method there is. Far more than you need today.
5. [Array.prototype.forEach()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)
6. [Array.prototype.map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
7. [Array.prototype.filter()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)
8. [Array.prototype.find()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find)
9. [Array.prototype.some()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/some)
10. [Array.prototype.every()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every)
