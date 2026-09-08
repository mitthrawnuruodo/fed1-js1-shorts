# Answer a question, or do a thing

This is an optional extra lesson covering the same ground as lessons 2.3 and 2.4, by a different road.

It carries on with the board game shelf at the village hall from the previous extra lesson, which by now has six games on it.

```js
const shelf = [
  { title: 'Fjord Traders', minPlayers: 3, minutes: 90, isComplete: true },
  { title: 'Hare and Hound', minPlayers: 2, minutes: 20, isComplete: true },
  { title: 'The Lighthouse Keeper', minPlayers: 2, minutes: 45, isComplete: false },
  { title: 'Snow Line', minPlayers: 4, minutes: 60, isComplete: true },
  { title: 'Cabin Fever', minPlayers: 3, minutes: 30, isComplete: false },
  { title: 'Rope Bridge', minPlayers: 2, minutes: 40, isComplete: true },
];
```

Two sentences carry the lesson:

**1. A function either answers a question or does a thing. Try not to write one that does both.**

**2. A parameter is a promise about what you will be given. A return is a promise about what you will hand back.**

---

## Part 1: A function with two problems

Start with a job. Somebody wants to know how many minutes it would take to play everything on the shelf.

You already know how to work that out with a loop. Wrapping it in a function looks like this:

```js
function showTotalMinutes() {
  let total = 0;

  for (let i = 0; i < shelf.length; i++) {
    total = total + shelf[i].minutes;
  }

  console.log('Total playing time: ' + total + ' minutes.');
}

showTotalMinutes();
```

```
Total playing time: 285 minutes.
```

That is a function. It works. It is also the version almost every beginner writes first, and it has two problems that will bite within a week.

**Problem one: it prints the answer instead of handing it back.**

Suppose you now want to know whether the shelf holds more than four hours of games. You have the total inside the function, but it went to the screen and vanished. You cannot compare it to anything, store it, or use it in a calculation. The only thing this function can do is put words in the console.

**Problem two: it reaches out for the shelf instead of being given one.**

The word `shelf` appears inside the function, but it was never mentioned in the brackets. The function is reaching outside itself to grab a variable that happens to exist nearby. That works until somebody renames `shelf`, or until there are two shelves, at which point the function is stuck describing whichever one it was written next to.

Fixing those two problems is most of what lesson 2.3 is about, so let us fix them in that order.

---

## Part 2: Answering a question

`return` hands a value back to whoever called the function.

```js
function totalMinutes() {
  let total = 0;

  for (let i = 0; i < shelf.length; i++) {
    total = total + shelf[i].minutes;
  }

  return total;
}
```

Two changes. The `console.log` became a `return`, and the name lost its `show`, because the function no longer shows anything. It works something out and gives it to you.

Now the answer is a value like any other value, and everything you can do with a value you can do with this:

```js
const minutes = totalMinutes();

console.log(minutes);
console.log('That is ' + (minutes / 60) + ' hours of games.');

if (totalMinutes() > 240) {
  console.log('More than four hours on that shelf.');
}
```

```
285
That is 4.75 hours of games.
More than four hours on that shelf.
```

None of that was possible with the printing version. This is what "answers a question" buys you.

### return also stops the function

`return` does two jobs at once. It hands a value back, and it ends the function immediately. Anything written after it never runs.

```js
function describeShelf() {
  return 'A shelf of board games.';

  console.log('This line never runs.');
}

console.log(describeShelf());
```

```
A shelf of board games.
```

That second job turns out to be useful. Here is a function that reads a game's playing time, where the game might not exist:

```js
function minutesFor(game) {
  if (game === undefined) {
    return 0;
  }

  return game.minutes;
}

console.log(minutesFor(shelf[1]));
console.log(minutesFor(shelf[99]));
```

```
20
0
```

That first `if` is called a guard clause. It deals with the awkward case straight away and leaves the function, so the rest of the function can get on with the normal case without wrapping everything in an `else`. You will see this shape constantly, and it is worth recognising: **check for trouble first, leave early, then do the real work unindented.**

### A function with no return

If a function never reaches a `return`, it hands back `undefined`.

```js
function showTotalMinutes() {
  console.log('Total playing time: 285 minutes.');
}

const result = showTotalMinutes();

console.log(result);
```

```
Total playing time: 285 minutes.
undefined
```

This is the same grey `undefined` from the first extra lesson, and it is the same reason: the function printed something but produced nothing. If you ever store the result of a function and find `undefined` in the variable, check whether that function actually returns.

---

## Part 3: Doing a thing

Not every function should return something. Some functions exist precisely to have an effect on the world, and printing a report is a fair example.

```js
function printShelf(games) {
  for (let i = 0; i < games.length; i++) {
    const game = games[i];

    console.log((i + 1) + '. ' + game.title + ' (' + game.minutes + ' minutes)');
  }
}

printShelf(shelf);
```

```
1. Fjord Traders (90 minutes)
2. Hare and Hound (20 minutes)
3. The Lighthouse Keeper (45 minutes)
4. Snow Line (60 minutes)
5. Cabin Fever (30 minutes)
6. Rope Bridge (40 minutes)
```

That function returns nothing, and that is correct. Its whole purpose is the printing.

So the rule is not "always return". It is this:

**Decide which kind of function you are writing, and then be that kind.**

A function that works something out should hand it back and print nothing. A function that displays something should take what it needs and return nothing. When one function does both, you cannot reuse the calculation without also getting the output, and you cannot change the wording without touching the calculation.

The names should tell you which kind you are looking at. `totalMinutes` sounds like it hands you a number. `printShelf` sounds like it puts something on the screen. If you cannot name a function without using "and", that is usually a sign it is doing two jobs.

<details>

<summary>Rabbit hole: what about a function that changes something?</summary>

There is a third case worth knowing about, even though the two-way split covers most of what you will write for now.

Some functions do not print and do not return, but change something they were given.

```js
function addGame(games, newGame) {
  games.push(newGame);
}

addGame(shelf, { title: 'Ice Fishing', minPlayers: 2, minutes: 25, isComplete: true });

console.log(shelf.length);
```

```
7
```

The function returned nothing, but the shelf is longer than it was. This works because of the thing from the previous extra lesson's rabbit hole on object identity: `games` inside the function and `shelf` outside are two names tied to the same array, so pushing through one name changes what both names point at.

This is powerful and it is also the source of a great many confusing bugs, because from the outside there is no clue that anything happened. For now: if a function changes something it was given, say so in its name. `addGame` is honest. `checkGame` would not be.

</details>

---

## Part 4: Promises at the edges

Now the second problem from Part 1. The function should be **given** a shelf rather than reaching out for one.

```js
function totalMinutes(games) {
  let total = 0;

  for (let i = 0; i < games.length; i++) {
    total = total + games[i].minutes;
  }

  return total;
}

console.log(totalMinutes(shelf));
```

```
285
```

`games` is a **parameter**: a name listed in the function's definition, standing in for something that will be supplied later. `shelf` in the last line is an **argument**: the actual thing supplied.

The distinction sounds like pedantry and is worth ten seconds of your attention, because the words appear in every error message and every piece of documentation you will ever read. The parameter is the promise. The argument is the thing that keeps it.

The function is now better in a way you can demonstrate. It works on any list of games, not just the one it was written beside:

```js
const twoPlayerGames = [
  { title: 'Hare and Hound', minPlayers: 2, minutes: 20, isComplete: true },
  { title: 'Rope Bridge', minPlayers: 2, minutes: 40, isComplete: true },
];

console.log(totalMinutes(shelf));
console.log(totalMinutes(twoPlayerGames));
```

```
285
60
```

Same function, two answers. That is the payoff.

### More than one parameter, and order matters

```js
function fitsTheEvening(game, minutesAvailable) {
  return game.minutes <= minutesAvailable;
}

console.log(fitsTheEvening(shelf[0], 60));
console.log(fitsTheEvening(shelf[1], 60));
```

```
false
true
```

Arguments are matched to parameters strictly by position. First to first, second to second. Nothing checks that they make sense, so calling `fitsTheEvening(60, shelf[0])` does not produce a helpful complaint. It produces nonsense, or an error somewhere further in, about a line that looked fine.

### When an argument is missing

Leave one out and the parameter is `undefined`.

```js
console.log(fitsTheEvening(shelf[1]));
```

```
false
```

`20 <= undefined` is false, so you get an answer that looks like a real answer and is not one. Nothing was reported as wrong.

A default value protects against this:

```js
function fitsTheEvening(game, minutesAvailable = 60) {
  return game.minutes <= minutesAvailable;
}

console.log(fitsTheEvening(shelf[1]));
console.log(fitsTheEvening(shelf[0], 120));
```

```
true
true
```

The default is used only when no argument arrives for that parameter. Supply one and yours wins.

Defaults belong on parameters that are genuinely optional, where there is a sensible ordinary value. They do not belong on parameters you simply forgot to pass, because then a missing argument turns into a wrong answer rather than a loud complaint.

---

## Part 5: What a function can see

A variable declared inside a function exists only inside that function.

```js
function totalMinutes(games) {
  let total = 0;

  for (let i = 0; i < games.length; i++) {
    total = total + games[i].minutes;
  }

  return total;
}

totalMinutes(shelf);

console.log(total);
```

```
Uncaught ReferenceError: total is not defined
```

`total` was created when the function was called and destroyed when it finished. This is called local scope, and it is a feature rather than a restriction. It means you can use `total` and `i` inside a function without checking whether anything else in your program already uses those names.

Looking the other way, a function **can** see variables declared outside it. That is why the broken version in Part 1 worked at all:

```js
const hallName = 'Bygdehuset';

function printHeading() {
  console.log('Game night at ' + hallName);
}

printHeading();
```

```
Game night at Bygdehuset
```

Which brings us to the practical rule, and it is a habit rather than a fact:

**Pass in what the function needs, rather than reaching out for it.**

A function that only touches its own parameters and its own variables can be read on its own, moved to another file, and tested by calling it with different arguments. A function that reaches outside can only be understood by also reading everything around it.

There are sensible exceptions, and `hallName` above is one: a value that genuinely belongs to the whole program and never changes. The rule is about the working data, not about every last constant.

<details>

<summary>Rabbit hole: two variables with the same name</summary>

What if a function declares a variable with a name that is already in use outside?

```js
let status = 'Shelf tidy';

function borrowGame() {
  let status = 'Someone is browsing';

  console.log('Inside the function: ' + status);
}

console.log('Before: ' + status);
borrowGame();
console.log('After: ' + status);
```

```
Before: Shelf tidy
Inside the function: Someone is browsing
After: Shelf tidy
```

The inner `status` is a completely separate variable that happens to share a name. Inside the function it hides the outer one, and the outer one is entirely unaffected. This is called shadowing.

It is not an error and it is occasionally useful, but it is a reliable source of the sentence "but I changed it, why is it still the old value". If you are ever staring at that, check whether you have two variables where you thought you had one.

</details>

---

## Part 6: Reading arrow functions

There is a second way to write a function, and the reason to learn it now is not that you need it. It is that you are about to start seeing it everywhere, and in Module 3 you will meet array methods where the arrow form is what everybody writes.

So the aim of this part is **reading**, not writing. Here is one function in four costumes.

```js
function doubleMinutes(minutes) {
  return minutes * 2;
}
```

```js
const doubleMinutes = (minutes) => {
  return minutes * 2;
};
```

```js
const doubleMinutes = minutes => {
  return minutes * 2;
};
```

```js
const doubleMinutes = (minutes) => minutes * 2;
```

All four do exactly the same thing, and all four are called the same way:

```js
console.log(doubleMinutes(45));
```

```
90
```

What changed, one step at a time:

- **First to second.** The `function` keyword goes, an arrow `=>` appears after the brackets, and the whole thing is assigned to a `const`. Note the semicolon at the end now, because this is an assignment.
- **Second to third.** With exactly one parameter, the brackets around it are optional. With zero or two or more, they are required.
- **Third to fourth.** When the entire body is one expression that you want to return, you can drop the curly braces and the word `return`. The value of that expression is handed back automatically.

That last step is the one that trips people up when reading. **No braces means there is a hidden `return`.** These two are not the same:

```js
const doubleMinutes = (minutes) => minutes * 2;
```

```js
const doubleMinutes = (minutes) => { minutes * 2; };
```

The first returns the doubled number. The second calculates it, throws it away, and returns `undefined`, because once you write curly braces you are back to needing the word `return`.

For your own code, use whichever you find clearer. Both are fine, and neither is going away.

<details>

<summary>Rabbit hole: one real difference between the two forms</summary>

This works:

```js
console.log(doubleMinutes(45));

function doubleMinutes(minutes) {
  return minutes * 2;
}
```

```
90
```

The function was called on the first line and not defined until the third. JavaScript reads the whole file before running it and sets up function declarations in advance, so a declaration can be used before the point where it appears.

The arrow version does not get that treatment, because it is a `const` being assigned a value, and a `const` does not exist until its line runs:

```js
console.log(doubleMinutes(45));

const doubleMinutes = (minutes) => minutes * 2;
```

```
Uncaught ReferenceError: Cannot access 'doubleMinutes' before initialization
```

Best not to rely on this either way. Define your functions before you use them and the question never comes up.

There is a second difference, involving a keyword called `this`, which matters a great deal in some situations and not at all in anything you will write for a while. It is mentioned here only so you know it exists.

</details>

---

## Part 7: What `&&` and `||` actually hand back

You met `&&` briefly in Module 1, because the Module 1 task could not be finished without it. Here it gets the treatment it deserves, and the first thing to say is that it is stranger than it looked.

### Short-circuiting is the whole story

`&&` needs both sides to be true. So if the left side is false, the answer is settled and there is nothing to gain from looking at the right side. JavaScript does not look.

`||` needs one side to be true. So if the left side is true, the answer is settled, and again JavaScript stops.

That is called short-circuiting, and once you have it, everything else in this lesson follows.

You can watch it happen. Both lines below start with something false, and the right-hand side of the second one never runs:

```js
function isComplete(game) {
  console.log('Checking the box for ' + game.title);
  return game.isComplete;
}

const players = 1;

console.log(players >= 2 && isComplete(shelf[0]));
```

```
false
```

`Checking the box` never printed. `players >= 2` was false, the answer was settled, and `isComplete` was never called.

This is not just a saving. It is a technique. It lets you put a cheap or protective check on the left and something that would break otherwise on the right:

```js
const missing = shelf[99];

console.log(missing !== undefined && missing.minutes > 30);
```

```
false
```

Without the left-hand guard, `missing.minutes` would throw an error, because you cannot read a property of `undefined`. With it, JavaScript never gets there. Order matters enormously in a condition, and this is why.

### The part that surprises people

Now the strange bit. `&&` and `||` do not hand back `true` or `false`. They hand back **one of the two things you gave them**.

```js
console.log(0 || 'Guest');
console.log('Kari' || 'Guest');
console.log(true && 'Fjord Traders');
console.log(false && 'Fjord Traders');
```

```
Guest
Kari
Fjord Traders
false
```

Not one of those four is `true` or `false` except the last, and that is only because `false` was one of the things handed in.

The rule that produces all of it:

- `||` gives back the **first truthy value** it finds. If neither is truthy, it gives back the last one.
- `&&` gives back the **first falsy value** it finds. If neither is falsy, it gives back the last one.

Truthy and falsy are the ideas from Module 1: `false`, `0`, `''`, `null`, `undefined` and `NaN` are falsy, and everything else is truthy.

Nothing you learnt in Module 1 becomes wrong, because when both sides are already `true` or `false` these rules produce exactly the truth table you would expect. It is only when you feed in strings and numbers that the difference shows.

### Which is why this works

```js
let visitorName = '';

const displayName = visitorName || 'Guest';

console.log('Welcome, ' + displayName);
```

```
Welcome, Guest
```

`''` is falsy, so `||` moved on and handed back `'Guest'`. Give `visitorName` a real value and it hands that back instead. A default value in one line, and it only makes sense once you know that `||` returns an operand rather than a boolean.

Be careful with numbers, for the reason covered in Module 1:

```js
const gamesPlayed = 0;

console.log(gamesPlayed || 'no games recorded');
```

```
no games recorded
```

Zero is a real, meaningful count, and `||` threw it away because zero is falsy. This trap catches everybody at least once.

<details>

<summary>Rabbit hole: this is not the same as a default parameter</summary>

Part 4 had default parameters, and they look like they do the same job. They do not.

```js
function greet(name = 'Guest') {
  return 'Welcome, ' + name;
}

function greetOr(name) {
  return 'Welcome, ' + (name || 'Guest');
}

console.log(greet(''));
console.log(greetOr(''));
```

```
Welcome, 
Welcome, Guest
```

A default parameter fires only when the argument is **missing**. An empty string is not missing, so `greet` used it and produced a greeting to nobody.

`||` fires whenever the value is **falsy**, and an empty string is falsy, so `greetOr` replaced it.

Neither is right in general. Which you want depends on whether an empty string is a real answer or a sign that something went wrong. What matters is knowing they differ, because they are easy to mistake for each other.

</details>

### `!` and the two shorthand operators

`!` flips truthy to `false` and falsy to `true`.

```js
console.log(!true);
console.log(!'');
console.log(!shelf[2].isComplete);
```

```
false
true
true
```

That last one reads nicely: not complete. As noted back in Module 1, `!game.isComplete` says the same thing as `game.isComplete === false` and reads better, provided the boolean was named as a yes or no question in the first place.

Finally, two shorthands that exist entirely because of short-circuiting.

`||=` assigns only if the current value is falsy. It is the default-setter from a moment ago, written shorter:

```js
let visitorName = '';

visitorName ||= 'Guest';

console.log(visitorName);
```

```
Guest
```

`&&=` assigns only if the current value is truthy. It updates something that is already there, and leaves a missing value missing:

```js
let currentGame = 'Snow Line';
let nextGame = '';

currentGame &&= currentGame + ' (in progress)';
nextGame &&= nextGame + ' (in progress)';

console.log(currentGame);
console.log('Next: [' + nextGame + ']');
```

```
Snow Line (in progress)
Next: []
```

`currentGame` had something in it, so it was updated. `nextGame` was empty, so `&&=` left it alone rather than producing the nonsense of `' (in progress)'` with nothing in front of it.

Use `||=` to fill in a missing value. Use `&&=` to change a value only if there is one there to change.

### One caution about mixing them

`&&` is evaluated before `||`, in the same way that multiplication happens before addition. So this:

```js
const enoughPlayers = false;
const gameIsShort = true;
const hostIsKeen = true;

console.log(enoughPlayers && gameIsShort || hostIsKeen);
```

```
true
```

is read as `(enoughPlayers && gameIsShort) || hostIsKeen`, not as `enoughPlayers && (gameIsShort || hostIsKeen)`, which would give `false`.

Rather than remembering that, add the brackets. They cost nothing, they remove all doubt, and the person reading your code in three weeks does not have to remember it either.

---

## Exercises

Try each one before opening the solution.

### Exercise 1: Which kind is it?

For each description, say whether you would write a function that **answers a question** or one that **does a thing**, and give it a name.

1. Work out how many games on the shelf are missing pieces.
2. Show the shelf in the console as a numbered list.
3. Decide whether a particular game can be played by four people.
4. Put a returned game back on the shelf.
5. Work out the average playing time of a list of games.

<details>

<summary>Solution</summary>

1. **Answers a question.** `countIncompleteGames`. Hands back a number, prints nothing.
2. **Does a thing.** `printShelf`. Takes the games, returns nothing.
3. **Answers a question.** `canPlayWith`. Hands back `true` or `false`.
4. **Does a thing.** `addGame`, and specifically one that changes what it was given, which is the third case from the rabbit hole in Part 3.
5. **Answers a question.** `averageMinutes`.

Notice that the names for the question-answering ones are nouns or noun phrases, or start with words like `is` and `can`, because you will end up writing `const total = countIncompleteGames(shelf)` and that should read like an assignment of a value. The names for the doing ones are verbs, because you will write `printShelf(shelf)` on a line by itself, and that should read like an instruction.

</details>

### Exercise 2: Print, or return?

Here is a function that does both.

```js
function averageMinutes(games) {
  let total = 0;

  for (let i = 0; i < games.length; i++) {
    total = total + games[i].minutes;
  }

  console.log('Average playing time: ' + (total / games.length));
}
```

1. Rewrite it so it answers a question instead of printing.
2. Then write a separate function that does the printing, using the first one.
3. Then use the first one for something the original version could not do: print a different message depending on whether the average is above or below 45 minutes.

<details>

<summary>Solution</summary>

```js
function averageMinutes(games) {
  let total = 0;

  for (let i = 0; i < games.length; i++) {
    total = total + games[i].minutes;
  }

  return total / games.length;
}

function printAverage(games) {
  console.log('Average playing time: ' + averageMinutes(games) + ' minutes.');
}

printAverage(shelf);

if (averageMinutes(shelf) > 45) {
  console.log('This shelf is for people with an evening to spare.');
} else {
  console.log('Plenty here for a quick game.');
}
```

```
Average playing time: 47.5 minutes.
This shelf is for people with an evening to spare.
```

Step 3 is the whole argument for the split. The original function had the average in its hands and threw it at the screen, so there was nothing left to make a decision with. Once it returns instead, the same calculation can feed a message, a comparison, a further sum, or nothing at all.

The printing did not disappear. It moved into a function whose only job is printing, which means you can change the wording without going anywhere near the arithmetic.

</details>

### Exercise 3: Fix the reaching

This function works, and it has the second problem from Part 1.

```js
const shelf = [
  { title: 'Fjord Traders', minPlayers: 3, minutes: 90, isComplete: true },
  { title: 'Hare and Hound', minPlayers: 2, minutes: 20, isComplete: true },
];

function longestGame() {
  let longest = shelf[0];

  for (let i = 1; i < shelf.length; i++) {
    if (shelf[i].minutes > longest.minutes) {
      longest = shelf[i];
    }
  }

  return longest.title;
}
```

1. Rewrite it so the list of games arrives as a parameter.
2. Prove the rewrite was worth doing by calling it on a second, different list.
3. There is a case where this function goes wrong no matter which version you use. What is it, and what would a guard clause do about it?

<details>

<summary>Solution</summary>

```js
function longestGame(games) {
  let longest = games[0];

  for (let i = 1; i < games.length; i++) {
    if (games[i].minutes > longest.minutes) {
      longest = games[i];
    }
  }

  return longest.title;
}

const shortGames = [
  { title: 'Cabin Fever', minPlayers: 3, minutes: 30, isComplete: false },
  { title: 'Rope Bridge', minPlayers: 2, minutes: 40, isComplete: true },
];

console.log(longestGame(shelf));
console.log(longestGame(shortGames));
```

```
Fjord Traders
Rope Bridge
```

**Step 3.** An empty list. `games[0]` is `undefined`, the loop never runs, and `longest.title` tries to read a property of `undefined`, which is a real error that stops your program.

```js
function longestGame(games) {
  if (games.length === 0) {
    return 'No games';
  }

  let longest = games[0];

  for (let i = 1; i < games.length; i++) {
    if (games[i].minutes > longest.minutes) {
      longest = games[i];
    }
  }

  return longest.title;
}

console.log(longestGame([]));
```

```
No games
```

Notice this version starts from `games[0]` and loops from `i = 1`, rather than starting from zero minutes as we did in the previous extra lesson. Both work here. Starting from the first item is the more honest version, because it does not assume anything about how big or small the values might be, and it is why the empty list needs guarding.

</details>

### Exercise 4: Read the arrows

Each of these is a working function. For each one, say what it hands back when called with the number 40, and rewrite it as an ordinary function declaration.

```js
const a = (minutes) => minutes + 10;
```

```js
const b = (minutes) => {
  return minutes + 10;
};
```

```js
const c = minutes => minutes > 30;
```

```js
const d = (minutes) => { minutes + 10; };
```

<details>

<summary>Solution</summary>

```
a(40)   50       implicit return, no braces
b(40)   50       explicit return, exactly the same result
c(40)   true     one parameter, brackets omitted, implicit return of a comparison
d(40)   undefined
```

`d` is the trap. The braces mean the body is a block, and a block needs the word `return`. The addition is performed and the result is dropped on the floor.

As declarations:

```js
function a(minutes) {
  return minutes + 10;
}

function b(minutes) {
  return minutes + 10;
}

function c(minutes) {
  return minutes > 30;
}

function d(minutes) {
  minutes + 10;
}
```

Written out like this, `d` is obviously wrong in a way it was not when it was one line long. That is the risk of the short forms, and the reason this part of the lesson is about reading them carefully.

</details>

### Exercise 5: What does it hand back?

Write down what each line prints. Several of them are not `true` or `false`.

```js
console.log(true && false);
```

```js
console.log('Snow Line' && 'Rope Bridge');
```

```js
console.log('' || 'Guest');
```

```js
console.log(0 || 45);
```

```js
console.log(45 || 0);
```

```js
console.log(null && 'Rope Bridge');
```

```js
console.log(!'Snow Line');
```

<details>

<summary>Solution</summary>

```
true && false                    false      first falsy value
'Snow Line' && 'Rope Bridge'     Rope Bridge   neither is falsy, so the last one
'' || 'Guest'                    Guest      first truthy value
0 || 45                          45         0 is falsy, so it moves on
45 || 0                          45         45 is truthy, so it stops there
null && 'Rope Bridge'            null       first falsy value, and it stops immediately
!'Snow Line'                     false      a non-empty string is truthy, and ! flips it
```

The pair in the middle is the one to sit with. `0 || 45` and `45 || 0` both give 45, but for opposite reasons, and neither gives you a boolean.

Line six is worth noticing too. `&&` stopped at `null` and never looked at the string, which is exactly the short-circuiting behaviour that makes the guard pattern work.

</details>

### Exercise 6: Guard the lookup

This line causes an error:

```js
const chosen = shelf[99];

console.log(chosen.title + ' takes ' + chosen.minutes + ' minutes.');
```

```
Uncaught TypeError: Cannot read properties of undefined (reading 'title')
```

1. Explain in one sentence what went wrong.
2. Fix it with an `if` statement.
3. Fix it a second way, using `&&` and short-circuiting, in a single line.
4. Which of the two would you rather read, and why?

<details>

<summary>Solution</summary>

**Step 1.** There is no game 99 places along the shelf, so `shelf[99]` gave `undefined`, and `undefined` has no `title` to read.

**Step 2:**

```js
const chosen = shelf[99];

if (chosen === undefined) {
  console.log('No such game on the shelf.');
} else {
  console.log(chosen.title + ' takes ' + chosen.minutes + ' minutes.');
}
```

```
No such game on the shelf.
```

**Step 3:**

```js
const chosen = shelf[99];

console.log(chosen !== undefined && chosen.minutes > 30);
```

```
false
```

`&&` checked the left side, found it false, and never evaluated `chosen.minutes`. No error.

**Step 4.** There is no single right answer, and the useful observation is that the two do different things.

The `if` version handles the missing case with a sensible message. The `&&` version quietly produces `false` and tells nobody. That is fine when you genuinely want a yes or no answer and a missing game counts as no, and it is a bad idea when the missing game was itself the problem you needed to know about.

Short-circuiting protects against the error. It does not tell you the error nearly happened.

</details>

---

## Task: the game night picker

This is an alternative to the Module 2 task. Same skills, different application, and no to-do list anywhere.

People turn up at the village hall, and somebody has to pick a game. Your program takes the number of people present and how long the evening is, and works out which games on the shelf will actually do.

### Brief

1. Create a file called `game-night.js`, or work in the console.

2. Start with the shelf from the top of this lesson, as a `const`.

3. **Write a function that answers a question.** Call it `gameFits`. It takes three things: a game, the number of players, and the minutes available. It returns `true` if all three of these hold, and `false` otherwise:
   - the game's `minPlayers` is not more than the number of players present
   - the game's `minutes` is not more than the minutes available
   - the game has all its pieces

   This function prints nothing.

4. **Write a second function that answers a question.** Call it `findGames`. It takes the shelf, the number of players, and the minutes available, and it returns a **new array** containing only the games that fit. Start with an empty array, loop the shelf, use `gameFits` on each game, and `push` the ones that pass.

   Give the minutes parameter a default of 60, so that `findGames(shelf, 3)` means "three of us, an ordinary evening".

5. **Write a function that does a thing.** Call it `printGames`. It takes an array of games and a heading, and prints the heading followed by a numbered list of titles with their playing times. It returns nothing.

   Give it a guard clause: if the array is empty, print something sensible and leave.

6. Use all three together. Print the results for:
   - three players with 60 minutes
   - four players with 120 minutes
   - one player, with the minutes left to the default

Work out on paper which games should come back in each case before you run anything. Every number you need is in the shelf at the top of this lesson.

### Challenges

**One.** Add a function `totalMinutesFor` that takes an array of games and returns the total playing time, then print how long it would take to play everything that fits the four-player evening.

**Two.** `findGames` currently ignores games with missing pieces entirely. Add a fourth parameter, `allowIncomplete`, defaulting to `false`. When it is `true`, games with missing pieces are allowed through. You will need to change the condition inside `gameFits`, and you may find `||` useful.

**Three.** Rewrite `gameFits` as an arrow function with an implicit return, then check that everything still works. This is a good test of whether Part 6 landed.

<details>

<summary>Solution</summary>

```js
const shelf = [
  { title: 'Fjord Traders', minPlayers: 3, minutes: 90, isComplete: true },
  { title: 'Hare and Hound', minPlayers: 2, minutes: 20, isComplete: true },
  { title: 'The Lighthouse Keeper', minPlayers: 2, minutes: 45, isComplete: false },
  { title: 'Snow Line', minPlayers: 4, minutes: 60, isComplete: true },
  { title: 'Cabin Fever', minPlayers: 3, minutes: 30, isComplete: false },
  { title: 'Rope Bridge', minPlayers: 2, minutes: 40, isComplete: true },
];

function gameFits(game, players, minutesAvailable) {
  const enoughPlayers = game.minPlayers <= players;
  const shortEnough = game.minutes <= minutesAvailable;

  return enoughPlayers && shortEnough && game.isComplete;
}

function findGames(games, players, minutesAvailable = 60) {
  const suitable = [];

  for (let i = 0; i < games.length; i++) {
    if (gameFits(games[i], players, minutesAvailable)) {
      suitable.push(games[i]);
    }
  }

  return suitable;
}

function printGames(games, heading) {
  console.log(heading);

  if (games.length === 0) {
    console.log('Nothing on the shelf fits. Suggest a walk instead.');
    return;
  }

  for (let i = 0; i < games.length; i++) {
    console.log((i + 1) + '. ' + games[i].title + ' (' + games[i].minutes + ' minutes)');
  }
}

printGames(findGames(shelf, 3, 60), 'Three of us, an hour:');
printGames(findGames(shelf, 4, 120), 'Four of us, a long evening:');
printGames(findGames(shelf, 1), 'On my own:');
```

```
Three of us, an hour:
1. Hare and Hound (20 minutes)
2. Rope Bridge (40 minutes)
Four of us, a long evening:
1. Fjord Traders (90 minutes)
2. Hare and Hound (20 minutes)
3. Snow Line (60 minutes)
4. Rope Bridge (40 minutes)
On my own:
Nothing on the shelf fits. Suggest a walk instead.
```

Four things worth pulling out.

**The three functions are three different kinds.** `gameFits` answers a question about one game. `findGames` answers a question about a whole shelf, and it uses `gameFits` to do it. `printGames` does a thing. None of them does two jobs, which is why they combine so easily on those last three lines.

**`findGames` builds a new array rather than changing the shelf.** The shelf is exactly as long after those three calls as it was before. Returning something new instead of altering what you were given is almost always the safer choice.

**The guard clause in `printGames` uses a bare `return`.** No value after it. That is allowed, and it means "I am finished, there is nothing to hand back", which is exactly right for a function whose job is doing rather than answering.

**Named conditions inside `gameFits`.** `enoughPlayers && shortEnough && game.isComplete` reads as a sentence. The one-line version would work identically, and would be three comparisons long and much harder to fault-find when it starts returning the wrong thing.

**Challenge one:**

```js
function totalMinutesFor(games) {
  let total = 0;

  for (let i = 0; i < games.length; i++) {
    total = total + games[i].minutes;
  }

  return total;
}

const longEvening = findGames(shelf, 4, 120);

console.log('That is ' + totalMinutesFor(longEvening) + ' minutes of games.');
```

```
That is 210 minutes of games.
```

**Challenge two:**

```js
function gameFits(game, players, minutesAvailable, allowIncomplete = false) {
  const enoughPlayers = game.minPlayers <= players;
  const shortEnough = game.minutes <= minutesAvailable;
  const piecesAreFine = game.isComplete || allowIncomplete;

  return enoughPlayers && shortEnough && piecesAreFine;
}

function findGames(games, players, minutesAvailable = 60, allowIncomplete = false) {
  const suitable = [];

  for (let i = 0; i < games.length; i++) {
    if (gameFits(games[i], players, minutesAvailable, allowIncomplete)) {
      suitable.push(games[i]);
    }
  }

  return suitable;
}

printGames(findGames(shelf, 3, 60, true), 'Three of us, missing pieces allowed:');
```

```
Three of us, missing pieces allowed:
1. Hare and Hound (20 minutes)
2. The Lighthouse Keeper (45 minutes)
3. Cabin Fever (30 minutes)
4. Rope Bridge (40 minutes)
```

`game.isComplete || allowIncomplete` reads as: the pieces are all there, or we do not mind. Either one is enough, which is exactly what `||` is for.

**Challenge three:**

```js
const gameFits = (game, players, minutesAvailable) =>
  game.minPlayers <= players && game.minutes <= minutesAvailable && game.isComplete;
```

It works, and it is shorter, and it is worse. The named conditions are gone and the line has to be read all the way to the end before you know what it is deciding. Short is not the same as clear, and a function like this one is a good place to notice the difference.

One practical warning if you try this: an arrow function assigned to a `const` cannot be used before the line that creates it, so `gameFits` must now appear above `findGames` in the file. The declaration version did not care.

</details>

---

## Where this leaves you

A function either answers a question or does a thing. Question-answering functions return and print nothing. Doing functions take what they need and return nothing. When you cannot name one without the word "and", it is probably two functions.

A parameter is a promise about what you will be given, and the habit that follows is to pass in what a function needs rather than reaching outside for it. A function that only touches its own parameters can be read, moved and tested on its own.

`return` hands a value back and ends the function, which is what makes the guard clause work: deal with the awkward case first, leave early, then do the real work without an `else` wrapped round it.

Arrow functions are the same thing in shorter clothes, and the one to watch when reading them is the version with no braces, where the `return` is hidden.

And `&&` and `||` stop as soon as the answer is settled, and hand back one of the values you gave them rather than a boolean. That single fact explains the guard pattern, the default-value pattern, `||=`, `&&=`, and the trap where a perfectly good `0` gets thrown away.

That is Module 2. You now have somewhere to put data, and something to do to it.
