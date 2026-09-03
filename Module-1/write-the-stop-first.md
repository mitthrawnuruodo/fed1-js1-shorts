# Extra lesson: Write the stop first

This is an optional extra lesson covering the same ground as lessons 1.3 and 1.4, by a different road.

The regular lessons treat decisions and loops as two separate topics. This one treats them as one, because they are. Three sentences carry it:

**1. A condition is a question with only two possible answers.**

**2. A loop is that same question asked over and over, and something inside has to change the answer.**

**3. Write the stop before you write the work.**

Everything in this lesson uses one running example: walking a marked route with a fixed amount of daylight left. No dice, no random numbers. Every run gives the same answer, so you can work out on paper what your code should print and then check whether it agrees with you.

---

## Part 1: Name the question

A comparison is a question. JavaScript answers it with `true` or `false`, and nothing else.

```js
console.log(240 >= 45);
console.log(240 === 45);
console.log(240 !== 45);
```

```
true
false
true
```

That answer is a value like any other, so you can give it a name:

```js
const minutesOfLight = 240;
const stageMinutes = 45;

const hasLightForAnotherStage = minutesOfLight >= stageMinutes;

console.log(hasLightForAnotherStage);
```

```
true
```

`hasLightForAnotherStage` is a boolean, the third type from the previous lesson. Check it if you like:

```js
console.log(typeof hasLightForAnotherStage);
```

```
boolean
```

**Naming your questions is the single habit worth taking from this lesson.** You will see code written like this:

```js
if (minutesOfLight >= stageMinutes) {
```

and code written like this:

```js
if (hasLightForAnotherStage) {
```

Both work. The second one tells the reader what the question means rather than only how it is calculated, and when conditions get longer, the difference stops being cosmetic.

### The comparison operators

Six of them, and you have already met two.

| Operator | Question |
| --- | --- |
| `===` | are these exactly the same? |
| `!==` | are these different? |
| `>` | is the left bigger? |
| `<` | is the left smaller? |
| `>=` | is the left bigger, or the same? |
| `<=` | is the left smaller, or the same? |

The two you will reach for most often are `>=` and `===`.

Use three equals signs, always. Two equals signs (`==`) also exists, and it tries to be helpful by converting types before comparing, so `240 == '240'` is `true`. That is the same helpfulness that gave you `'6' + 4` in the previous lesson, and it causes the same kind of bug. `===` compares the value and the type and does no converting. There is no situation in this course where you need `==`.

<details>

<summary>Rabbit hole: one equals sign in a condition</summary>

One equals sign is not a comparison at all. It is assignment: put this value into this name.

```js
let stagesWalked = 0;   // assignment. Puts 0 into stagesWalked.
stagesWalked === 0      // comparison. Asks whether stagesWalked is 0.
```

Mixing them up inside an `if` is a classic. This looks like a question but is not one:

```js
if (stagesWalked = 5) {
```

That line sets `stagesWalked` to 5, and then asks whether 5 counts as true. It does, so the block always runs, and your counter has been quietly overwritten into the bargain.

Modern editors will underline this in yellow and ask whether you meant `===`. When you see that warning, you almost always did.

Read them out loud as you type: one equals is "becomes", three equals is "is the same as".

</details>

---

## Part 2: if and else

An `if` runs a block of code only when a condition is true.

```js
const minutesOfLight = 240;
const stageMinutes = 45;

const hasLightForAnotherStage = minutesOfLight >= stageMinutes;

if (hasLightForAnotherStage) {
  console.log('Carry on to the next waymarker.');
}
```

```
Carry on to the next waymarker.
```

The curly braces mark where the block starts and stops. Everything inside them is skipped entirely when the condition is false. Indent the contents by two spaces so you can see at a glance what belongs to the `if`.

`else` gives you the other path:

```js
const minutesOfLight = 30;
const stageMinutes = 45;

if (minutesOfLight >= stageMinutes) {
  console.log('Carry on to the next waymarker.');
} else {
  console.log('Stop here and put the tent up.');
}
```

```
Stop here and put the tent up.
```

One of the two blocks runs. Never both, never neither.

<details>

<summary>Rabbit hole: what happens if you put something that is not a question in there?</summary>

JavaScript will accept anything at all inside the brackets of an `if` and decide whether it counts as true. Most values count as true. A short list of values count as false:

`false`, `0`, `''` (an empty string), `null`, `undefined`, `NaN`.

Everything else counts as true, including negative numbers and the string `'false'`.

This gets used as a shortcut for "does this have anything in it":

```js
if (walkerName) {
  console.log('Hello, ' + walkerName);
}
```

Handy for text. A trap for numbers, because `0` counts as false:

```js
let minutesOfLight = 0;

if (minutesOfLight) {
  console.log('There is light left');   // never runs
}
```

Zero minutes of light is a real, meaningful measurement, and this code treats it as missing data. Ask the question you actually mean:

```js
if (minutesOfLight > 0) {
```

The rule of thumb: use the shortcut for text, and write the comparison out in full for numbers.

</details>

---

## Part 3: Joining questions together

Real decisions rarely rest on one thing. You want to walk on only if there is enough light **and** you are not too tired. There are three operators for combining questions.

These get a lesson of their own in Module 2, where you will see more of what they can do. Module 1 needs them sooner than that, because the Module 1 task cannot be finished without `&&`, so here is enough to be going on with.

**`&&` means and.** True only when both sides are true.

```js
const minutesOfLight = 240;
const stageMinutes = 45;
const isExhausted = false;

const shouldWalkOn = minutesOfLight >= stageMinutes && isExhausted === false;

console.log(shouldWalkOn);
```

```
true
```

**`||` means or.** True when either side is true. Those are two pipe characters, the vertical bars.

```js
const minutesOfLight = 30;
const stageMinutes = 45;
const isAtCabin = true;

const shouldStop = minutesOfLight < stageMinutes || isAtCabin;

console.log(shouldStop);
```

```
true
```

You would stop for either reason. Here, both happen to be true, which is fine. `||` only needs one.

**`!` means not.** It flips a true into a false and back.

```js
const isExhausted = false;

console.log(!isExhausted);
```

```
true
```

`!` lets you write that earlier condition more naturally:

```js
const shouldWalkOn = minutesOfLight >= stageMinutes && !isExhausted;
```

Read it aloud: there is enough light, and you are not exhausted. Note that `!isExhausted` says the same thing as `isExhausted === false`, and reads better. This is another reason to name your booleans well, because `!isExhausted` only reads nicely if the name was a yes or no question in the first place.

A warning about long conditions. This is legal:

```js
if (minutesOfLight >= stageMinutes && !isExhausted && weatherIsClear && !pathIsFlooded) {
```

and it is horrible to read and worse to debug. Break it up and name the pieces:

```js
const hasTimeAndEnergy = minutesOfLight >= stageMinutes && !isExhausted;
const conditionsAreSafe = weatherIsClear && !pathIsFlooded;

if (hasTimeAndEnergy && conditionsAreSafe) {
```

Same logic. Now when it misbehaves you can log the two halves separately and find out which one is lying to you.

That is as much as Module 1 asks of you. When these come round again in Module 2 you will find they are a little stranger than they look here, and rather more useful with it.

---

## Part 4: The ladder

When there are more than two outcomes, chain the conditions with `else if`.

```js
const minutesOfLight = 145;

if (minutesOfLight >= 180) {
  console.log('Plenty of time. Take the long route by the lake.');
} else if (minutesOfLight >= 90) {
  console.log('Enough time for the direct route.');
} else if (minutesOfLight >= 30) {
  console.log('Head straight for the nearest shelter.');
} else {
  console.log('Stop now and get the head torch out.');
}
```

```
Enough time for the direct route.
```

**The rule that matters: first match wins.** JavaScript works down the ladder from the top, and the moment a condition is true it runs that block and abandons the rest. It never looks at the conditions below.

`145 >= 180` is false, so it moves on. `145 >= 90` is true, so it prints and stops. It never even asks about 30, even though `145 >= 30` is also true.

That is why the order of a ladder is part of its logic, not a matter of taste. Rungs have to go from the most demanding condition to the least. Put them the other way round and every walker falls into the first rung:

```js
const minutesOfLight = 240;

if (minutesOfLight >= 30) {
  console.log('Head straight for the nearest shelter.');
} else if (minutesOfLight >= 90) {
  console.log('Enough time for the direct route.');
}
```

```
Head straight for the nearest shelter.
```

Four hours of daylight and it tells you to run for the shelter. Nothing here is broken in a way JavaScript can warn you about. The code is doing exactly what you wrote.

The final `else` is optional, but it is nearly always worth having. It is the rung that catches everything the others missed, and if you leave it out and your ladder has a gap, your program will silently do nothing at all.

---

## Part 5: switch, and the trap in it

Ladders are for ranges. When you are checking one value against a list of exact possibilities, `switch` says the same thing more plainly.

The waymarkers on our route have a single letter painted on them.

```js
const marker = 'W';

switch (marker) {
  case 'C':
    console.log('Cabin ahead.');
    break;
  case 'W':
    console.log('Drinking water here.');
    break;
  case 'J':
    console.log('Junction. Keep left.');
    break;
  default:
    console.log('Unknown marker. Check the map.');
}
```

```
Drinking water here.
```

`switch` compares the value in the brackets against each `case` using `===`, so the same type rules apply. A marker of `'W'` will not match a case of `'w'`.

`default` is the catch-all, the same job as the final `else` on a ladder.

### The trap

`break` means "leave the switch now". Leave one out and execution does not stop at the end of a case. It carries straight on into the next one. This is called fallthrough.

```js
const marker = 'W';

switch (marker) {
  case 'C':
    console.log('Cabin ahead.');
    break;
  case 'W':
    console.log('Drinking water here.');
  case 'J':
    console.log('Junction. Keep left.');
    break;
  default:
    console.log('Unknown marker. Check the map.');
}
```

```
Drinking water here.
Junction. Keep left.
```

One missing `break` and your walker is sent down the wrong path at a junction that is not there. Notice that `default` did not run: fallthrough continues to the next case and stops at the first `break` it meets, wherever that is.

Nothing warns you about this. Get into the habit of typing the `break` at the same moment you type the `case`, before you write what goes in between.

<details>

<summary>Rabbit hole: fallthrough on purpose</summary>

Fallthrough is a feature, not an oversight. It is how you make several values share one outcome.

```js
const marker = 'B';

switch (marker) {
  case 'C':
  case 'S':
  case 'B':
    console.log('Shelter of some kind ahead.');
    break;
  default:
    console.log('Keep walking.');
}
```

```
Shelter of some kind ahead.
```

Cases `'C'` and `'S'` have no code at all, so a match falls straight through to `'B'` and runs its block. This is a tidy way of saying "any of these three".

It is also the reason the accidental version is so easy to write. The language cannot tell the difference between the fallthrough you meant and the `break` you forgot.

</details>

---

## Part 6: Loops, or the same question asked again

Here is the connection this lesson is built on.

```js
if (minutesOfLight >= stageMinutes) {
  // walk a stage
}
```

That asks once. Now imagine asking it again the moment the block finishes, and again after that, until the answer comes back false. That is a loop, and it is very nearly the same code:

```js
while (minutesOfLight >= stageMinutes) {
  // walk a stage
}
```

One word changed. `if` asks once. `while` keeps asking.

Which leads directly to the thing that makes loops dangerous. If the code inside never changes the answer, the answer never changes, and the loop never ends.

### Write the stop first

Before you write a single line of the body, answer three questions:

1. **What has to stay true for this loop to continue?**
2. **Which variable is in that question?**
3. **Where inside the loop does that variable change?**

If you cannot answer the third one, you have written an infinite loop. Answering these three before you start is worth more than any amount of debugging afterwards.

For our walk:

1. There has to be enough light left for another stage.
2. `minutesOfLight`.
3. Each stage subtracts its length from `minutesOfLight`.

Now the loop can be written.

```js
let minutesOfLight = 240;
let stageMinutes = 45;
let stagesWalked = 0;

while (minutesOfLight >= stageMinutes) {
  minutesOfLight = minutesOfLight - stageMinutes;
  stagesWalked = stagesWalked + 1;
  stageMinutes = stageMinutes + 5;

  console.log('Stage ' + stagesWalked + ' done. Light left: ' + minutesOfLight);
}

console.log('Walked ' + stagesWalked + ' stages with ' + minutesOfLight + ' minutes to spare.');
```

```
Stage 1 done. Light left: 195
Stage 2 done. Light left: 145
Stage 3 done. Light left: 90
Stage 4 done. Light left: 30
Walked 4 stages with 30 minutes to spare.
```

The `stageMinutes = stageMinutes + 5` line is the walker slowing down: every stage takes five minutes longer than the one before. Which means two variables are moving towards each other, and the loop ends when they cross.

Note that all three are declared with `let`, not `const`. They have to move, and `const` would stop them.

### The shorthands

`minutesOfLight = minutesOfLight - stageMinutes` says the variable's name twice. This is so common inside loops that there are shorter ways to write it.

```js
minutesOfLight -= stageMinutes;   // same as minutesOfLight = minutesOfLight - stageMinutes
stageMinutes += 5;                // same as stageMinutes = stageMinutes + 5
```

And when you are adding or subtracting exactly one, shorter still:

```js
stagesWalked++;   // same as stagesWalked += 1
stagesWalked--;   // same as stagesWalked -= 1
```

There are `*=` and `/=` too, following the same pattern. The loop body becomes:

```js
while (minutesOfLight >= stageMinutes) {
  minutesOfLight -= stageMinutes;
  stagesWalked++;
  stageMinutes += 5;

  console.log('Stage ' + stagesWalked + ' done. Light left: ' + minutesOfLight);
}
```

Identical behaviour, and now the three moving parts line up where you can see them.

### When you know the count: for

Sometimes you know exactly how many times you want to go round. Printing the plan for a five stage route, for instance.

You could do it with a while loop:

```js
let stage = 1;

while (stage <= 5) {
  console.log('Stage ' + stage + ' takes about ' + (40 + stage * 5) + ' minutes.');
  stage++;
}
```

That works, but the three pieces of the counting are scattered: the start is above the loop, the stop is in the brackets, and the step is buried at the bottom of the body. Miss the last one and you have an infinite loop.

A `for` loop gathers all three onto one line:

```js
for (let stage = 1; stage <= 5; stage++) {
  console.log('Stage ' + stage + ' takes about ' + (40 + stage * 5) + ' minutes.');
}
```

```
Stage 1 takes about 45 minutes.
Stage 2 takes about 50 minutes.
Stage 3 takes about 55 minutes.
Stage 4 takes about 60 minutes.
Stage 5 takes about 65 minutes.
```

The brackets hold three things separated by semicolons:

- `let stage = 1` is the **start**, and it runs once, before anything else.
- `stage <= 5` is the **stop**, checked before every pass. While it is true, the body runs.
- `stage++` is the **step**, and it runs after each pass through the body.

Same three ingredients as the while loop, in the same order, gathered where you cannot forget one.

So which do you use? **Use `for` when you know how many times. Use `while` when you are waiting for something to happen.** Our route walk is a `while`, because nobody knows in advance how many stages fit into 240 minutes. The plan printer is a `for`, because the route has five stages.

<details>

<summary>Rabbit hole: why does everyone call it i, and why start at 0?</summary>

You will see this everywhere:

```js
for (let i = 0; i < 5; i++) {
```

`i` is short for index. The convention is decades old and predates JavaScript, and while a descriptive name like `stage` is friendlier, `i` is so universal that nobody will be confused by it.

Starting at 0 will make sense in a couple of modules, when you meet lists. The positions in a list are numbered from 0, not 1, so a loop that visits every item starts at 0.

Notice the pairing. Starting at 0 goes with `<`, and starting at 1 goes with `<=`. Both of the following go round exactly five times:

```js
for (let i = 0; i < 5; i++)    // 0, 1, 2, 3, 4
for (let i = 1; i <= 5; i++)   // 1, 2, 3, 4, 5
```

Mixing the pairs up gives you four passes or six. This is the most common counting mistake in programming and it has a name, the off-by-one error. When a loop does almost the right thing, check this first.

</details>

### When it has to happen at least once: do while

Our route has no shelter at the car park, so the first stage gets walked whatever the light is doing. You have to leave.

A plain `while` checks before the first pass, so if the light is already short it never moves at all. `do...while` puts the check at the end:

```js
let minutesOfLight = 30;
let stageMinutes = 45;
let stagesWalked = 0;

do {
  minutesOfLight -= stageMinutes;
  stagesWalked++;
} while (minutesOfLight >= stageMinutes);

console.log('Stages walked: ' + stagesWalked);
console.log('Light left: ' + minutesOfLight);
```

```
Stages walked: 1
Light left: -15
```

One stage was walked despite there never having been enough light for it, which is exactly what we asked for. The minus figure is the walker finishing in the dark.

Note the semicolon after the closing bracket. `do...while` is the one loop that needs it.

You will not reach for this often. When you do, it is always the same reason: the action has to happen before you can know whether to repeat it.

### The loop that never ends

```js
let minutesOfLight = 240;
const stageMinutes = 45;

while (minutesOfLight >= stageMinutes) {
  console.log('Walking...');
}
```

Do not run that. `minutesOfLight` starts at 240 and nothing in the body touches it, so the question is asked, answered true, asked again, answered true, forever. The tab will lock up and stop responding, because your loop never gives the browser a moment to do anything else.

If it happens, close the tab. The page is not coming back.

Then go back to question three: where inside the loop does that variable change? Here it does not, because the update was forgotten. The other common version is an update that pushes the variable in the wrong direction, which is why the countdown loop below never ends:

```js
for (let stage = 5; stage > 0; stage++) {
```

The stop wants `stage` to get smaller. The step makes it bigger. It needed `stage--`.

**Every loop should have a line you can point at and say: this is the line that gets me out.** If you cannot point at it, you have not finished writing the loop.

---

## Exercises

Try each one before opening the solution.

### Exercise 1: Name the questions

Each of these `if` statements has its condition written inline. Rewrite each one so the condition is stored in a well named boolean first, then used in the `if`.

```js
if (minutesOfLight < 60) {
  console.log('Get the head torch out.');
}
```

```js
if (marker !== 'C') {
  console.log('Not the cabin yet.');
}
```

```js
if (minutesOfLight >= stageMinutes && isExhausted === false) {
  console.log('Walk on.');
}
```

```js
if (packWeight > 12 || bootsAreWet) {
  console.log('This is going to be a slow one.');
}
```

<details>

<summary>Solution</summary>

```js
const isGettingDark = minutesOfLight < 60;

if (isGettingDark) {
  console.log('Get the head torch out.');
}
```

```js
const isAtCabin = marker === 'C';

if (!isAtCabin) {
  console.log('Not the cabin yet.');
}
```

```js
const canWalkOn = minutesOfLight >= stageMinutes && !isExhausted;

if (canWalkOn) {
  console.log('Walk on.');
}
```

```js
const isHardGoing = packWeight > 12 || bootsAreWet;

if (isHardGoing) {
  console.log('This is going to be a slow one.');
}
```

Two things worth noticing.

In the second one, the name is `isAtCabin` rather than `isNotAtCabin`, and the `if` uses `!`. Naming a boolean for the positive case and flipping it where needed almost always reads better. `if (!isAtCabin)` is plain English. `if (isNotAtCabin)` is fine on its own, but sooner or later you need the opposite and end up with `if (!isNotAtCabin)`, which nobody can read.

In the third, `isExhausted === false` became `!isExhausted`. Comparing a boolean to `true` or `false` is always redundant. The boolean is already the answer.

</details>

### Exercise 2: The broken ladder

This ladder is meant to describe how much light is left. Whatever number you put in, it always gives the same answer.

```js
const minutesOfLight = 200;

if (minutesOfLight > 0) {
  console.log('Some light left.');
} else if (minutesOfLight > 60) {
  console.log('An hour or so.');
} else if (minutesOfLight > 180) {
  console.log('Most of the afternoon.');
} else {
  console.log('Dark.');
}
```

Say what it prints and why, then fix it. Then work out what number, if any, still reaches the final `else`.

<details>

<summary>Solution</summary>

It prints `Some light left.` for 200, and for 1, and for 10000. Any positive number satisfies `minutesOfLight > 0`, and first match wins, so the ladder stops at the top rung every time. The other two rungs are unreachable.

The fix is to reverse the order, most demanding first:

```js
const minutesOfLight = 200;

if (minutesOfLight > 180) {
  console.log('Most of the afternoon.');
} else if (minutesOfLight > 60) {
  console.log('An hour or so.');
} else if (minutesOfLight > 0) {
  console.log('Some light left.');
} else {
  console.log('Dark.');
}
```

```
Most of the afternoon.
```

The final `else` is now reached by `0` and by any negative number, which is what you want: nothing above zero should ever be described as dark.

In the original, the `else` was reachable too, by exactly the same values. That is the awkward part of this bug. Nothing errors, nothing looks obviously wrong, and one of the four branches even works correctly. It is only wrong in the answers it gives, which is why writing down what you expect before you run something matters so much.

</details>

### Exercise 3: The missing break

Predict what each of these prints. Then run them and check.

```js
const marker = 'C';

switch (marker) {
  case 'C':
    console.log('Cabin ahead.');
  case 'W':
    console.log('Drinking water here.');
    break;
  case 'J':
    console.log('Junction. Keep left.');
    break;
  default:
    console.log('Unknown marker.');
}
```

```js
const marker = 'J';

switch (marker) {
  case 'C':
    console.log('Cabin ahead.');
  case 'W':
    console.log('Drinking water here.');
    break;
  case 'J':
    console.log('Junction. Keep left.');
    break;
  default:
    console.log('Unknown marker.');
}
```

```js
const marker = 'c';

switch (marker) {
  case 'C':
    console.log('Cabin ahead.');
    break;
  default:
    console.log('Unknown marker.');
}
```

<details>

<summary>Solution</summary>

**First** prints two lines:

```
Cabin ahead.
Drinking water here.
```

`'C'` matched, its block ran, and with no `break` execution fell through into the `'W'` block and ran that too. It stopped at the `break` in the `'W'` case.

**Second** prints one line:

```
Junction. Keep left.
```

The missing `break` is still there, in the `'C'` case, but nothing matched `'C'` this time so it was never entered. A fallthrough bug only bites when the case above it matches, which is why these survive testing for so long.

**Third** prints:

```
Unknown marker.
```

`switch` compares with `===`, and `'c'` is not `'C'`. Different type, same value, no match here either: `switch (3)` will not match `case '3'`.

The fix for the first two is the same, and it is one line:

```js
  case 'C':
    console.log('Cabin ahead.');
    break;
```

</details>

### Exercise 4: Write the stop first

Here is a loop body with no loop around it. A walker is filling water bottles at a spring, one litre at a time, until the four litre carrier is full.

```js
  litresCarried++;
  console.log('Filled bottle ' + litresCarried + '.');
```

Answer the three questions, then write the loop, including any variables that need to exist beforehand.

1. What has to stay true for this loop to continue?
2. Which variable is in that question?
3. Where inside the loop does that variable change?

Then do the same for this body, where the walker is instead eating into a food supply of 2400 calories at 600 calories a day:

```js
  caloriesLeft -= 600;
  daysFed++;
  console.log('Day ' + daysFed + '. Calories left: ' + caloriesLeft);
```

<details>

<summary>Solution</summary>

**First loop.** The carrier is not yet full, the variable is `litresCarried`, and it changes on the first line of the body.

```js
let litresCarried = 0;

while (litresCarried < 4) {
  litresCarried++;
  console.log('Filled bottle ' + litresCarried + '.');
}
```

```
Filled bottle 1.
Filled bottle 2.
Filled bottle 3.
Filled bottle 4.
```

Since the number of passes is known in advance, a `for` loop is arguably the better fit here:

```js
for (let litresCarried = 1; litresCarried <= 4; litresCarried++) {
  console.log('Filled bottle ' + litresCarried + '.');
}
```

Either is correct. Note the pairing from earlier: `0` with `<`, or `1` with `<=`.

**Second loop.** There has to be at least a day's food left, the variable is `caloriesLeft`, and it changes on the first line.

```js
let caloriesLeft = 2400;
let daysFed = 0;

while (caloriesLeft >= 600) {
  caloriesLeft -= 600;
  daysFed++;
  console.log('Day ' + daysFed + '. Calories left: ' + caloriesLeft);
}
```

```
Day 1. Calories left: 1800
Day 2. Calories left: 1200
Day 3. Calories left: 600
Day 4. Calories left: 0
```

Watch the last pass. When `caloriesLeft` is 600 the condition `600 >= 600` is true, so the fourth day happens and the supply lands exactly on zero. Had the condition been `caloriesLeft > 600`, day four would never have run and a full day of food would have been left uneaten. One character, one day's difference.

</details>

### Exercise 5: Trace it by hand

Do not run this. Work it out on paper first.

```js
let minutesOfLight = 150;
let stageMinutes = 40;
let stagesWalked = 0;

while (minutesOfLight >= stageMinutes) {
  minutesOfLight -= stageMinutes;
  stagesWalked++;
  stageMinutes += 10;
}

console.log(stagesWalked);
console.log(minutesOfLight);
```

Fill in a table with one row per pass. Start with the values before the loop begins, then add a row each time the body runs, and write down the answer to the condition each time it is asked.

| Pass | minutesOfLight | stageMinutes | stagesWalked | condition |
| --- | --- | --- | --- | --- |
| before | 150 | 40 | 0 | |

Then say what the two `console.log` lines print. Only then run it.

<details>

<summary>Solution</summary>

| Pass | minutesOfLight | stageMinutes | stagesWalked | condition asked before this pass |
| --- | --- | --- | --- | --- |
| before | 150 | 40 | 0 | |
| 1 | 110 | 50 | 1 | 150 >= 40, true |
| 2 | 60 | 60 | 2 | 110 >= 50, true |
| 3 | 0 | 70 | 3 | 60 >= 60, true |
| stop | 0 | 70 | 3 | 0 >= 70, false |

```
3
0
```

The third pass is the one that catches people. `60 >= 60` is true, because `>=` includes equality, so the walker sets off on a stage that uses up every remaining minute and arrives at exactly nightfall.

Tracing by hand like this is not a beginner's crutch you grow out of. It is what experienced developers do when a loop misbehaves, and it finds problems that staring at the code does not. Anything you can work out on paper you can also predict, and anything you can predict you can test.

</details>

### Exercise 6: Swap the loops

Rewrite this `while` loop as a `for` loop, without changing what it prints.

```js
let marker = 10;

while (marker <= 50) {
  console.log('Waymarker at ' + marker + ' km.');
  marker += 10;
}
```

Then rewrite this `for` loop as a `while` loop.

```js
for (let stage = 6; stage >= 1; stage--) {
  console.log('Stage ' + stage + ' remaining.');
}
```

<details>

<summary>Solution</summary>

The while, as a for:

```js
for (let marker = 10; marker <= 50; marker += 10) {
  console.log('Waymarker at ' + marker + ' km.');
}
```

```
Waymarker at 10 km.
Waymarker at 20 km.
Waymarker at 30 km.
Waymarker at 40 km.
Waymarker at 50 km.
```

The step does not have to be `++`. Any expression that moves the counter will do.

The for, as a while:

```js
let stage = 6;

while (stage >= 1) {
  console.log('Stage ' + stage + ' remaining.');
  stage--;
}
```

```
Stage 6 remaining.
Stage 5 remaining.
Stage 4 remaining.
Stage 3 remaining.
Stage 2 remaining.
Stage 1 remaining.
```

The three pieces did not change, only where they are written. Start above the loop, stop in the brackets, step at the bottom of the body.

There is one real difference. In the `for` version, `stage` exists only inside the loop, and trying to use it afterwards is an error. In the `while` version it is declared outside, so it survives, holding `0`. That occasionally matters, and when it does, it is usually a reason to prefer the `while`.

</details>

---

## Task: the route planner

This is an alternative to the Module 1 task. It uses the same skills as the guessing game, but every run gives the same result, so you can check your own work without playing it twenty times.

You are planning a walk. There is a fixed amount of daylight left, the route has a fixed number of stages, and each stage takes longer than the last because you get slower as you go. The program works out how far you get before the light runs out, and whether that is far enough to reach the cabin.

### Brief

1. Create a file called `route-planner.js` and link it to an HTML page with a `<script>` tag, or paste it straight into the console.

2. Set up your starting values. Use `const` for the ones that never change and `let` for the ones that move.

   - `minutesOfLight`, starting at 300
   - `stageMinutes`, starting at 45
   - `stagesInRoute`, a constant, 6
   - `stagesWalked`, starting at 0

3. Print a plan before setting off. Use a `for` loop counting from 1 to `stagesInRoute` that prints one line per stage saying how long that stage should take. Stage one takes 45 minutes and each stage after takes 5 minutes longer, so you will need to work the number out from the counter.

4. Now walk the route. Use a `while` loop that keeps going as long as there is enough light left for the next stage **and** you have not already finished the route. You will need `&&`.

   Inside the loop:
   - take the stage length off `minutesOfLight`
   - add one to `stagesWalked`
   - add 5 to `stageMinutes`
   - print which stage was completed and how much light is left

5. After the loop, print how many stages were walked and how many minutes of light are left.

6. Then use `if` and `else` to say whether the cabin was reached. You reached it if `stagesWalked` is the same as `stagesInRoute`. If not, say how many stages were left.

Work out on paper what you expect before you run it. Then run it and see whether you were right.

### Challenges

**One.** Replace the plain `if` and `else` at the end with a ladder that gives a more useful verdict:

- reached the cabin with more than 60 minutes of light to spare: "Comfortable. You could have taken the lake route."
- reached the cabin at all: "Made it, but only just."
- walked at least half the stages: "Short of the cabin. Camp where you are."
- anything else: "Barely started. Turn back."

Get the order right, and remember that first match wins.

**Two.** Ask the walker how much daylight they have instead of hard coding 300. `prompt` hands back text, not a number, so you will need `Number()` from the rabbit hole in the previous extra lesson.

**Three.** Add a `switch` that reads a single letter for the weather (`'S'` for sun, `'R'` for rain, `'F'` for fog) and adjusts `stageMinutes` before the walk starts. Rain adds 5 minutes to every stage, fog adds 15, sun changes nothing. Give it a `default` for an unrecognised letter, and check your `break` statements.

<details>

<summary>Solution</summary>

```js
let minutesOfLight = 300;
let stageMinutes = 45;
let stagesWalked = 0;

const stagesInRoute = 6;

// Print the plan before setting off.
console.log('Planned route:');

for (let stage = 1; stage <= stagesInRoute; stage++) {
  console.log('Stage ' + stage + ': about ' + (40 + stage * 5) + ' minutes.');
}

console.log('Setting off with ' + minutesOfLight + ' minutes of light.');

// Walk until the light runs out or the route is finished.
while (minutesOfLight >= stageMinutes && stagesWalked < stagesInRoute) {
  minutesOfLight -= stageMinutes;
  stagesWalked++;
  stageMinutes += 5;

  console.log('Stage ' + stagesWalked + ' done. Light left: ' + minutesOfLight);
}

console.log('Walked ' + stagesWalked + ' of ' + stagesInRoute + ' stages.');
console.log('Light remaining: ' + minutesOfLight + ' minutes.');

if (stagesWalked === stagesInRoute) {
  console.log('You reached the cabin.');
} else {
  console.log('Stages short of the cabin: ' + (stagesInRoute - stagesWalked));
}
```

```
Planned route:
Stage 1: about 45 minutes.
Stage 2: about 50 minutes.
Stage 3: about 55 minutes.
Stage 4: about 60 minutes.
Stage 5: about 65 minutes.
Stage 6: about 70 minutes.
Setting off with 300 minutes of light.
Stage 1 done. Light left: 255
Stage 2 done. Light left: 205
Stage 3 done. Light left: 150
Stage 4 done. Light left: 90
Stage 5 done. Light left: 25
Walked 5 of 6 stages.
Light remaining: 25 minutes.
Stages short of the cabin: 1
```

Five stages out of six, with 25 minutes to spare and one stage still to go.

The second half of the loop condition matters. Without `stagesWalked < stagesInRoute` the walker carries on past the cabin as long as there is light, which is not a walk, it is getting lost.

The plan and the walk agree with each other because both build the stage length from the same rule. Stage 1 is 45 minutes in the plan, and the first pass of the while loop takes 45 off the light. Had you typed those numbers separately, they would have drifted apart the first time you changed one.

**Challenge one:**

```js
if (stagesWalked === stagesInRoute && minutesOfLight > 60) {
  console.log('Comfortable. You could have taken the lake route.');
} else if (stagesWalked === stagesInRoute) {
  console.log('Made it, but only just.');
} else if (stagesWalked >= stagesInRoute / 2) {
  console.log('Short of the cabin. Camp where you are.');
} else {
  console.log('Barely started. Turn back.');
}
```

```
Short of the cabin. Camp where you are.
```

The first two rungs are in the only order that works. Both are true for a comfortable arrival, and first match wins, so the more demanding one has to go on top. Swap them and nobody ever gets the first message.

**Challenge two:**

```js
let minutesOfLight = Number(prompt('How many minutes of daylight are left?'));
```

Without `Number()`, `minutesOfLight` is text. `minutesOfLight -= stageMinutes` would still work, because `-` has only one job, but `minutesOfLight >= stageMinutes` would compare text against a number and give you strange answers, and any `+` you added later would join instead of add.

**Challenge three:**

```js
const weather = 'R';

switch (weather) {
  case 'S':
    break;
  case 'R':
    stageMinutes += 5;
    console.log('Rain. Every stage will take 5 minutes longer.');
    break;
  case 'F':
    stageMinutes += 15;
    console.log('Fog. Every stage will take 15 minutes longer.');
    break;
  default:
    console.log('Unknown weather code. Assuming clear.');
}
```

Put this before the while loop, and before the plan printer if you want the plan to reflect it.

The `'S'` case has nothing in it but a `break`. That is deliberate and correct: sunshine changes nothing, and the `break` stops it falling through into the rain. An empty case with a `break` says "this is handled, and the answer is do nothing", which is different from having no case at all, because that would fall to `default` and report sunshine as unknown weather.

</details>

---

## Where this leaves you

A condition is a question with only two possible answers, and giving it a name is the cheapest way to make code readable. `&&`, `||` and `!` join questions together, and when they start piling up, break them into named halves. Module 2 comes back to these three properly.

A ladder works down from the top and stops at the first match, which makes its order part of its logic. `switch` compares one value against exact possibilities using `===`, and every case needs a `break` unless you meant otherwise.

A loop is that same question asked over and over. Write the stop before the work: what has to stay true, which variable is in the question, and where in the body does that variable change. If you cannot point at the line that gets you out, the loop is not finished.

And `for` when you know the count, `while` when you are waiting for something to happen.

That is the whole of Module 1. Everything after this is built from these pieces.
