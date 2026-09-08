# Unpacking data, and getting out of one file

**Estimated time:** about 2 hours for the core path, plus 60 to 90 minutes for the self study task  
**Prerequisites:** Lessons 3.1 and 3.2. You should be comfortable with objects, arrays, arrow functions and `forEach` / `map` / `filter` / `find` before you start.

## How this lesson is different

The regular lessons walk through this material feature by feature: here is `Object.keys()`, here is `Object.entries()`, here is destructuring, here is the spread operator, here is `export`, here is `import`. Each one gets an introduction, an example and an exercise. That works, and you should still do those lessons.

This one is built around three ideas instead of seven features. Everything in Lesson 3.3 collapses into two sentences, and everything in Lesson 3.4 collapses into one:

1. **The shape goes on the left.** Destructuring is not new syntax to memorise. It is you drawing a picture of the data you want, on the left-hand side of the `=`.
2. **Three dots, two directions.** The same `...` either collects loose things into one container, or pours one container out into a new one. Which one it does depends entirely on which side of the `=` it sits.
3. **A file is a room with a door.** A module is not a special kind of file. It is an ordinary file where everything is private unless you deliberately open a door.

There is also a change of order. The regular lessons teach `Object.entries()` first and then say "we will use destructuring on this in the next lesson". We are going to do it the other way round, because `Object.entries(x).forEach(([key, value]) => ...)` is genuinely confusing until you already know what `[key, value]` in a parameter list means. So destructuring comes first, and then the object methods land as a two-minute idea instead of a mystery.

Every example and exercise here uses different data from the regular lessons, so you cannot pattern-match your way through. You will have to think about the shapes.

---

# Part 1: The shape goes on the left

## Start with the tedious version

Here is a reading from a small coastal weather station.

```js
const reading = {
  station: 'Utsira',
  tempC: 6.4,
  windMs: 14.2,
  observedAt: '06:00'
};
```

Suppose you want each of those values in its own variable, so you can use them without typing `reading.` over and over. The way you already know:

```js
const station = reading.station;
const tempC = reading.tempC;
const windMs = reading.windMs;
const observedAt = reading.observedAt;

console.log(station, tempC, windMs, observedAt);
```

Nothing wrong with that. It works. But look at what you actually typed: every line says the same word three times. `station` is on the left, `station` is on the right, and the only real information in the line is "yes, I want that one".

Read those four lines again and notice something. Stacked up, they are starting to look like an object.

## Fold it up

Destructuring lets you write the four lines as one:

```js
const { station, tempC, windMs, observedAt } = reading;

console.log(station, tempC, windMs, observedAt);
```

That single line does exactly what the four lines did. Nothing more, nothing less. It creates four ordinary `const` variables.

Here is the mental model that makes the rest of this easy. Look at the left-hand side on its own:

```js
{ station, tempC, windMs, observedAt }
```

That looks like an object. It is not an object. It is a **picture of the object you are pulling from**, with the parts you want named and the parts you do not want left out. You are saying to JavaScript: "the thing on the right has a shape roughly like this; give me the bits I have drawn."

That is why you can leave things out. This is perfectly legal:

```js
const { tempC } = reading;

console.log(tempC); // 6.4
```

You drew a picture with one thing in it, so you got one thing. The other three properties are still sitting in `reading`, untouched.

<details>

<summary>Rabbit hole: why does the curly brace mean two different things?</summary>

This is the single most confusing thing about destructuring, and it is worth thirty seconds.

`{ }` in JavaScript means "object literal" when it is on the **right** of an `=`:

```js
const point = { x: 1, y: 2 }; // building an object
```

The same `{ }` means "pattern" when it is on the **left** of an `=`:

```js
const { x, y } = point; // taking an object apart
```

Same characters, opposite jobs. On the right you are packing; on the left you are unpacking. Once you have seen that, the odd-looking bits later in this lesson stop being odd. Square brackets work the same way: `[1, 2, 3]` on the right builds an array, `[a, b, c]` on the left takes one apart.

If you ever find yourself staring at a line and unsure what it does, look for the `=` and ask which side you are on.

</details>

## When the name is wrong: renaming

Sometimes the property name is not the name you want in your code. Maybe it is too long, or too vague, or it clashes with a variable you already have.

```js
const { observedAt: time } = reading;

console.log(time); // "06:00"
```

Read that as: "find `observedAt`, and call it `time` here."

The direction trips almost everyone up the first time, because it looks like an object literal where the value goes on the right of the colon. In a pattern, the roles are swapped: **the property name is on the left of the colon, and your new variable name is on the right.**

One thing to be clear about: `observedAt` is not a variable now. Only `time` is.

```js
const { observedAt: time } = reading;

console.log(time);        // "06:00"
console.log(observedAt);  // ReferenceError: observedAt is not defined
```

## When the property is missing: defaults

Weather stations do not all report the same things. This one has no barometer, so there is no `pressureHpa` property at all. Pull it out anyway and you get `undefined`, which will quietly poison any sum or comparison you do later.

You can supply a fallback with `=` inside the pattern:

```js
const { tempC, pressureHpa = 1013 } = reading;

console.log(tempC);       // 6.4
console.log(pressureHpa); // 1013
```

The default only fires when the property is missing or is literally `undefined`. It does **not** fire for `0`, `false` or `''`, which is exactly what you want. A station reporting `tempC: 0` is reporting real information, and you would be furious if JavaScript overwrote it with a guess.

```js
const quiet = { windMs: 0 };
const { windMs = 5 } = quiet;

console.log(windMs); // 0, not 5
```

You can combine renaming and defaulting, though you will not need it often:

```js
const { pressureHpa: pressure = 1013 } = reading;
```

## The best place to do this: the parameter list

Here is where destructuring earns its keep. A function that takes an object can draw its picture right in the parameter list.

The version you already know how to write:

```js
function describeReading(readingObject) {
  return readingObject.station + ': ' + readingObject.tempC + ' degrees';
}
```

The same function, destructured:

```js
function describeReading({ station, tempC }) {
  return station + ': ' + tempC + ' degrees';
}

console.log(describeReading(reading)); // "Utsira: 6.4 degrees"
```

You still call it exactly the same way, by passing one whole object. But now the function signature is a list of its own ingredients. Anyone reading the first line knows this function cares about `station` and `tempC` and nothing else, without reading the body.

This works with arrow functions too, which is where you will meet it most often:

```js
const stations = [
  { station: 'Utsira', tempC: 6.4 },
  { station: 'Slettnes', tempC: -2.1 },
  { station: 'Torungen', tempC: 8.0 }
];

const lines = stations.map(({ station, tempC }) => station + ': ' + tempC);

console.log(lines);
// [ 'Utsira: 6.4', 'Slettnes: -2.1', 'Torungen: 8' ]
```

Compare that to `stations.map((s) => s.station + ': ' + s.tempC)`. Both are fine. The destructured one tells you more.

## Arrays: position instead of name

Arrays have no property names, so there is nothing to write down except position. The pattern uses square brackets, and the variable names are yours to choose, because JavaScript is matching by **slot**, not by name.

```js
const position = [59.31, 4.87];

const [latitude, longitude] = position;

console.log(latitude);  // 59.31
console.log(longitude); // 4.87
```

Names are entirely up to you here. `const [a, b] = position;` works identically. That freedom is also the danger: if you get the order wrong, nothing complains, you just end up somewhere in the wrong hemisphere. Array destructuring is at its best when the order is genuinely fixed and obvious, like a coordinate pair or a `[key, value]` pair, and at its worst when someone has crammed five unrelated things into a list.

You can skip a slot by leaving a gap:

```js
const forecast = [6.4, 6.9, 7.2, 7.0];

const [now, , inTwoHours] = forecast;

console.log(now);        // 6.4
console.log(inTwoHours); // 7.2
```

That lonely comma with nothing before it is not a typo. It means "there is a slot here, I do not want it".

<details>

<summary>Rabbit hole: swapping two variables in one line</summary>

Array destructuring gives you the classic swap for free. No temporary variable:

```js
let high = 3;
let low = 9;

[high, low] = [low, high];

console.log(high, low); // 9 3
```

The right-hand side builds a throwaway array `[9, 3]`, and the left-hand side immediately tears it apart into the two existing variables. Note there is no `const` or `let` on that line, because both variables already exist and we are only reassigning them, which also means they must have been declared with `let`, not `const`.

One warning, because it bites people. A line starting with `[` can be misread by JavaScript as continuing the previous line. If the line above does not end in a semicolon, you can get a baffling error. Either keep your semicolons, or start the line with one:

```js
;[high, low] = [low, high];
```

This is the one place in JavaScript where leaving off a semicolon genuinely breaks working code, which is a decent argument for just always writing them.

</details>

## Exercise 1

Create a file called `extra-destructuring.js` and copy in this data:

```js
const gauge = {
  river: 'Otra',
  levelCm: 214,
  flowM3s: 88.5,
  measuredAt: '14:00',
  trend: 'rising'
};

const range = [180, 214, 260];
```

1. Use one line of object destructuring to create the variables `river` and `levelCm`. Log both.
2. Use destructuring to pull `measuredAt` out, but call the variable `time`. Log it.
3. The gauge has no `temperatureC` property. Use destructuring with a default value of `4` to create a `temperatureC` variable. Log it to confirm the default was used.
4. Use array destructuring on `range` to create `minimum` and `maximum`, skipping the middle value. Log both.
5. Write a function `describeGauge` that takes **one whole object** as its argument, but destructures `river`, `levelCm` and `trend` in its parameter list. It should return a string like `"Otra: 214 cm and rising"`. Call it with `gauge` and log the result.

<details>

<summary>Solution to Exercise 1</summary>

```js
const gauge = {
  river: 'Otra',
  levelCm: 214,
  flowM3s: 88.5,
  measuredAt: '14:00',
  trend: 'rising'
};

const range = [180, 214, 260];

// 1. Two properties, one line. The names on the left must match the keys.
const { river, levelCm } = gauge;
console.log(river, levelCm); // Otra 214

// 2. Property name on the left of the colon, new variable name on the right.
const { measuredAt: time } = gauge;
console.log(time); // "14:00"

// 3. temperatureC does not exist on gauge, so the default is used.
const { temperatureC = 4 } = gauge;
console.log(temperatureC); // 4

// 4. The empty slot between the commas skips the middle value.
const [minimum, , maximum] = range;
console.log(minimum, maximum); // 180 260

// 5. The function still receives one object. The pattern in the parameter
// list unpacks it on the way in.
function describeGauge({ river, levelCm, trend }) {
  return river + ': ' + levelCm + ' cm and ' + trend;
}

console.log(describeGauge(gauge)); // "Otra: 214 cm and rising"
```

A note on step 5: the variables `river` and `levelCm` inside `describeGauge` are completely separate from the ones you made in step 1. They live inside the function and vanish when it returns. That is not a clash, it is just scope doing its job.

</details>

---

# Part 2: An object is not an array, until you convert it

## The problem, stated plainly

Here is a bakery's tally of unsold loaves at closing time.

```js
const leftovers = {
  sourdough: 3,
  rye: 0,
  baguette: 7,
  cinnamonBun: 12
};
```

You have a toolkit of excellent array methods. Try to use one:

```js
leftovers.forEach((count) => console.log(count));
// TypeError: leftovers.forEach is not a function
```

That error is not a bug in your thinking. It is a correct description of the world. `forEach`, `map`, `filter` and `find` are methods on **arrays**. Objects do not have them, and for a decent reason: those methods all rely on there being a first item, a second item, a third item, and objects are not built around order in that way.

So you have two options. Either learn a completely separate set of tools for objects, or convert the object into an array and keep using the tools you already have.

JavaScript gives you three converters, and that is the whole of Lesson 3.3's first half.

## The three converters

```js
console.log(Object.keys(leftovers));
// [ 'sourdough', 'rye', 'baguette', 'cinnamonBun' ]

console.log(Object.values(leftovers));
// [ 3, 0, 7, 12 ]

console.log(Object.entries(leftovers));
// [ [ 'sourdough', 3 ], [ 'rye', 0 ], [ 'baguette', 7 ], [ 'cinnamonBun', 12 ] ]
```

Three things worth noticing straight away.

**They are written `Object.keys(x)`, not `x.keys()`.** You call them on the word `Object` itself and hand your object over as an argument. This is different from every array method you have met, and getting it backwards is the most common mistake in this whole section.

**Keys always come out as strings.** Even if you wrote them without quotes, even if they look like numbers.

**`Object.entries()` gives you an array of little two-item arrays.** Position 0 is the key, position 1 is the value. Always in that order.

Now you have arrays, and your whole toolkit works again.

```js
// How many kinds of bread are left over at all?
console.log(Object.keys(leftovers).length); // 4

// How many loaves in total?
let totalLoaves = 0;
Object.values(leftovers).forEach((count) => {
  totalLoaves = totalLoaves + count;
});
console.log(totalLoaves); // 22
```

## Entries plus destructuring: the pattern you will actually use

`Object.keys()` gives you names without amounts. `Object.values()` gives you amounts without names. Most of the time you want both at once, which is why `Object.entries()` is the one you will reach for.

But an entry is an array like `['baguette', 7]`, and working with it by index is grim:

```js
Object.entries(leftovers).forEach((entry) => {
  console.log(entry[0] + ': ' + entry[1]);
});
```

That works, and if you are ever unsure what is going on, write it that way first. But you learned the fix in Part 1. An entry is an array with a fixed order, which is precisely the case where array destructuring shines. So put the pattern in the parameter list:

```js
Object.entries(leftovers).forEach(([name, count]) => {
  console.log(name + ': ' + count);
});

// sourdough: 3
// rye: 0
// baguette: 7
// cinnamonBun: 12
```

Look carefully at the parameter list: `([name, count])`. The round brackets are the function's parameters, as always. The square brackets inside are a destructuring pattern applied to the one argument being passed in. `forEach` hands over `['baguette', 7]`, and the pattern immediately splits it into `name` and `count`.

This is the line the regular lesson shows you before you have met destructuring. Now it should read as ordinary.

And because entries are just an array, the rest of your toolkit comes along:

```js
// Which breads sold out completely?
const soldOut = Object.entries(leftovers)
  .filter(([name, count]) => count === 0)
  .map(([name]) => name);

console.log(soldOut); // [ 'rye' ]

// Is there anything we have far too much of?
const glut = Object.entries(leftovers).find(([name, count]) => count > 10);

console.log(glut); // [ 'cinnamonBun', 12 ]
```

Note `([name])` in that `map`. The pattern only mentions the first slot, so you only get the first slot. Same idea as leaving properties out of an object pattern.

## What about `for...in`?

There is an older loop built specifically for objects:

```js
for (const key in leftovers) {
  console.log(key + ': ' + leftovers[key]);
}
```

It gives you the keys, one at a time, and you look up the value yourself with bracket notation. Note that it has to be `leftovers[key]` and not `leftovers.key`, because `leftovers.key` would go looking for a property literally named "key".

You will see `for...in` in older code and it is worth recognising. But there are two reasons this lesson does not lean on it. The first is that it hands you the key and makes you fetch the value, which is a step backwards from entries. The second is that in certain situations it can hand you properties that were never in your object, inherited from elsewhere in JavaScript's machinery. You are unlikely to hit that as a beginner, but it is the reason most style guides steer you towards `Object.entries()`.

Rule of thumb: **`for...in` is fine, `Object.entries()` is better, use whichever you can read faster today and drift towards entries.**

<details>

<summary>Rabbit hole: what order do the keys come out in?</summary>

You might reasonably expect keys to come out in the order you wrote them. Mostly they do, and for the objects you will write in this course, they always will.

The exception is keys that look like whole numbers. Those get sorted numerically and jump to the front, ahead of everything else:

```js
const scores = { banana: 1, 10: 'ten', apple: 2, 2: 'two' };

console.log(Object.keys(scores));
// [ '2', '10', 'banana', 'apple' ]
```

The `2` and `10` came first and in numeric order, even though they were written second and fourth. The string keys kept their insertion order behind them.

You will also notice that `'2'` and `'10'` are strings in the output. Object keys are always strings, no matter what you typed. Writing `{ 2: 'two' }` stores it under the key `'2'`.

The practical lesson: if the order of your data matters, put it in an **array**, which guarantees order. An object is for looking things up by name. Do not use one as a numbered list.

</details>

## Exercise 2

Create `extra-objects.js` with this data. It is the number of hours each room in a village hall was booked for last week.

```js
const hoursBooked = {
  mainHall: 14,
  kitchen: 3,
  smallRoom: 9,
  library: 0,
  cellar: 0
};
```

1. Log how many rooms there are, without counting them by hand.
2. Calculate and log the total number of hours booked across all rooms. Use `Object.values()` and a `forEach` with a running total.
3. Use `Object.entries()` with `forEach` and destructuring in the parameter list to log one line per room, in the format `"mainHall: 14 hours"`.
4. Build an array called `unused` containing only the **names** of rooms with 0 hours booked. Log it. You should get `[ 'library', 'cellar' ]`.
5. Use `find` on the entries to get the first room with more than 10 hours booked. Log just its name.

<details>

<summary>Solution to Exercise 2</summary>

```js
const hoursBooked = {
  mainHall: 14,
  kitchen: 3,
  smallRoom: 9,
  library: 0,
  cellar: 0
};

// 1. Object.keys gives an array, and arrays have .length.
console.log(Object.keys(hoursBooked).length); // 5

// 2. Object.values gives an array of just the numbers, which we can add up.
let totalHours = 0;
Object.values(hoursBooked).forEach((hours) => {
  totalHours = totalHours + hours;
});
console.log(totalHours); // 26

// 3. Each entry is ['mainHall', 14]. The pattern splits it on the way in.
Object.entries(hoursBooked).forEach(([room, hours]) => {
  console.log(room + ': ' + hours + ' hours');
});

// 4. filter keeps the entries we want, then map throws away the count
// and keeps only the name. The map pattern only mentions slot 0.
const unused = Object.entries(hoursBooked)
  .filter(([room, hours]) => hours === 0)
  .map(([room]) => room);

console.log(unused); // [ 'library', 'cellar' ]

// 5. find returns the whole entry, so we destructure the result to get
// at the name. Remember find returns undefined if nothing matches, so
// destructuring the result directly would crash on an empty result.
const busiest = Object.entries(hoursBooked).find(([room, hours]) => hours > 10);

if (busiest) {
  const [busiestRoom] = busiest;
  console.log(busiestRoom); // "mainHall"
}
```

Step 5 is worth a second look. It is tempting to write this:

```js
const [busiestRoom] = Object.entries(hoursBooked).find(([room, hours]) => hours > 10);
```

That works when something matches. When nothing matches, `find` returns `undefined`, and trying to destructure `undefined` throws `TypeError: undefined is not iterable`. Checking with `if` first is not being timid, it is being correct.

</details>

---

# Part 3: Three dots, two directions

You have already met `...` once in this lesson without me naming it. Now let us name it properly, because it is one symbol doing two opposite jobs and the only thing separating them is which side of the `=` it is on.

- On the **left**, `...` **collects**. It sweeps up whatever is left over into one new container. This is called the **rest** syntax.
- On the **right**, `...` **pours**. It empties an existing container out into a new one. This is called the **spread** operator.

If you can hold on to "left collects, right pours", the rest of this section is mostly examples.

## Collecting (rest)

Here is a list of the crew signed up for a boat trip, in the order they signed up.

```js
const crew = ['Ingrid', 'Marta', 'Kwame', 'Ola', 'Rosa'];

const [skipper, ...passengers] = crew;

console.log(skipper);    // "Ingrid"
console.log(passengers); // [ 'Marta', 'Kwame', 'Ola', 'Rosa' ]
```

`skipper` took the first slot as normal. `...passengers` said "and put everything still remaining into a new array called `passengers`".

Two rules. The rest element must be **last**, because "everything remaining" makes no sense in the middle. And it always produces an **array**, even when there is nothing left to collect:

```js
const soloCrew = ['Ingrid'];
const [alone, ...others] = soloCrew;

console.log(others); // [] , an empty array, not undefined
```

That last detail matters more than it looks. An empty array is safe to `forEach` over, safe to check `.length` on, safe to pass along. You never have to guard against it being missing.

## Pouring (spread)

Now the same three dots, on the other side.

```js
const original = ['Ingrid', 'Marta'];

const copy = [...original];

console.log(copy);              // [ 'Ingrid', 'Marta' ]
console.log(original === copy); // false
```

`[...original]` means "make a new array, and pour the contents of `original` into it". The result looks identical but is a **separate array**. That `false` is the entire point, and we will come back to why in a moment.

Pouring lets you build new arrays out of old ones without disturbing the old ones:

```js
const morning = ['Ingrid', 'Marta'];
const afternoon = ['Kwame', 'Ola'];

const everyone = [...morning, ...afternoon];
console.log(everyone); // [ 'Ingrid', 'Marta', 'Kwame', 'Ola' ]

const withLateArrival = [...everyone, 'Rosa'];
console.log(withLateArrival); // [ 'Ingrid', 'Marta', 'Kwame', 'Ola', 'Rosa' ]

console.log(everyone.length); // still 4
```

Compare `[...everyone, 'Rosa']` with `everyone.push('Rosa')`. Both get Rosa onto a list. But `push` **changes** `everyone`, and spread **leaves it alone and hands you a new list**. Neither is wrong. But when a bug has you baffled because an array has mysteriously grown, it is nearly always a `push` you forgot about, and nearly never a spread.

## Pouring objects, and the "last one wins" rule

Spread works on objects too, and this is where it becomes genuinely useful rather than just tidy.

Here are the standard options on a rented mountain cabin, and what one particular group actually asked for.

```js
const cabinDefaults = {
  beds: 4,
  firewood: true,
  boat: false,
  dogsAllowed: false
};

const requested = {
  beds: 6,
  dogsAllowed: true
};

const booking = { ...cabinDefaults, ...requested };

console.log(booking);
// { beds: 6, firewood: true, boat: false, dogsAllowed: true }
```

Read what happened. Every default was poured in. Then everything requested was poured in on top. Where both objects had the same key, **the one poured in later won**.

That is the whole rule, and it makes order everything. Swap the two and you get a booking nobody asked for:

```js
const wrong = { ...requested, ...cabinDefaults };

console.log(wrong);
// { beds: 4, dogsAllowed: false, firewood: true, boat: false }
```

The defaults landed last, so the defaults won, and the group's requests were silently thrown away. No error. Just a family arriving to four beds and a sign saying no dogs.

**Defaults first, specifics last.** Write it on a sticky note.

The same rule gives you the one-line update. You do not have to spread a whole second object; you can just name a key after the spread:

```js
const upgraded = { ...booking, boat: true };

console.log(upgraded.boat); // true
console.log(booking.boat);  // false, the original is untouched
```

That pattern, "copy everything, then override one thing", is one of the most common lines in modern JavaScript. You will write it constantly once you start working with data that comes back from a server.

<details>

<summary>Rabbit hole: collecting from an object too</summary>

Rest works in object patterns as well, and it is the neat way to say "everything except".

```js
const gauge = {
  river: 'Otra',
  levelCm: 214,
  flowM3s: 88.5,
  trend: 'rising'
};

const { river, ...measurements } = gauge;

console.log(river);         // "Otra"
console.log(measurements);  // { levelCm: 214, flowM3s: 88.5, trend: 'rising' }
```

`river` was named, so it was taken out individually. `...measurements` collected everything not already named into a brand new object.

The classic use is stripping a field you must not pass on. If you have a user object with a password in it and you want to log the rest of it safely, `const { password, ...safeToLog } = user;` does it in one line.

</details>

## The one real trap: copies are only skin deep

Earlier, `original === copy` was `false` and I said that was the point. Here is the catch.

```js
const stationA = {
  name: 'Utsira',
  contact: { phone: '52 00 00 00' }
};

const stationB = { ...stationA };

stationB.name = 'Slettnes';
console.log(stationA.name); // "Utsira" , fine, unaffected

stationB.contact.phone = '78 00 00 00';
console.log(stationA.contact.phone); // "78 00 00 00" , changed!
```

Changing `name` on the copy left the original alone, exactly as promised. Changing the phone number inside `contact` changed **both**.

The reason is that spread copies each property one level down, and no further. `name` held a string, so a fresh string went into the copy. But `contact` did not hold an object, it held a **pointer to** an object sitting elsewhere in memory. Copying the pointer gives you a second signpost to the same house. Walk down either signpost, redecorate, and both signposts now lead to a redecorated house.

This is called a **shallow copy**. Every copying technique you will meet for a long while is shallow.

The practical advice for now: this bites you when your objects have objects or arrays inside them. If yours are flat, spread away without a second thought. If they are nested, be aware that the nested parts are shared, and be suspicious when a change shows up somewhere you did not expect.

## Exercise 3

Create `extra-spread.js` and copy in this data. A community minibus has a standard configuration and a booking that wants to change some of it.

```js
const busDefaults = {
  seats: 16,
  wheelchairSpace: false,
  driverIncluded: true,
  luggageTrailer: false
};

const bookingRequest = {
  wheelchairSpace: true,
  luggageTrailer: true
};

const waitingList = ['Sunniva', 'Petter', 'Amina', 'Jonas', 'Leila'];
```

1. Create a `confirmedBus` object that starts from the defaults and applies the request on top. Log it. Check that `driverIncluded` survived.
2. Now deliberately create `brokenBus` with the two objects in the wrong order. Log it, and write a comment explaining in your own words which property came out wrong and why.
3. From `confirmedBus`, create `finalBus` which is the same but with `seats` set to `20`. Do it in a single line. Log both `finalBus.seats` and `confirmedBus.seats` to prove the original was not changed.
4. Use array destructuring with rest to split `waitingList` into `firstInLine` and `stillWaiting`. Log both.
5. Create `updatedList`, which is `waitingList` plus `'Tor'` on the end, **without** changing `waitingList`. Log `updatedList.length` and `waitingList.length` to prove it worked.

<details>

<summary>Solution to Exercise 3</summary>

```js
const busDefaults = {
  seats: 16,
  wheelchairSpace: false,
  driverIncluded: true,
  luggageTrailer: false
};

const bookingRequest = {
  wheelchairSpace: true,
  luggageTrailer: true
};

const waitingList = ['Sunniva', 'Petter', 'Amina', 'Jonas', 'Leila'];

// 1. Defaults first, specifics last. The request overrides the two keys it
// mentions, and every other default survives untouched.
const confirmedBus = { ...busDefaults, ...bookingRequest };
console.log(confirmedBus);
// { seats: 16, wheelchairSpace: true, driverIncluded: true, luggageTrailer: true }

// 2. Wrong order. The defaults are poured in last, so they win.
const brokenBus = { ...bookingRequest, ...busDefaults };
console.log(brokenBus);
// { wheelchairSpace: false, luggageTrailer: false, seats: 16, driverIncluded: true }
//
// wheelchairSpace and luggageTrailer both came out false. The request set
// them to true first, but busDefaults was spread afterwards and overwrote
// them with its own false values. Later always wins.

// 3. Copy everything, then name one key to override it.
const finalBus = { ...confirmedBus, seats: 20 };
console.log(finalBus.seats);     // 20
console.log(confirmedBus.seats); // 16, the original is unchanged

// 4. One named slot, then rest collects everything remaining into an array.
const [firstInLine, ...stillWaiting] = waitingList;
console.log(firstInLine);  // "Sunniva"
console.log(stillWaiting); // [ 'Petter', 'Amina', 'Jonas', 'Leila' ]

// 5. Pour the old array into a new one and add to the end.
// Using waitingList.push('Tor') would have changed the original.
const updatedList = [...waitingList, 'Tor'];
console.log(updatedList.length); // 6
console.log(waitingList.length); // 5
```

</details>

---

# Part 4: A file is a room with a door

The regular lesson opens by telling you that big files are hard to navigate and hard to maintain. That is true, but it is also the kind of advice that is easy to nod along to and then ignore, because "hard to navigate" sounds like a comfort problem rather than a correctness problem.

So let us start somewhere more alarming. Big files do not just annoy you. They can be **silently wrong**.

## The bug

Here is `trip.js`, a single file for a hiking trip planner. Two features, written weeks apart, both living in the same file. Imagine two hundred lines of other code between them, so you never see them side by side.

```js
// trip.js

// ---- Packing list ----
const packing = [
  { item: 'Head torch', grams: 90 },
  { item: 'Rain jacket', grams: 340 }
];

function formatLine(entry) {
  return entry.item + ' (' + entry.grams + ' g)';
}

// ... two hundred lines of other code ...

// ---- Route legs ----
const legs = [
  { from: 'Car park', to: 'Bridge', km: 2.4 }
];

function formatLine(leg) {
  return leg.from + ' to ' + leg.to + ' (' + leg.km + ' km)';
}

packing.forEach((entry) => console.log(formatLine(entry)));
legs.forEach((leg) => console.log(formatLine(leg)));
```

Before you read on, decide what you expect this to print.

Here is what it actually prints:

```
undefined to undefined (undefined km)
undefined to undefined (undefined km)
Car park to Bridge (2.4 km)
```

There is no error. No warning. Nothing red in the console. The packing list simply produces nonsense, one line of it per item.

What happened is that both functions are called `formatLine`, and in one file there can only be one `formatLine`. The second declaration quietly replaced the first. By the time either line runs, the only `formatLine` that exists is the route one, so when it is handed `{ item: 'Head torch', grams: 90 }` it dutifully looks for `.from`, `.to` and `.km`, finds none of them, and reports `undefined` three times.

Now imagine that inside a real project. You did not touch the packing feature. You wrote a route feature at the other end of a long file. The packing feature broke anyway, and nothing told you.

**That** is the real argument for modules. Not tidiness. Isolation.

## The fix, and what it feels like

Split the file so that each feature lives in its own room:

```js
// packing.js
const packing = [
  { item: 'Head torch', grams: 90 },
  { item: 'Rain jacket', grams: 340 }
];

function formatLine(entry) {
  return entry.item + ' (' + entry.grams + ' g)';
}

export function printPacking() {
  packing.forEach((entry) => console.log(formatLine(entry)));
}
```

```js
// legs.js
const legs = [
  { from: 'Car park', to: 'Bridge', km: 2.4 }
];

function formatLine(leg) {
  return leg.from + ' to ' + leg.to + ' (' + leg.km + ' km)';
}

export function printLegs() {
  legs.forEach((leg) => console.log(formatLine(leg)));
}
```

```js
// main.js
import { printPacking } from './packing.js';
import { printLegs } from './legs.js';

printPacking();
printLegs();
```

Now it prints what you expected all along:

```
Head torch (90 g)
Rain jacket (340 g)
Car park to Bridge (2.4 km)
```

Both files still have a function called `formatLine`. They no longer collide, because they are in different rooms. Neither one can see the other, and neither one needs to.

Here is the detail that ought to sell you on modules for good. If you paste both `formatLine` functions into a **single module file**, you no longer get the silent nonsense. You get this, before a single line runs:

```
SyntaxError: Identifier 'formatLine' has already been declared
```

Modules do not only prevent the collision. When a collision does happen, they turn a silent wrong answer into a loud refusal to start. A loud error is a gift. A quiet wrong answer is a week of your life.

## Private by default, and `export` is the door

Look again at `packing.js`. Three things are declared in it: `packing`, `formatLine` and `printPacking`. Only one has `export` in front of it.

That is the whole model. **Everything in a module is private to that module unless you export it.** You are not hiding things; hidden is the starting position. `export` is you deciding to open a door.

Try to reach through the wall and you get nothing:

```js
// main.js
import { formatLine } from './packing.js';
// SyntaxError: The requested module './packing.js' does not provide an export named 'formatLine'
```

Good. `formatLine` is packing.js's own business. The rest of the application does not need to know it exists, which means the rest of the application cannot come to depend on it, which means you can rename it or delete it tomorrow without fear.

This is a design decision you get to make deliberately for every file you write: **what is the smallest set of things I can put in the doorway?** The fewer doors, the easier the file is to change later.

You can export as you declare, as above, or list everything at the bottom, which some people prefer because it puts the file's public face in one place:

```js
// packing.js, alternative ending
function printPacking() {
  packing.forEach((entry) => console.log(formatLine(entry)));
}

function totalGrams() {
  let total = 0;
  packing.forEach((entry) => {
    total = total + entry.grams;
  });
  return total;
}

export { printPacking, totalGrams };
```

Both styles do exactly the same thing. Pick one and be consistent within a project.

## `import`: walking through the door

Named exports come in with curly braces, and the names must match:

```js
import { printPacking, totalGrams } from './packing.js';
```

Those curly braces are not destructuring, even though they look exactly like it. It is separate syntax that happens to have borrowed the appearance. The practical consequence is that you cannot do clever pattern things in there; you can only list names, and optionally rename with `as`:

```js
import { printPacking as showKit } from './packing.js';

showKit();
```

Renaming on import is your escape hatch when two modules export the same name and you need both in one file.

Two details about the path that cause most beginner errors:

- **The `./` is required.** `import { x } from 'packing.js'` does not mean "the file next door", it means "a package called packing.js from somewhere else", and the browser will fail to find it.
- **The `.js` is required.** Node and various build tools let you drop it. Browsers do not. Since you are writing for the browser, always write the extension.

## Default exports

Sometimes a file is really about one single thing, and wrapping it in braces feels like ceremony. For those, there is `export default`, and each file may have at most one:

```js
// route.js
const route = {
  name: 'Bridge to cabin',
  km: 5.1,
  ascent: 320
};

export default route;
```

```js
// main.js
import route from './route.js';

console.log(route.name);
```

No braces on the way out, no braces on the way in. And because there is only one default per file, **the name is yours to choose**. This is legal and does the same thing:

```js
import whateverIWant from './route.js';
```

That flexibility is a mild downside as well as a feature: two files importing the same default under two different names makes a codebase harder to search. The habit most people settle on is to use a name that matches the file.

You can mix the two in one file, though a file that needs both is often a file doing two jobs:

```js
// route.js
const route = { name: 'Bridge to cabin', km: 5.1, ascent: 320 };

export default route;

export function isSteep(leg) {
  return leg.ascent > 250;
}
```

```js
// main.js
import route, { isSteep } from './route.js';

console.log(isSteep(route)); // true
```

The default comes first, then the braces.

**When to use which:** if your file provides a collection of related tools, use named exports. If it provides one obvious thing, a default is fine. If you cannot decide, use named exports. They are easier to search for and easier to add to later.

## Making it run in a browser

Write `import` in an ordinary script tag and the browser will refuse:

```
Uncaught SyntaxError: Cannot use import statement outside a module
```

That is not the browser being difficult. Modules and plain scripts genuinely behave differently, so the browser needs telling which set of rules to apply. You tell it with one attribute:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Trip planner</title>
    <script type="module" src="./main.js"></script>
  </head>
  <body>
    <h1>Trip planner</h1>
    <p>Open the console.</p>
  </body>
</html>
```

You only point at your **entry point**, `main.js`. You do not add script tags for `packing.js` and `legs.js`. The browser reads `main.js`, sees what it imports, goes and fetches those, sees what **they** import, and keeps going until it has the whole tree. Then it runs everything.

Three behaviours change once a script is a module, and all three are improvements:

- **Modules always run in strict mode.** A set of sloppy old behaviours is switched off for you, and several silent mistakes become errors instead.
- **Modules wait for the page.** A module script behaves as though it has `defer` on it, so the HTML is fully parsed before your code runs. That old advice about putting your script tag at the bottom of the `<body>` no longer applies. In the head is fine.
- **Top-level variables stay in the module.** A `const` at the top of a module belongs to that module, not to the global `window` object. That is the isolation from the beginning of this section, stated formally.

<details>

<summary>Rabbit hole: "it worked as a plain script and now it does not load at all"</summary>

You will hit this and it is worth ten seconds now rather than an hour later.

If you double-click your `index.html` and open it that way, the address bar says `file:///C:/Users/...`. Plain scripts work fine like that. Module scripts **do not**, and you get an error mentioning CORS or `file://` origins.

The reason is that modules are fetched over the network the way any other resource is, and the browser applies its security rules to that fetch. Files opened straight from your disk have no proper origin, so the rules block it.

The fix is to serve the folder over HTTP instead. In VS Code, install the Live Server extension, right-click `index.html` and choose "Open with Live Server". The address bar will say `http://127.0.0.1:5500/` or similar, and everything works.

From here on, get into the habit of always using Live Server. There is nothing to lose by it, and later in the course you will hit the same restriction again when you start fetching data.

</details>

<details>

<summary>Rabbit hole: a module runs once, and everyone shares the same copy</summary>

Suppose `counter.js` looks like this:

```js
// counter.js
let count = 0;

export function bump() {
  count = count + 1;
  return count;
}
```

Now two different files both import it:

```js
// pageA.js
import { bump } from './counter.js';
console.log(bump()); // 1
```

```js
// pageB.js
import { bump } from './counter.js';
console.log(bump()); // 2, not 1
```

`pageB` did not get a fresh counter starting at zero. It got the **same** counter.

A module's code runs exactly once, the first time anything imports it. Every later import is handed the result that already exists. There is one `count` in the whole application and both files are looking at it.

Most of the time this is exactly what you want. It is how a module can hold a shopping basket, a logged-in user or a cache that the whole application agrees on. But it does mean that a variable at the top of a module is shared state, and shared state can be changed by code you are not currently looking at. Worth knowing before it surprises you.

</details>

## Exercise 4

You are given one file that has the same disease as `trip.js`. Your job is to diagnose it and split it up.

Create a folder called `pool-log` and put this in `everything.js`:

```js
// everything.js

const sessions = [
  { lane: 1, activity: 'Lane swimming', minutes: 60 },
  { lane: 2, activity: 'Aqua aerobics', minutes: 45 },
  { lane: 3, activity: 'Lessons', minutes: 30 }
];

function describe(session) {
  return 'Lane ' + session.lane + ': ' + session.activity;
}

const instructors = [
  { name: 'Hedda', qualified: true },
  { name: 'Bjorn', qualified: false }
];

function describe(instructor) {
  return instructor.name + ' (qualified: ' + instructor.qualified + ')';
}

sessions.forEach((session) => console.log(describe(session)));
instructors.forEach((instructor) => console.log(describe(instructor)));
```

1. Run it as a plain script first, either with Node or in a normal `<script>` tag. Note down what it prints and write a one-line comment explaining why.
2. Split it into three files:
   - `sessions.js` holding the sessions array and its own private `describe`, exporting a single named function `printSessions()`.
   - `instructors.js` holding the instructors array and its own private `describe`, exporting a single named function `printInstructors()`.
   - `main.js` importing both and calling them.
3. Add a fourth file, `format.js`, exporting a named function `heading(text)` that returns the text in upper case with `'--- '` before it and `' ---'` after. Import it into `main.js` and print a heading before each section.
4. Create `index.html` that loads `./main.js` as a module, and open it with Live Server. Confirm the console shows all five lines with no errors.
5. In `sessions.js`, add a `totalMinutes()` function that adds up the minutes of every session, and export it alongside `printSessions()` using an export list at the bottom of the file. Log the total from `main.js`.

<details>

<summary>Solution to Exercise 4</summary>

**Step 1.** Run as a plain script, it prints:

```
undefined (qualified: undefined)
undefined (qualified: undefined)
undefined (qualified: undefined)
Hedda (qualified: true)
Bjorn (qualified: false)
```

Both functions are called `describe`, so the second declaration replaced the first before anything ran. Every call, including the three for sessions, went to the instructor version, which looked for `.name` and `.qualified` on session objects and found neither. Note that the lane numbers do not appear at all: the session version of `describe` was never called, so nothing ever read `.lane`.

**sessions.js**

```js
const sessions = [
  { lane: 1, activity: 'Lane swimming', minutes: 60 },
  { lane: 2, activity: 'Aqua aerobics', minutes: 45 },
  { lane: 3, activity: 'Lessons', minutes: 30 }
];

// Private to this module. instructors.js has one with the same name and
// they will never meet.
function describe(session) {
  return 'Lane ' + session.lane + ': ' + session.activity;
}

function printSessions() {
  sessions.forEach((session) => console.log(describe(session)));
}

function totalMinutes() {
  let total = 0;
  sessions.forEach((session) => {
    total = total + session.minutes;
  });
  return total;
}

// Step 5: an export list at the bottom puts the module's public face in
// one place.
export { printSessions, totalMinutes };
```

**instructors.js**

```js
const instructors = [
  { name: 'Hedda', qualified: true },
  { name: 'Bjorn', qualified: false }
];

function describe(instructor) {
  return instructor.name + ' (qualified: ' + instructor.qualified + ')';
}

export function printInstructors() {
  instructors.forEach((instructor) => console.log(describe(instructor)));
}
```

**format.js**

```js
export function heading(text) {
  return '--- ' + text.toUpperCase() + ' ---';
}
```

**main.js**

```js
import { printSessions, totalMinutes } from './sessions.js';
import { printInstructors } from './instructors.js';
import { heading } from './format.js';

console.log(heading('Sessions'));
printSessions();
console.log('Total pool time: ' + totalMinutes() + ' minutes');

console.log(heading('Instructors'));
printInstructors();
```

**index.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Pool log</title>
    <script type="module" src="./main.js"></script>
  </head>
  <body>
    <h1>Pool log</h1>
    <p>Open the console.</p>
  </body>
</html>
```

**Console output**

```
--- SESSIONS ---
Lane 1: Lane swimming
Lane 2: Aqua aerobics
Lane 3: Lessons
Total pool time: 135 minutes
--- INSTRUCTORS ---
Hedda (qualified: true)
Bjorn (qualified: false)
```

Notice that `main.js` only imports three names in total. It has no idea that `describe` exists, twice. That is the isolation working.

</details>

---

# Self study task: the seed library

A village runs a seed library. People borrow seeds in spring, grow them, and return seeds from the best plants in autumn. You are building a small console report for the volunteer who runs it.

This task uses all three ideas from the lesson: destructuring, the three dots, and modules.

## Setup

Make a folder called `seed-library` with four files: `index.html`, `seeds.js`, `report.js` and `main.js`.

Put this in `seeds.js` and nothing else for now:

```js
const seeds = [
  { name: 'Broad bean', family: 'legume', packets: 12, year: 2024, notes: 'Very reliable' },
  { name: 'Kale', family: 'brassica', packets: 3, year: 2023 },
  { name: 'Carrot', family: 'umbellifer', packets: 0, year: 2024 },
  { name: 'Runner bean', family: 'legume', packets: 7, year: 2022, notes: 'Slow to germinate' },
  { name: 'Swede', family: 'brassica', packets: 5, year: 2024 },
  { name: 'Parsnip', family: 'umbellifer', packets: 2, year: 2021 }
];
```

## Brief

**1. Make `seeds.js` a module.** Use a default export to share the `seeds` array.

**2. Build `report.js`.** It should import the seeds and provide these named exports. None of them should print anything; each should return a value, and `main.js` will do the printing.

- `describeSeed(seedObject)` returns a line like `"Broad bean (legume): 12 packets - Very reliable"`. It must destructure in its parameter list, and it must use a **default value** so that seeds without a `notes` property get `"no notes"` instead of `undefined`.
- `packetsByFamily()` returns an **object** where each key is a family name and each value is the total number of packets for that family. From the data above it should produce `{ legume: 19, brassica: 8, umbellifer: 2 }`.
- `outOfStock()` returns an array of the **names** of every seed with 0 packets.
- `withRestock(restockObject)` takes an object like `{ Carrot: 10, Kale: 6 }` and returns a **brand new array** of seed objects where the named seeds have their `packets` replaced by the new figure. It must not change the original `seeds` array in any way.

**3. Build `main.js`.** Import what you need and print a readable report: a line for every seed, the packets-per-family breakdown, the out-of-stock list, and then the restocked figures for Carrot and Kale. Finish by printing the original Carrot packet count to prove `withRestock` did not touch it.

**4. Build `index.html`** loading `./main.js` as a module, and run it with Live Server.

## Hints

- For `packetsByFamily()` you are **building** an object rather than reading one. Start with `const totals = {};` and add to it as you loop. Bracket notation is how you set a key whose name is in a variable: `totals[family] = ...`. You will need to check whether the key exists yet.
- For `withRestock()`, `map` is the right tool, because you want a new array of the same length. Inside the callback, decide whether this seed is being restocked, and either return a spread copy with `packets` overridden or return the seed unchanged.
- `Object.keys(restock).includes(name)` is one way to ask "is this seed in the restock list?".

<details>

<summary>Full solution</summary>

**seeds.js**

```js
const seeds = [
  { name: 'Broad bean', family: 'legume', packets: 12, year: 2024, notes: 'Very reliable' },
  { name: 'Kale', family: 'brassica', packets: 3, year: 2023 },
  { name: 'Carrot', family: 'umbellifer', packets: 0, year: 2024 },
  { name: 'Runner bean', family: 'legume', packets: 7, year: 2022, notes: 'Slow to germinate' },
  { name: 'Swede', family: 'brassica', packets: 5, year: 2024 },
  { name: 'Parsnip', family: 'umbellifer', packets: 2, year: 2021 }
];

// This file is about one thing, so a default export is a fair choice.
export default seeds;
```

**report.js**

```js
import seeds from './seeds.js';

// Destructuring in the parameter list, with a default for the property
// that is missing from most of the records.
function describeSeed({ name, family, packets, notes = 'no notes' }) {
  return name + ' (' + family + '): ' + packets + ' packets - ' + notes;
}

// Here we are building an object rather than reading one, so we start
// with an empty one and fill it in as we go.
function packetsByFamily() {
  const totals = {};

  seeds.forEach(({ family, packets }) => {
    if (totals[family] === undefined) {
      totals[family] = packets;
    } else {
      totals[family] = totals[family] + packets;
    }
  });

  return totals;
}

function outOfStock() {
  return seeds
    .filter(({ packets }) => packets === 0)
    .map(({ name }) => name);
}

// map gives us a new array of the same length. For each seed we either
// return a modified copy or the original object untouched.
function withRestock(restock) {
  const restockedNames = Object.keys(restock);

  return seeds.map((seed) => {
    if (restockedNames.includes(seed.name)) {
      // Copy everything, then override the one key. The original seed
      // object is not touched.
      return { ...seed, packets: restock[seed.name] };
    }

    return seed;
  });
}

export { describeSeed, packetsByFamily, outOfStock, withRestock };
```

**main.js**

```js
import seeds from './seeds.js';
import { describeSeed, packetsByFamily, outOfStock, withRestock } from './report.js';

console.log('--- ALL SEEDS ---');
seeds.forEach((seed) => console.log(describeSeed(seed)));

console.log('--- PACKETS BY FAMILY ---');
Object.entries(packetsByFamily()).forEach(([family, packets]) => {
  console.log(family + ': ' + packets);
});

console.log('--- OUT OF STOCK ---');
console.log(outOfStock().join(', '));

console.log('--- AFTER RESTOCK ---');
const restocked = withRestock({ Carrot: 10, Kale: 6 });
restocked.forEach((seed) => console.log(describeSeed(seed)));

console.log('--- ORIGINAL UNTOUCHED ---');
const originalCarrot = seeds.find(({ name }) => name === 'Carrot');
console.log('Carrot packets in the original array: ' + originalCarrot.packets);
```

**index.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Seed library</title>
    <script type="module" src="./main.js"></script>
  </head>
  <body>
    <h1>Seed library report</h1>
    <p>Open the console.</p>
  </body>
</html>
```

**Expected output**

```
--- ALL SEEDS ---
Broad bean (legume): 12 packets - Very reliable
Kale (brassica): 3 packets - no notes
Carrot (umbellifer): 0 packets - no notes
Runner bean (legume): 7 packets - Slow to germinate
Swede (brassica): 5 packets - no notes
Parsnip (umbellifer): 2 packets - no notes
--- PACKETS BY FAMILY ---
legume: 19
brassica: 8
umbellifer: 2
--- OUT OF STOCK ---
Carrot
--- AFTER RESTOCK ---
Broad bean (legume): 12 packets - Very reliable
Kale (brassica): 6 packets - no notes
Carrot (umbellifer): 10 packets - no notes
Runner bean (legume): 7 packets - Slow to germinate
Swede (brassica): 5 packets - no notes
Parsnip (umbellifer): 2 packets - no notes
--- ORIGINAL UNTOUCHED ---
Carrot packets in the original array: 0
```

**Two things to notice in this solution.**

First, `describeSeed` never mentions `year`, even though every seed has one. The destructuring pattern in the parameter list is a statement of what the function actually needs. Adding a new property to the seed data tomorrow will not disturb it.

Second, `withRestock` returns unmodified seed objects for the ones it is not changing, rather than copying every single one. Those are shared between the old array and the new array. That is fine here because nothing ever modifies a seed object in place, but it is exactly the shallow-copy situation from Part 3, and worth being aware of.

</details>

---

# Where this leaves you

Three sentences to take away.

**The shape goes on the left.** Any time you find yourself typing `something.thing` three times in a row, or reaching into an argument object at the top of a function, there is a pattern waiting to be written on the left of the `=`.

**Three dots, two directions.** Left collects, right pours. And the pouring version gives you "copy everything, override one thing", which is how you change data without breaking whatever else was holding on to it.

**A file is a room with a door.** Everything is private until you export it, so the real question for every module you write is not what to export but how little.

The next module moves on to the DOM, where your JavaScript starts changing what people actually see on the page. When you get there you will find that `Object.entries()` and destructuring turn up constantly, because data arriving from anywhere outside your own file tends to come as objects that you need to take apart before you can display them.
