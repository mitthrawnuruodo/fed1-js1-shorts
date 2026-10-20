# Claims, checked before running

**Estimated time:** about 2 hours for the core path, plus 30 to 45 minutes for the self study task.  
**Prerequisites:** Modules 1 to 6. No installation needed, everything happens in your browser.  

This is an optional extra lesson covering the same ground as lessons 7.1 to 7.4, by a different road.

The regular lessons start by teaching you how to write types. This one starts with JavaScript you could already write, hands it to TypeScript, and watches what TypeScript works out on its own. You will write surprisingly few types in this lesson, and that is the point. Two sentences carry it:

**1. A type is a claim about what a value is allowed to be, and TypeScript checks your claims before the code ever runs.**

**2. TypeScript works out most of the claims by itself, so you only write them down at the edges, where values arrive that it cannot see.**

The running example is the departure board for a small local ferry company. The timetable is made up, so do not plan a trip with it.

---

## Part 1 - Somewhere to try it

Open the TypeScript Playground in a new tab:

```text
https://www.typescriptlang.org/play
```

On the left is an editor. On the right is a panel with some tabs along the top. The two you need are **.JS** and **Logs**. Above the editor is a **Run** button.

Delete whatever example code is in the editor and paste this in. It is plain JavaScript, nothing new:

```ts
const departures = [
  { destination: 'Holmen', time: '08:15', minutesLate: 0 },
  { destination: 'Sandvik', time: '09:40', minutesLate: 5 },
  { destination: 'Rong', time: '11:05', minutesLate: 0 },
];

for (const departure of departures) {
  console.log(departure.time + ' to ' + departure.destination);
}
```

Press **Run**, then open the **Logs** tab. Three lines, exactly as if you had run it in the browser console.

That is the first thing worth knowing about TypeScript: **every JavaScript file is already a TypeScript file.** You are not starting a new language from nothing. You are adding one layer on top of what you have.

Now open the **.JS** tab. That is the JavaScript that TypeScript produced from your code, and right now it looks almost exactly the same as what you wrote. Hold on to that tab, it will become interesting in Part 4.

### Exercise 1 - Hover and predict

Before you hover over anything, write down what you think TypeScript has decided each of these is. Then paste the lines into the Playground (below the code you already have) and hover your mouse over each variable name to check.

```ts
let ferryCount = 3;
let companyName = 'Island Hopper';
let isRunning = true;
let firstDeparture = departures[0];
const homeQuay = 'Strandkaien';
```

<details>
<summary>Solution</summary>

`ferryCount` is a `number`, `companyName` is a `string` and `isRunning` is a `boolean`. You wrote no types at all, and TypeScript worked them out from the values. This is called **inference**.

`firstDeparture` is the one to look at properly. The hover shows something like:

```text
let firstDeparture: {
    destination: string;
    time: string;
    minutesLate: number;
}
```

TypeScript looked at your array of objects and worked out the shape of every object in it. You never told it what a departure looks like. It figured that out too.

`homeQuay` is the odd one. The hover does not say `string`, it says `"Strandkaien"`. That is not a mistake. See the rabbit hole below if you want to know why, but you can safely carry on without it.

</details>

<details>
<summary>Rabbit hole: why does the const say "Strandkaien" instead of string?</summary>

A `let` can be given a new value later, so TypeScript has to allow any string in it. A `const` can never change, so TypeScript knows more than "some string": it knows it is exactly the text `'Strandkaien'` and nothing else, ever.

So TypeScript gives it the narrowest type it can, a type with only one possible value in it. Types like that are called literal types. They are very useful later, for things like "the status can only be `'success'` or `'error'`". For now, treat `"Strandkaien"` as a very precise way of saying `string`.

</details>

---

## Part 2 - The red line is a comment, not a crash

Add one more departure to the end of the array, with a small mistake in it:

```ts
const departures = [
  { destination: 'Holmen', time: '08:15', minutesLate: 0 },
  { destination: 'Sandvik', time: '09:40', minutesLate: 5 },
  { destination: 'Rong', time: '11:05', minutesLate: 0 },
];

departures.push({ destination: 'Fedje', time: '12:30', minutesLate: '10' });
```

A wavy red line appears under part of the new line. Hover over it. The message is long, but the important bit is near the end: `Type 'string' is not assignable to type 'number'`.

TypeScript worked out in Part 1 that `minutesLate` holds numbers. You claimed, by writing the array that way, that this is what a departure looks like. Now you are trying to put a text value in there, and TypeScript is pointing out that the claim and the value disagree.

Here is the part that surprises almost everyone. Press **Run**.

### Exercise 2 - Predict, then run

Before you press **Run**, decide: will the Playground refuse to run the code, run it and crash, or run it as normal? Then add this line at the end and run it:

```ts
console.log(departures[3].minutesLate + 15);
```

<details>
<summary>Solution</summary>

It runs as normal, and logs `1015`.

The red line did not stop anything. TypeScript is a reader, not a guard at the door. It reads your code, tells you what looks wrong, and then the code runs anyway, exactly the way JavaScript would run it.

And JavaScript did exactly what you learned in Module 1: `'10' + 15` joins text instead of adding numbers. That is the bug TypeScript was warning you about. A ferry that is fifteen minutes later than ten minutes late is twenty-five minutes late, not a thousand and fifteen.

So a red line means "this will probably go wrong when it runs". It does not mean "this cannot run". Your job is to fix every red line before you trust the code, because nothing else will make you.

</details>

Fix the mistake by changing `'10'` to `10`, and the red line goes away.

---

## Part 3 - The edges: where TypeScript cannot see

So far TypeScript has worked everything out by itself. Here is where that stops. Write this function below your array:

```ts
/**
 * Builds one line of text for the departure board.
 * @param departure The departure to describe.
 * @returns A line like "09:40 Sandvik".
 */
function describe(departure) {
  return departure.time + ' ' + departure.destination;
}
```

`departure` gets a red line. Hover over it: `Parameter 'departure' implicitly has an 'any' type.`

Think about why. Everything in Part 1 had a value right there in the code for TypeScript to look at. A parameter does not. It is an empty slot that gets filled in later, by whoever calls the function, and TypeScript cannot know who that will be. So it asks you.

This is what the second sentence means. Inside your code, TypeScript can follow values around by itself. At the edges, where a value comes in from somewhere it cannot see, you have to make the claim yourself.

The type `any` it mentions means "anything at all, I will not check it". TypeScript is warning you that it is about to stop checking, which is the opposite of what you want it for.

### Giving the shape a name

You could write the whole object shape out inside the brackets, but that gets long quickly, and you are going to need it in more than one function. Give it a name instead, with an **interface**:

```ts
interface Departure {
  destination: string;
  time: string;
  minutesLate: number;
}
```

An interface is nothing but a named description of a shape. Read it as "a `Departure` is an object with these three properties, of these three types". It does not create any objects. It is a label you can stick on things.

Now you can stick it on the parameter, and on the array too:

```ts
interface Departure {
  destination: string;
  time: string;
  minutesLate: number;
}

const departures: Departure[] = [
  { destination: 'Holmen', time: '08:15', minutesLate: 0 },
  { destination: 'Sandvik', time: '09:40', minutesLate: 5 },
  { destination: 'Rong', time: '11:05', minutesLate: 0 },
];

/**
 * Builds one line of text for the departure board.
 * @param departure The departure to describe.
 * @returns A line like "09:40 Sandvik".
 */
function describe(departure: Departure) {
  return departure.time + ' ' + departure.destination;
}

for (const departure of departures) {
  console.log(describe(departure));
}
```

`Departure[]` means "an array of `Departure` objects", the same way `string[]` would mean an array of strings.

Type `departure.` inside the function and look at the list of suggestions that pops up. The Playground knows the three properties now, so it can offer them, the same way VS Code will in a real project. That alone saves a lot of typos.

### The return value, and the promise you can add

Hover over the name `describe`. TypeScript shows `function describe(departure: Departure): string`. That `: string` at the end is the type of the value the function returns, and you did not write it. It was inferred from the `return` line.

You can write it yourself, and sometimes you should:

```ts
/**
 * Builds one line of text for the departure board.
 * @param departure The departure to describe.
 * @returns A line like "09:40 Sandvik" or "09:40 Sandvik (5 min late)".
 */
function describe(departure: Departure): string {
  if (departure.minutesLate > 0) {
    return departure.time + ' ' + departure.destination + ' (' + departure.minutesLate + ' min late)';
  }
  return departure.time + ' ' + departure.destination;
}
```

Writing `: string` turns the return type from a description into a promise. If you later edit the function and some path through it forgets to return anything, TypeScript will complain at the function, where the mistake is, rather than somewhere far away where a mysterious `undefined` turns up on the screen. Exercise 3 shows you what that looks like.

And when a function does a job but hands nothing back, like one that only logs, its return type is `void`, which means "nothing worth using":

```ts
/**
 * Prints the whole board to the console.
 * @param list The departures to print.
 */
function printBoard(list: Departure[]): void {
  for (const departure of list) {
    console.log(describe(departure));
  }
}

printBoard(departures);
```

### Exercise 3 - The missing return

This version of `describe` has a mistake. Read it and decide where TypeScript will put the red line, and what the board would show if you ran it anyway. Then paste it in place of your `describe` to check.

```ts
/**
 * Builds one line of text for the departure board.
 * @param departure The departure to describe.
 * @returns A line of text for the board.
 */
function describe(departure: Departure): string {
  if (departure.minutesLate > 0) {
    return departure.time + ' ' + departure.destination + ' (late)';
  } else if (departure.minutesLate === 0) {
    return departure.time + ' ' + departure.destination;
  }
}
```

<details>
<summary>Solution</summary>

The red line goes under the `: string` after the brackets, and the message says the function `lacks ending return statement and return type does not include 'undefined'`.

Compiler messages are written for experienced programmers, so here it is in plain words: "You promised this function always gives back a string. But there is a way through it that reaches the end without hitting a `return`, and then it would give back `undefined` instead."

To a person, the two branches look like they cover everything. To TypeScript, `minutesLate` is a `number`, and a number can also be negative. A ferry that leaves early (`-2`) matches neither branch, falls out of the bottom of the function, and the function returns `undefined` without saying so.

If you run it anyway with a departure whose `minutesLate` is `-2`, the board shows the word `undefined`, and nothing anywhere tells you why. With `: string` written on the function, you are told at the exact line, before anything runs.

The simplest fix is to make the last case an `else` with no condition, so there is no way out of the function without a `return`:

```ts
/**
 * Builds one line of text for the departure board.
 * @param departure The departure to describe.
 * @returns A line of text for the board.
 */
function describe(departure: Departure): string {
  if (departure.minutesLate > 0) {
    return departure.time + ' ' + departure.destination + ' (late)';
  } else {
    return departure.time + ' ' + departure.destination;
  }
}
```

</details>

<details>
<summary>Rabbit hole: JSDoc already had types, so why both?</summary>

In Module 6 you may have seen JSDoc comments with types in them, like `@param {number} minutes`. Those are hints for VS Code. Nothing checks that they are true, and they can quietly drift away from the code.

TypeScript types are part of the code itself, and they are checked every time. So in a TypeScript file, the types move out of the comment and into the code, and the comment keeps only the human part: what the function is for, and what each parameter means. That is why the JSDoc in this lesson has descriptions but no types in curly braces.

</details>

---

## Part 4 - The types disappear

Open the **.JS** tab again, with your interface, array and functions still in the editor.

### Exercise 4 - Where did it go?

Before you look closely, predict: what has happened to the interface, the `: Departure` and the `: string` in the JavaScript output?

<details>
<summary>Solution</summary>

They are all gone. The interface has vanished completely, and every `: Departure`, `: string`, `: void` and `Departure[]` has been removed. What is left is the plain JavaScript you would have written in Module 6.

That is how all of TypeScript works. The browser cannot run TypeScript. Before your code runs, the types are checked, and then they are stripped out, and the browser only ever sees ordinary JavaScript.

This has one big consequence, and the rest of this lesson is about it: **TypeScript can only check what it can see before the code runs.** Once the code is running, the types do not exist any more. Nothing checks anything at that point.

</details>

---

## Part 5 - Optional, and what optional really means

Some departures get cancelled. Most do not, and you do not want to have to write `cancelled: false` on every single one. A question mark makes a property optional:

```ts
interface Departure {
  destination: string;
  time: string;
  minutesLate: number;
  cancelled?: boolean;
}

const departures: Departure[] = [
  { destination: 'Holmen', time: '08:15', minutesLate: 0 },
  { destination: 'Sandvik', time: '09:40', minutesLate: 5, cancelled: true },
  { destination: 'Rong', time: '11:05', minutesLate: 0 },
  { destination: 'Fedje', time: '12:30', minutesLate: 0, cancelled: false },
];
```

Now hover over `cancelled` in any line where you use it, for example in `departures[0].cancelled`. The type is not `boolean`. It is `boolean | undefined`.

The `|` means "or", so read `boolean | undefined` out loud as "a boolean, or undefined". It is how TypeScript says that a value can be one of several types. You do not need to write these yourself in this lesson, but you will see them when you hover, so it helps to recognise them.

So `?` does not mean "you can ignore this property". It means **"this property might not be there, and if it is not, reading it gives you `undefined`"**. A departure without the flag does not have `cancelled: false`. It has no `cancelled` at all.

### Exercise 5 - Counting the ferries that run

Here are two functions that are both meant to count the departures that are not cancelled. Predict what each one returns for the four departures above, then run them.

```ts
/**
 * Counts the departures that are not cancelled.
 * @param list The departures to count.
 * @returns How many are running.
 */
function countRunningA(list: Departure[]): number {
  const running = list.filter((departure) => departure.cancelled === false);
  return running.length;
}

/**
 * Counts the departures that are not cancelled.
 * @param list The departures to count.
 * @returns How many are running.
 */
function countRunningB(list: Departure[]): number {
  const running = list.filter((departure) => !departure.cancelled);
  return running.length;
}

console.log(countRunningA(departures));
console.log(countRunningB(departures));
```

<details>
<summary>Solution</summary>

`countRunningA` returns `1`. `countRunningB` returns `3`, which is the right answer.

Only Fedje actually has `cancelled: false` written on it. Holmen and Rong have no `cancelled` property, so for them `departure.cancelled` is `undefined`, and `undefined === false` is `false`. Version A quietly throws away two ferries that are running perfectly well.

Version B uses `!`, which you met with the logical operators in Module 2. `!undefined` is `true`, just like `!false`, so missing and false are treated the same way, which is what you want here.

Notice that TypeScript did not put a red line under either version. Both are perfectly valid code. It did tell you, the moment you hovered, that the type was `boolean | undefined`. Reading that hover is the habit that would have caught it.

</details>

---

## Part 6 - The edge you cannot see across: data from a server

There is one more edge, and it is the biggest one. When your code fetches data from an API, the data comes from another computer, at the moment the code runs. TypeScript checks your code before it runs. It has never seen that data and never will.

Try this in a fresh Playground tab. It uses the Dog CEO API, which is free, needs no key, and answers with a random dog photo:

```ts
/**
 * Fetches one random dog photo address and logs it.
 */
async function showRandomDog(): Promise<void> {
  const response = await fetch('https://dog.ceo/api/breeds/image/random');
  const data = await response.json();
  console.log(data.message);
}

showRandomDog();
```

Press **Run** and look in **Logs**. You get an address ending in `.jpg`. Paste it into a new browser tab if you want to see the dog.

The return type `Promise<void>` is how you write "this is an `async` function that does not return anything useful". An `async` function always hands back a promise, as you saw in Module 4, so its return type always has `Promise<...>` around it.

The answer from the server looks like this:

```json
{
  "message": "https://images.dog.ceo/breeds/terrier-norfolk/n02094114_1505.jpg",
  "status": "success"
}
```

### Exercise 6 - The typo nobody catches

Change `data.message` to `data.mesage`, with one `s`. Before you run it: will there be a red line? What will the log show?

<details>
<summary>Solution</summary>

No red line at all. When it runs, the log shows `undefined`.

Hover over `data` and you will see why: its type is `any`. The `json()` method cannot know what the server is going to send, so it gives you `any`, which as you saw in Part 3 means "I will not check this". TypeScript has switched itself off for everything you do with `data`, typos included.

</details>

So, at this edge as well, you make the claim yourself. Describe the answer with an interface, and put it on the variable:

```ts
interface DogPhotoAnswer {
  message: string;
  status: string;
}

/**
 * Fetches one random dog photo address and logs it.
 */
async function showRandomDog(): Promise<void> {
  const response = await fetch('https://dog.ceo/api/breeds/image/random');
  const data: DogPhotoAnswer = await response.json();
  console.log(data.message);
}

showRandomDog();
```

Now `data.mesage` gets a red line, and TypeScript even asks if you meant `message`.

But be honest about what just happened. You did not check anything about the server's answer. You **told** TypeScript what the answer looks like, and TypeScript believed you, because it has no way to look. If the API changed tomorrow, your claim would be wrong, and no red line would ever tell you. Remember Part 4: once the code runs, the types are gone.

That is the whole of the first sentence, in the most important place: a type is a claim. Most of your claims, TypeScript can check. This one, it can only trust.

TypeScript has a more careful option for values like this, called `unknown`. A value of type `unknown` is one TypeScript refuses to let you use at all until you have checked what it is, with code that runs. That is what lesson 7.5 is about, and it is where the third TypeScript extra lesson picks up.

<details>
<summary>Rabbit hole: why does fetch not just know?</summary>

It cannot. `fetch` works with any address on the internet, and every API answers with a different shape. Even the same API can answer differently on different days: for example, the Dog CEO API answers with an error message instead of a photo address if you ask for a breed it does not have.

Some APIs publish a description of their answers that tools can turn into TypeScript interfaces automatically. That saves typing, but it is still a claim about what the server promises to send, not a check of what it actually sent.

</details>

---

## Exercises

These all use the `Departure` interface from Part 5, with the optional `cancelled`. Do them in the Playground.

### Exercise 7 - Three red lines

This code has three problems that TypeScript will catch. Find each one by reading it first, then paste it in and see whether you were right. Fix all three.

```ts
interface Departure {
  destination: string;
  time: string;
  minutesLate: number;
  cancelled?: boolean;
}

const evening: Departure[] = [
  { destination: 'Holmen', time: '18:15', minutesLate: 0 },
  { destination: 'Sandvik', time: 19.40, minutesLate: 0 },
];

/**
 * Builds a short label for a departure.
 * @param departure The departure to label.
 * @returns The destination in capital letters.
 */
function shortLabel(departure: Departure): string {
  return departure.destinaton.toUpperCase();
}

/**
 * Works out the new departure time in minutes after midnight.
 * @param minutesAfterMidnight The planned time.
 * @param delay How late the ferry is.
 * @returns The expected time.
 */
function expectedTime(minutesAfterMidnight: number, delay) {
  return minutesAfterMidnight + delay;
}

console.log(shortLabel(evening[0]));
console.log(expectedTime(1095, evening[0].minutesLate));
```

<details>
<summary>Solution</summary>

**One:** `time: 19.40` is a number, and the interface says `time` is a `string`. Without TypeScript this would show as `19.4` on the board, with the zero gone. Fix it to `'19:40'`.

**Two:** `departure.destinaton` is a typo. TypeScript says the property does not exist and suggests `destination`. In plain JavaScript this would be `undefined`, and calling `.toUpperCase()` on `undefined` would crash when it runs.

**Three:** `delay` has no type, so it is implicitly `any`. Give it one:

```ts
/**
 * Works out the new departure time in minutes after midnight.
 * @param minutesAfterMidnight The planned time.
 * @param delay How late the ferry is.
 * @returns The expected time.
 */
function expectedTime(minutesAfterMidnight: number, delay: number): number {
  return minutesAfterMidnight + delay;
}
```

Adding `: number` as the return type is optional here, since TypeScript works it out, but it does no harm.

</details>

### Exercise 8 - Write it typed from the start

Write a function `lateDepartures` that takes an array of departures and returns a new array with only the ones that are more than 0 minutes late and not cancelled. Write the parameter type and the return type yourself. Test it on this list, where the right answer is Sandvik only:

```ts
const morning: Departure[] = [
  { destination: 'Holmen', time: '07:10', minutesLate: 0 },
  { destination: 'Sandvik', time: '07:45', minutesLate: 12 },
  { destination: 'Rong', time: '08:20', minutesLate: 30, cancelled: true },
  { destination: 'Fedje', time: '08:55', minutesLate: 0, cancelled: false },
];
```

<details>
<summary>Solution</summary>

```ts
/**
 * Finds the departures that are running late.
 * @param list The departures to look through.
 * @returns A new array with the late departures that are still running.
 */
function lateDepartures(list: Departure[]): Departure[] {
  return list.filter((departure) => departure.minutesLate > 0 && !departure.cancelled);
}

const late = lateDepartures(morning);
console.log(late.length);
console.log(late[0].destination);
```

This logs `1` and then `Sandvik`. Rong is late too, but it is cancelled, so it is not late, it is just not coming.

Notice that you did not have to write a type for `departure` inside the arrow function. TypeScript already knows `list` is a `Departure[]`, so it knows every item that `filter` hands to your callback is a `Departure`. That is inference doing the work again, inside your own code.

</details>

### Exercise 9 - A number or nothing

Write a function `totalDelay` that adds up `minutesLate` for every departure that is not cancelled, and returns the total. Use a `for...of` loop. What should the return type be? For the `morning` list above, the answer is `12`.

Then write a function `logTotalDelay` that takes the list, calls `totalDelay`, and logs `Total delay: 12 minutes`. What should its return type be?

<details>
<summary>Solution</summary>

```ts
/**
 * Adds up the delays of every departure that is still running.
 * @param list The departures to add up.
 * @returns The total number of minutes late.
 */
function totalDelay(list: Departure[]): number {
  let total = 0;
  for (const departure of list) {
    if (!departure.cancelled) {
      total = total + departure.minutesLate;
    }
  }
  return total;
}

/**
 * Logs the total delay in a sentence.
 * @param list The departures to add up.
 */
function logTotalDelay(list: Departure[]): void {
  console.log('Total delay: ' + totalDelay(list) + ' minutes');
}

logTotalDelay(morning);
```

`totalDelay` returns a `number`. `logTotalDelay` returns nothing useful, so `void`.

Notice that `total` needed no type. It starts as `0`, so TypeScript knows it is a `number`. If you later tried `total = 'none'`, you would get a red line.

</details>

---

## Self study task: let the checker find the bugs

Below is a small departure board script written in plain JavaScript. It runs without crashing, but some of what it prints is wrong. Your job is not to hunt for the bugs by reading the code. Your job is to **add types, one step at a time, and let TypeScript point at the bugs for you**.

Each level adds one kind of type, and each one makes a new bug light up. Take them in order, and fix what you find before moving on.

Paste this into a fresh Playground tab and press **Run** first, so you can see the wrong output for yourself:

```ts
const board = [
  { destination: 'Holmen', time: '14:10', minutesLate: 0 },
  { destination: 'Sandvik', time: '14:45', minutesLate: '10' },
  { destination: 'Rong', time: '15:20', minutesLate: 0, cancelled: true },
  { destination: 'Fedje', time: '16:00', minutesLate: 25 },
];

function formatDelay(destination, minutes) {
  if (minutes === 0) {
    return destination + ': on time';
  } else if (minutes > 0) {
    return destination + ': ' + minutes + ' min late';
  }
}

function newTime(departure) {
  const minutesLate = departure.minutesLate + 5;
  return departure.time + ' (+' + minutesLate + ')';
}

for (const departure of board) {
  if (departure.cancelled) {
    console.log(departure.destinaton + ': cancelled');
  } else {
    console.log(formatDelay(departure.minutesLate, departure.destination));
    console.log(newTime(departure));
  }
}
```

The output has `undefined` in several places and a delay of `105` minutes that should have been `15`.

There are already a few red lines. Three of them are under the parameters `destination`, `minutes` and `departure`. Those are TypeScript asking for types, exactly like `describe` did in Part 3. Leave them for now, you will deal with them in Level 2.

### Level 1: the one it found by itself

There is one more red line, which is not about a missing type. Find it, read the hover message, and fix it. Then run again and see which line of output is now right.

<details>
<summary>Solution</summary>

`departure.destinaton` is a typo, and TypeScript says so: `Property 'destinaton' does not exist`, and it suggests `destination`. You never told it what a departure looks like. It worked that out from the array, just like in Part 1.

Fix the spelling, and the line `undefined: cancelled` becomes `Rong: cancelled`.

</details>

### Level 2: describe the data

Write a `Departure` interface for the board. Remember that only one of the departures has `cancelled`, so it needs a `?`. Then put it on the array, so it reads `const board: Departure[] = [`.

A new red line appears. Find it, work out what it would have done to the output, and fix it.

Then give the three parameters their types: `destination` and `minutes` in `formatDelay`, and `departure` in `newTime`. Another red line appears, somewhere else. Find it and fix it too.

<details>
<summary>Solution</summary>

With the interface on the array, Sandvik's `minutesLate: '10'` lights up, because the interface says `minutesLate` is a `number`. That was the `105`: `'10' + 5` joins text instead of adding. Change it to `10`.

```ts
interface Departure {
  destination: string;
  time: string;
  minutesLate: number;
  cancelled?: boolean;
}
```

With the parameter types in place (`destination: string, minutes: number` and `departure: Departure`), the call to `formatDelay` in the loop lights up. The two arguments are the wrong way round: a number is going in where the text should be, and the other way round. Swap them:

```ts
    console.log(formatDelay(departure.destination, departure.minutesLate));
```

Run again. The `undefined` lines are gone, because `formatDelay` is finally being given what it expects.

</details>

### Level 3: make the promises

Add `: string` as the return type on both functions. One of them gets a red line. Which one, and why?

Then add this departure to the end of the board, and fix the function so it gets a sensible message:

```ts
  { destination: 'Hjelme', time: '16:30', minutesLate: -3 },
```

<details>
<summary>Solution</summary>

`formatDelay` gets the red line. It handles zero and more than zero, but a ferry that leaves early, with a negative number, falls out of the bottom of the function and returns `undefined`. That is the same problem as Exercise 3. Nothing on the board was early, so the bug was hiding. The return type found it before any early ferry ever turned up.

Add an `else` for early ferries. `Math.abs`, from Module 2, turns `-3` into `3`:

```ts
/**
 * Describes how late, early or on time a ferry is.
 * @param destination Where the ferry is going.
 * @param minutes How late it is. Negative means early.
 * @returns A line for the board.
 */
function formatDelay(destination: string, minutes: number): string {
  if (minutes === 0) {
    return destination + ': on time';
  } else if (minutes > 0) {
    return destination + ': ' + minutes + ' min late';
  } else {
    return destination + ': ' + Math.abs(minutes) + ' min early';
  }
}
```

The point of the task is the order you worked in. You did not find the bugs by reading. You described the data and the functions honestly, and the bugs found themselves. That is what TypeScript is for.

</details>

<details>
<summary>The whole thing, finished</summary>

```ts
interface Departure {
  destination: string;
  time: string;
  minutesLate: number;
  cancelled?: boolean;
}

const board: Departure[] = [
  { destination: 'Holmen', time: '14:10', minutesLate: 0 },
  { destination: 'Sandvik', time: '14:45', minutesLate: 10 },
  { destination: 'Rong', time: '15:20', minutesLate: 0, cancelled: true },
  { destination: 'Fedje', time: '16:00', minutesLate: 25 },
  { destination: 'Hjelme', time: '16:30', minutesLate: -3 },
];

/**
 * Describes how late, early or on time a ferry is.
 * @param destination Where the ferry is going.
 * @param minutes How late it is. Negative means early.
 * @returns A line for the board.
 */
function formatDelay(destination: string, minutes: number): string {
  if (minutes === 0) {
    return destination + ': on time';
  } else if (minutes > 0) {
    return destination + ': ' + minutes + ' min late';
  } else {
    return destination + ': ' + Math.abs(minutes) + ' min early';
  }
}

/**
 * Shows the planned time and the delay plus five minutes for boarding.
 * @param departure The departure to show.
 * @returns A line like "14:45 (+15)".
 */
function newTime(departure: Departure): string {
  const minutesLate = departure.minutesLate + 5;
  return departure.time + ' (+' + minutesLate + ')';
}

for (const departure of board) {
  if (departure.cancelled) {
    console.log(departure.destination + ': cancelled');
  } else {
    console.log(formatDelay(departure.destination, departure.minutesLate));
    console.log(newTime(departure));
  }
}
```

</details>

---

## Where this leaves you

A type is a claim about what a value is allowed to be. TypeScript reads your code before it runs and checks every claim it can. A red line is a warning, not a stop sign, and the code will run anyway, wrong answers and all, so fixing the red lines is your job.

TypeScript works out most types by itself from the values it can see. Hover to find out what it decided. You write types at the edges, where values arrive that it cannot see: function parameters, and data from outside your program. An interface gives a shape a name so you can use it in more than one place. A return type turns a description into a promise, and `void` means "nothing worth using". A `?` on a property means it might be `undefined`, not that it is `false`.

And all of it disappears before the code runs. That makes data from a server the one edge where TypeScript can only take your word for it. The next TypeScript extra lesson sets up a real project on your own computer, and the one after that uses it to check data like this for real.
