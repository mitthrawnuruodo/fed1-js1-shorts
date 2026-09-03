# Extra lesson: One value at a time

This is an optional extra lesson. It goes over the same ground as lessons 1.1 and 1.2, but by a different road, with different examples. If the regular lessons made sense, this one will make them stick. If they did not, this one might be the version that lands.

You do not need to create a single file for this lesson. Everything happens in the browser console.

Two sentences carry the whole lesson:

**1. Every line you write either works out a value or gives a value a name.**

**2. What an operator does depends on the type of the value it is given.**

The first one tells you what code is. The second one explains the most common bug beginners hit in their first fortnight.

---

## Part 1: The console is a workbench

Open any web page. Press F12, or Ctrl+Shift+I on Windows and Linux, or Cmd+Option+I on a Mac. Click the Console tab.

You should see a `>` and a blinking cursor. Click next to it and type this, then press Enter:

```js
2 + 2
```

The console answers `4`.

That is worth pausing on. You did not tell it to print anything. You handed it something to work out, and it worked it out and showed you the answer. That is what the console does with every line you give it.

Try a few more. Press Enter after each one:

```js
100 - 7
```

```js
6 * 7
```

```js
'Earl Grey'
```

That last one is not a sum, but it is still a value, and the console still hands it back to you.

### The grey undefined

Now type this one:

```js
console.log(2 + 2)
```

You get `4`, and then underneath it, in grey, the word `undefined`.

Nothing has gone wrong. Two separate things happened on that one line.

`console.log` is an instruction that means "print this where I can see it". It printed `4`. That is the first line you see.

Then the console did what it always does, which is report what the line handed back. `console.log` prints things, but it does not hand anything back. JavaScript has a word for "there is nothing here", and that word is `undefined`. So the console reports `undefined`.

You will see that grey `undefined` constantly. It is not an error. It is the console being honest about a line that printed something but produced nothing.

<details>

<summary>Rabbit hole: then why does anyone bother with console.log?</summary>

Because the automatic echo only happens in the console itself.

When your code lives in a `.js` file and the browser runs it, nobody is standing there watching each line and reporting the result. The values are worked out and then thrown away. If you want to see one, you have to ask, and `console.log` is how you ask.

So in the console, `2 + 2` is enough. In a file, you need `console.log(2 + 2)`.

That is the whole difference. Once you start writing files in the next part of the course, `console.log` becomes your only window into what your code is doing.

</details>

---

## Part 2: Giving a value a name

So far every value has been worked out and immediately forgotten. To keep one, you give it a name.

```js
const waterTemperature = 95
```

Press Enter, and the console shows grey `undefined` again. Same reason as before: that line stored something, but it did not hand anything back.

The value is there, though. Type the name on its own:

```js
waterTemperature
```

The console answers `95`.

You might have met the picture of a variable as a labelled box that you put a value into. Here is a slightly different picture that will serve you better later: **a name is a tag you tie onto a value.** The value exists; the name is a label pointing at it.

That picture makes `const` easy to understand. `const` means the tag stays tied where it is. Try this:

```js
const waterTemperature = 95
waterTemperature = 80
```

The console tells you off:

```
Uncaught TypeError: Assignment to constant variable.
```

You tried to move the tag onto a different value, and `const` would not let you.

When you know the tag will need to move, use `let` instead:

```js
let cupsPoured = 0
cupsPoured = 1
cupsPoured = 2
```

No complaint. The tag moved twice.

The habit worth building: **reach for `const` first, every time.** Only change it to `let` when you find you actually need to move the name. This way, anyone reading your code can tell at a glance which values settle down and which ones wander, and that is genuinely useful information.

### Names are for people

JavaScript does not care what you call things. `const q = 95` runs exactly as well as `const waterTemperature = 95`.

But you will read this code far more often than you write it, and so will anyone you work with. A name is your chance to say what a number means. `95` on its own could be a temperature, a percentage, a price or a score. `waterTemperature` cannot be mistaken for any of those.

Three conventions to follow from the start:

- Start with a lowercase letter, then capitalise each word after the first: `waterTemperature`, `brewSeconds`, `takesMilk`. No spaces.
- Say what it is, not how big it is. `brewSeconds` is better than `bigNumber`.
- Put the unit in the name when there is one. `brewSeconds` tells you not to pass it minutes.

---

## Part 3: Three kinds of value

Every value in this lesson is one of three kinds. JavaScript calls these types.

**Text** is called a string. You write it inside quotes.

```js
const teaName = 'Assam'
```

**Numbers** are just written down, with no quotes. Whole numbers and decimals are both simply numbers in JavaScript, with no distinction between them.

```js
const brewSeconds = 180
const spoonfuls = 1.5
```

**Yes or no answers** are called booleans, and there are only two of them in the entire language: `true` and `false`. No quotes.

```js
const takesMilk = true
```

Let us put those together into something. This is a brewing card, the sort of thing printed on the side of a tea caddy.

```js
const teaName = 'Assam';
const waterTemperature = 95;
const brewSeconds = 180;
const takesMilk = true;

console.log(teaName + ': ' + waterTemperature + ' degrees for ' + brewSeconds + ' seconds. Milk: ' + takesMilk);
```

That prints:

```
Assam: 95 degrees for 180 seconds. Milk: true
```

Notice the blank line in the middle. The first block gathers the facts, the second block does something with them. The browser ignores blank lines completely, so that one is there purely for you. Grouping related lines together and putting a gap between groups is one of the cheapest things you can do to make code readable.

<details>

<summary>Rabbit hole: single quotes, double quotes, and apostrophes</summary>

`'Assam'` and `"Assam"` are the same string. JavaScript does not care which you use, as long as the quote you open with is the quote you close with. Pick one and stay with it, and let Prettier tidy up if you drift.

The one time it matters is when the text itself contains a quote:

```js
const label = 'It's ready'
```

That breaks. JavaScript reads `'It'` as a finished string and then has no idea what `s ready'` is meant to be. You will get a `SyntaxError`.

Two ways out. Wrap it in the other kind of quote:

```js
const label = "It's ready"
```

Or put a backslash in front of the apostrophe, which tells JavaScript that this one is part of the text rather than the end of it:

```js
const label = 'It\'s ready'
```

The first is easier to read. Use it.

</details>

---

## Part 4: One plus sign, two jobs

Here is the second of our two sentences: **what an operator does depends on the type of the value it is given.**

The `+` sign has two completely separate jobs.

Given two numbers, it adds:

```js
console.log(brewSeconds + 30);
```

```
210
```

Given two strings, it joins them end to end. This is called concatenation:

```js
console.log('Brew for ' + 'three minutes');
```

```
Brew for three minutes
```

Both of those are fine. The trouble starts when you mix them.

```js
console.log('Brew for ' + brewSeconds);
```

```
Brew for 180
```

That worked, and it is the behaviour you wanted: JavaScript saw a string on the left, decided this was a joining job, and turned the number into text so it could be joined. Very helpful.

Now the same helpfulness, in a case where it is the last thing you want:

```js
const brewSeconds = '180';

console.log(brewSeconds + 30);
```

```
18030
```

You expected `210`. You got `18030`.

Look closely at the first line. Those quotes around `180` mean it is not the number 180. It is a piece of text that happens to contain digits. So `+` saw a string, decided this was a joining job, turned `30` into `'30'`, and stuck them together.

This is not a rare edge case. It is the single most common bug in your first weeks of JavaScript, and it turns up wherever a number arrives from somewhere else in the world rather than being typed straight into your code. Anything a person types into a box, for instance, arrives as text.

Try it. This will pop up a box; type `180` into it:

```js
const typed = prompt('How many seconds?');

console.log(typed + 30);
```

You typed digits. You still get `18030`. `prompt` hands back text, always, no matter what the person types.

<details>

<summary>Rabbit hole: turning text into a number</summary>

You have seen that `'180' + 30` gives `'18030'` rather than `210`. The `+` looked at a piece of text and did its joining job.

This matters because some things in JavaScript only ever hand you text. `prompt()` is one of them. Even if the person types 180, what you get back is the text `'180'`, not the number 180.

JavaScript has two ways to fix this.

**`Number()`** takes a piece of text and gives back the number it represents.

```js
const typed = '180';

console.log(typed + 30);          // "18030"
console.log(Number(typed) + 30);  // 210
```

`Number()` expects the whole string to be a number. If there is anything else in there, it gives up and returns `NaN`, which stands for Not a Number.

**`parseInt()`** is more forgiving. It reads from the start of the text, takes as many digits as it can, and stops at the first thing that is not part of a whole number.

```js
console.log(Number('12 kg'));    // NaN
console.log(parseInt('12 kg'));  // 12
```

Note the word "Int" in the name. It means integer, a whole number. So `parseInt` throws away anything after the decimal point:

```js
console.log(Number('3.9'));    // 3.9
console.log(parseInt('3.9'));  // 3
```

Here is the same set of strings through both:

| Text | `Number()` | `parseInt()` |
| --- | --- | --- |
| `'180'` | `180` | `180` |
| `'3.9'` | `3.9` | `3` |
| `'12 kg'` | `NaN` | `12` |
| `'kg 12'` | `NaN` | `NaN` |
| `''` | `0` | `NaN` |
| `'ten'` | `NaN` | `NaN` |

Neither of them can read `'kg 12'`, because neither starts at the digits. `parseInt` only looks at the front.

There is a third one, `parseFloat`, which behaves exactly like `parseInt` except that it keeps the decimal part:

```js
console.log(parseInt('3.9 metres'));    // 3
console.log(parseFloat('3.9 metres'));  // 3.9
```

**Which one should you reach for?** Use `Number()` when the whole thing should be a number and anything else is a mistake you want to hear about. Use `parseInt()` when you know there is a unit or some other text stuck on the end, like `'180 cm'`, and you only want the whole number at the front. Use `parseFloat()` in that same situation when the decimals matter.

Module 3 comes back to `parseInt` and `parseFloat` properly, alongside a way of controlling how many decimal places a number displays. Everything here is a first look, and enough to get you through Module 1.

**One last thing about `NaN`.** It means the conversion failed, so it is worth checking for. But it behaves oddly. It is the only value in JavaScript that is not equal to itself:

```js
console.log(NaN === NaN);  // false
```

That means `if (result === NaN)` never works. There is a proper way to test for it, and you will meet it later in the course. For now, just know that a `NaN` appearing in your console means a piece of text could not be read as a number, and the place to look is wherever that text came from.

</details>

### The tool for the job: typeof

When a value is not behaving, the first question to ask is what type it actually is. `typeof` answers that.

```js
console.log(typeof 'Assam');
console.log(typeof 180);
console.log(typeof true);
```

```
string
number
boolean
```

And on the bug from a moment ago:

```js
const brewSeconds = '180';

console.log(typeof brewSeconds);
```

```
string
```

There it is. Not a number, despite looking like one on the page.

Get into the habit: whenever a calculation gives you a nonsensical answer, `console.log(typeof yourVariable)` before you do anything else. It is a five second check that will save you hours.

<details>

<summary>Rabbit hole: why does only `+` have this problem?</summary>

Try this:

```js
console.log('180' * 2);
```

```
360
```

That works. So does `-`, and `/`.

The reason is that those operators only have one job. There is no such thing as subtracting one piece of text from another, so when JavaScript sees `'180' * 2` there is no ambiguity about what you meant. It converts the text to a number and multiplies.

`+` is the only operator with two jobs, and so it is the only one that has to guess which one you wanted. When it sees a string on either side, it guesses "join".

This is a good thing to know and a terrible thing to rely on. Do not write `'180' * 2` on purpose. Convert your value properly, at the point where it arrives, and then everything downstream is a real number and none of this comes up again.

</details>

---

## Part 5: Writing it down for the person who comes next

Everything so far has been about making code work. This part is about making it readable, which is the subject of lesson 1.2.

It is easy to treat readability as good manners, something you do to be polite to other developers. That undersells it. **You will read your own code more often than anyone else will, and you will do it after you have forgotten how it works.** Three weeks is plenty. Readability is a favour to yourself.

### A better name beats a comment

Beginners are often told to comment their code, and they conclude that more comments means better code. They do not. Look at this:

```js
// t is the water temperature
const t = 95;
```

Two lines to say what one line could have said:

```js
const waterTemperature = 95;
```

Before you write a comment explaining what something is, check whether a better name would make the comment unnecessary. Usually it will.

### What comments are actually for

The code already says what is happening. A comment earns its place when it says **why**.

```js
// Assam is a strong black tea, so it takes water just off the boil.
// Green teas would be scalded at this temperature.
const waterTemperature = 95;
```

You could stare at `95` all day and never work that out. That is a comment worth having.

### A stale comment is worse than no comment

Here is the real hazard:

```js
// Brew for a further 30 seconds for the second pot
const secondPotSeconds = brewSeconds + 60;
```

The comment says 30. The code says 60. One of them is wrong, and you have no way of knowing which without going and asking somebody.

An out of date comment is worse than nothing at all, because you trust it. Nothing forces a comment to stay true when the code beneath it changes, so every comment you write is a small promise to keep it updated. Write fewer, better comments, and you have fewer promises to keep.

### Indentation and blank lines

Indentation is not decoration. It shows what is inside what. You do not have anything nested yet, so it will matter more from the next lesson onwards, when `if` statements and loops arrive with their curly braces.

What you can do already is group your lines. Blank lines between groups do for code what paragraphs do for prose:

```js
const teaName = 'Assam';
const waterTemperature = 95;
const brewSeconds = 180;

const secondPotSeconds = brewSeconds + 60;

console.log(teaName + ' second pot: ' + secondPotSeconds + ' seconds');
```

Facts, then working, then output. You can see the shape of it before you have read a word.

And when you get tired of doing this by hand, install Prettier in VS Code and turn on Format On Save. It will handle indentation and spacing for you, permanently. Naming, comments and grouping stay your job, because those need a brain.

---

## Exercises

Try each one before opening the solution.

### Exercise 1: Predict, then run

Write down what you think each of these lines will print. Then type them into the console one at a time and see how you did.

```js
6 + 4
```

```js
'6' + 4
```

```js
6 + '4'
```

```js
'6' + '4'
```

```js
'Assam' + ' ' + 'tea'
```

```js
'Cups: ' + 4
```

```js
typeof '6'
```

```js
typeof (6 + 4)
```

Getting some of these wrong is the point. The ones that surprise you are the ones you have just learnt something from.

<details>

<summary>Solution</summary>

```
6 + 4              10          two numbers, so + adds
'6' + 4            "64"        a string is involved, so + joins
6 + '4'            "64"        the order makes no difference
'6' + '4'          "64"        two strings, joined
'Assam' + ' ' + 'tea'   "Assam tea"    the middle string is a single space
'Cups: ' + 4       "Cups: 4"   the number is turned into text so it can be joined
typeof '6'         "string"    quotes make it text, digits or not
typeof (6 + 4)     "number"    the sum is worked out first, and 10 is a number
```

The one people get wrong most often is `6 + '4'`. It feels as though putting the number first should make it a sum, but it does not. If either side is a string, `+` joins.

</details>

### Exercise 2: Build a brewing card

Make a brewing card for a tea of your choosing.

1. Open the console.
2. Create four variables: the name of the tea as a string, the water temperature as a number, the brewing time in seconds as a number, and whether it takes milk as a boolean.
3. Use `const` for all four.
4. Print one sentence that uses all four, joined with `+`.

Then, as a second step, work out the brewing time for a second pot, which needs 45 seconds longer, and print that as a second sentence. Do not retype the number: build it from the variable you already have.

<details>

<summary>Solution</summary>

```js
const teaName = 'Lapsang Souchong';
const waterTemperature = 95;
const brewSeconds = 240;
const takesMilk = false;

console.log(teaName + ': ' + waterTemperature + ' degrees for ' + brewSeconds + ' seconds. Milk: ' + takesMilk);

const secondPotSeconds = brewSeconds + 45;

console.log('Second pot: ' + secondPotSeconds + ' seconds');
```

```
Lapsang Souchong: 95 degrees for 240 seconds. Milk: false
Second pot: 285 seconds
```

Two things to notice.

The boolean printed as `false`, without quotes in your code but as text on the screen. `+` turned it into text so it could be joined, exactly as it did with the numbers.

And `secondPotSeconds` was built from `brewSeconds` rather than typed as `285`. If you later decide the first pot should brew for 300 seconds, you change one number and the second pot follows. Type `285` by hand and you have two numbers to remember to change, and one day you will change only one of them.

</details>

### Exercise 3: Three broken lines

Each of these three snippets prints something that surprises people. For each one, work out what it prints and why. Two of them need fixing. One does not, and working out which is the whole point of the exercise. Use `typeof` if you get stuck.

```js
const cupsPerPot = '4';
const cupsForTwoPots = cupsPerPot + cupsPerPot;
console.log(cupsForTwoPots);
```

```js
const potMillilitres = 880;
const label = 'Pot size: ' + potMillilitres + 'ml';
const doubleLabel = label + label;
console.log(doubleLabel);
```

```js
const brewMinutes = prompt('How many minutes?');
const brewSeconds = brewMinutes * 60;
const brewSecondsPlusRest = brewSeconds + 30;
console.log(brewSecondsPlusRest);
```

That last one is the interesting one. Type `3` when the box appears.

<details>

<summary>Solution</summary>

**First snippet** prints `44`, not `8`.

`cupsPerPot` is the string `'4'`, because of the quotes. So `+` joined `'4'` to `'4'`. The fix is to take the quotes off:

```js
const cupsPerPot = 4;
const cupsForTwoPots = cupsPerPot + cupsPerPot;
console.log(cupsForTwoPots);  // 8
```

**Second snippet** prints `Pot size: 880mlPot size: 880ml`.

Nothing is broken about the types here. `label` is a string, and joining a string to itself is exactly what `+` should do. The problem is that the person writing it expected some kind of separator. There is no bug to fix, only an expectation to correct: `+` joins strings end to end with nothing in between. If you want a gap, you have to put one there:

```js
const doubleLabel = label + ', ' + label;
```

Worth learning early. Not every wrong answer is a type problem.

**Third snippet** prints `210`. It is the one that does not need fixing.

If you predicted `18030`, you were following the pattern of the first two rather than reading the code, which is a very human mistake and worth catching yourself doing.

Here is what actually happens. `prompt` handed back the string `'3'`, so `brewMinutes` is text. But the next line multiplies, and `*` has only one job, so it converted the text to a number and gave back `180`. A real number. Check it:

```js
console.log(typeof brewSeconds);  // number
```

By the time `+ 30` runs, it is working with two numbers, so it adds them.

So the code works, but only by luck. The moment somebody changes that line from `* 60` to something involving `+`, it breaks, and the reason will be three lines further up in a line that looks perfectly innocent. Convert at the point where the text arrives and stop relying on which operator happens to come next:

```js
const brewMinutes = Number(prompt('How many minutes?'));
```

Now `brewMinutes` is a number from the start and every line after it is safe.

</details>

### Exercise 4: Pseudocode, backwards

Lesson 1.1 had you write pseudocode before writing code. This exercise goes the other way, which is a skill you will use far more often: reading unfamiliar code and working out what it was for.

Here is a working script. Write the pseudocode it came from, using plain English and the conventions from lesson 1.1, such as `SET`, `CALCULATE` and `DISPLAY`.

```js
const potMillilitres = 880;
const cupMillilitres = 220;

const cupsPerPot = potMillilitres / cupMillilitres;

console.log('One pot fills ' + cupsPerPot + ' cups.');
```

<details>

<summary>Solution</summary>

```
START
  SET potMillilitres to 880
  SET cupMillilitres to 220
  CALCULATE cupsPerPot = potMillilitres divided by cupMillilitres
  DISPLAY "One pot fills " + cupsPerPot + " cups."
END
```

Your wording will differ, and that is fine. Pseudocode has no rules to break.

What matters is that you could do it at all, and the reason you could is the naming. Try reading the same script with the names stripped out:

```
const a = 880;
const b = 220;
const c = a / b;
console.log('One pot fills ' + c + ' cups.');
```

Same code, same result, and now you have to reverse engineer the intent from a single line of output. This is what people mean when they say naming is the hard part.

</details>

### Exercise 5: Make the comments unnecessary

Here is a script that works. It also has four comments, and one of them is lying.

```js
// t is the water temperature in degrees
const t = 95;

// s is how long to brew for
const s = 180;

// the second pot needs 30 seconds longer
const x = s + 60;

// print it
console.log(t + ' degrees, ' + x + ' seconds');
```

Three steps:

1. Find the comment that does not match the code. Decide which one you think is right, the comment or the code.
2. Rename `t`, `s` and `x` so that the first two comments and the last one become pointless, then delete them.
3. Replace the remaining comment with one that explains why rather than what. Invent a plausible reason.

<details>

<summary>Solution</summary>

**Step 1.** The third comment says 30 seconds, the code adds 60. There is no way to tell from here which is correct, and that is precisely the problem with stale comments: they destroy your ability to trust the file. In real work you would go and ask whoever wrote it. Here, pick one and make the code and the comment agree.

**Steps 2 and 3:**

```js
const waterTemperature = 95;
const brewSeconds = 180;

// The leaves are already wet from the first pot, so the second
// brew needs longer to reach the same strength.
const secondPotSeconds = brewSeconds + 60;

console.log(waterTemperature + ' degrees, ' + secondPotSeconds + ' seconds');
```

Four comments became one. The three that went were all describing what the next line plainly said, and once the names were doing their job there was nothing left for them to add.

The one that survived is the only one carrying information that is not in the code. You could read `brewSeconds + 60` forever and never deduce why. That is the test to apply: **if the comment tells you something the code cannot, keep it. If it repeats the code, delete it and improve the name instead.**

Note also that the comment no longer states the number. It says the second brew needs longer, not that it needs 60 seconds longer. Now, when someone changes 60 to 75, the comment is still true. Comments that avoid repeating details from the code have a much better chance of surviving.

</details>

---

## Where this leaves you

Two sentences, and a short list of habits that follow from them.

Every line either works out a value or gives a value a name. That is why the console echoes results at you, why `console.log` leaves a grey `undefined` behind, and why `const waterTemperature = 95` is a complete and useful line of code.

What an operator does depends on the type of value it is given. That is why `'6' + 4` is `'64'`, why `typeof` is the first thing to reach for when a number misbehaves, and why text arriving from `prompt` needs converting before you do arithmetic on it.

And the habits: `const` first and `let` only when you need it, names that say what a value means, comments that say why rather than what, and blank lines between groups of related lines.

In the next lesson these values start making decisions.
