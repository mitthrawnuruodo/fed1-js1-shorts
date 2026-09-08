# Events, or how to hand your code to the browser

**Estimated time:** about 2 hours for the core path, plus 60 to 90 minutes for the self study task.  
**You will need:** variables, `if`, `for` and `for...of`, arrays, objects, functions, `forEach`, `map`, `filter` and `find`, plus Lessons 4.1 and 4.2 (selecting elements, changing them, creating them).  
**Covers:** Lesson 4.3 (events and forms) and Lesson 4.4 (best practices), from a different direction.  

## How this lesson is different

The regular lessons introduce events one at a time. Here is `click`, here is `mouseover`, here is `keydown`, here is the event object, here is `submit`. Each gets a small example. That is a perfectly sensible way to meet them, and you should still work through those pages.

This lesson is built on two ideas instead of a list of event names.

**Idea one: you never call your own function. The browser calls it for you.** Almost everything that feels strange about events comes from this one fact, and once you see it clearly, the strangeness goes away. Why is there no `()`? Why does a parameter appear that you never passed in? Why does returning a value do nothing? All the same answer.

**Idea two: a handler changes your data and asks for a redraw. It does not go poking at the page.** In the previous lesson you wrote `render(state)`, changed the data by hand, and called `render` again to see the change. This is the lesson where a user click takes over that job.

The best practices from Lesson 4.4 are not saved up for a section at the end. They are built into how the code is written from the first example: real buttons, no `onclick` in the HTML, classes instead of colours, named functions, and no accidental globals. Part 5 then gives them their proper names.

---

## Before you start

Make one folder with three files. You will keep editing these same three files rather than starting a new folder for every section.

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Events lab</title>
    <link rel="stylesheet" href="style.css" />
    <script src="app.js" defer></script>
  </head>
  <body>
    <h1>Events lab</h1>
  </body>
</html>
```

`app.js`:

```js
'use strict';

console.log('app.js is running');
```

`style.css` can stay empty for now.

Two things in there are worth explaining before we go on.

**`defer` on the script tag.** Without it, a script in the `<head>` runs before the browser has finished building the page, so `document.querySelector('#something')` gives you `null` and your first line of real code crashes. With `defer`, the browser waits until the page is built. If you ever find that your selections return `null` for an element you can plainly see, check this first. Putting the script tag at the very end of the `<body>` does the same job, and you will see that in the regular lessons.

**`'use strict';` on the first line of the JavaScript file.** This turns on a stricter version of the language. Its most useful effect for you right now is that a mistyped variable name becomes a visible error instead of quietly creating a new variable behind your back. It costs one line and it will save you an evening at some point. There is more on it in Part 5.

Open `index.html` in your browser, press F12 to open the developer tools, and keep the Console tab visible for the whole lesson.

---

## Part 1: You are not the one calling your function

Up to now, every function you have written, you have also called. You write `greet()` and the greeting happens, right there, on that line. Programs read top to bottom and you are in charge of when everything runs.

Events break that habit, and it is worth being explicit about it, because a lot of confusion comes from not noticing that the rules have changed.

With an event listener, you write a function and then **hand it over**. You do not call it. You give it to the browser, tell the browser what should make it run, and then your script finishes and goes quiet. Minutes later, when a user clicks something, the browser reaches into the function you left behind and calls it. Possibly many times. Possibly never.

You are not writing instructions any more. You are leaving instructions.

### 1.1 The recipe, not the meal

Let us build the smallest useful thing: a row counter, the sort of tally a knitter clicks once per row.

`index.html`, inside `<body>`:

```html
<p id="row-count">Row 0</p>
<button id="next-row">Next row</button>
<button id="reset-rows">Reset</button>
```

`app.js`:

```js
'use strict';

let rowCount = 0;

function showRowCount() {
  const display = document.querySelector('#row-count');
  display.textContent = 'Row ' + rowCount;
}

function increaseRowCount() {
  rowCount = rowCount + 1;
  showRowCount();
}

const nextRowButton = document.querySelector('#next-row');

nextRowButton.addEventListener('click', increaseRowCount);
```

Look closely at the last line. `increaseRowCount` has **no brackets after it**.

That is not a typo, and it is not a style choice. It is the whole idea.

- `increaseRowCount()` means *run this function now and give me the result*.
- `increaseRowCount` means *the function itself*, as a value you can pass around.

`addEventListener` wants the function itself, because it is going to keep it and call it later. If you write brackets, you run the function immediately and hand `addEventListener` whatever came back out of it.

You met this already with `forEach`, `map` and `filter`. You wrote `numbers.forEach(printItem)` and never `numbers.forEach(printItem())`, for exactly the same reason. An event listener is the same arrangement: a callback, handed to something that will call it for you. The only difference is that `forEach` calls yours immediately, and the browser might call yours in half an hour.

The usual way to put this: **you are handing over the recipe, not the meal.**

### 1.2 What the mistake actually looks like

This matters because the mistake is silent. Try it deliberately:

```js
nextRowButton.addEventListener('click', increaseRowCount());
```

Reload the page and watch what happens:

- The counter jumps to "Row 1" **immediately**, before you have clicked anything.
- Clicking the button then does nothing at all, forever.
- There is no error message. The console is clean.

Here is why. `increaseRowCount()` ran on the spot, which is why the display changed. It has no `return` statement, so it produced `undefined`. So the line the browser really saw was this:

```js
nextRowButton.addEventListener('click', undefined);
```

The browser accepts that quite happily. There is nothing to call, so nothing happens when you click.

"It works once when the page loads and then never again" is the signature of this bug. When you see that pattern, go and look for a stray pair of brackets.

### 1.3 Two ways to hand a function over

You will see both of these constantly, and they do the same thing.

```js
// A named function, written elsewhere, handed over by name
nextRowButton.addEventListener('click', increaseRowCount);

// An unnamed function, written on the spot
nextRowButton.addEventListener('click', function () {
  rowCount = rowCount + 1;
  showRowCount();
});
```

The second one is an anonymous function, exactly like the ones you have been passing to `filter` and `map`. It is convenient for one or two lines.

This lesson will mostly use the first form, and that is a deliberate best practice. A named function can be read on its own, tested on its own, and reused. It also keeps the registration line short enough to take in at a glance. When a listener grows past about three lines, give it a name.

A common naming convention, and the one we will use, is `handleSomething`:

```js
function handleNextRowClick() {
  rowCount = rowCount + 1;
  showRowCount();
}

nextRowButton.addEventListener('click', handleNextRowClick);
```

That line now reads almost like English: when this button is clicked, handle the next row click.

### 1.4 The parameter that appears from nowhere

Change the handler to take a parameter:

```js
function handleNextRowClick(event) {
  console.log(event);
  console.log(event.type);
  rowCount = rowCount + 1;
  showRowCount();
}
```

Click the button, and you will see an object logged, and the word `click`.

You never passed anything in. You wrote `addEventListener('click', handleNextRowClick)` with no arguments anywhere. So where did `event` come from?

From the browser. **The browser is the one calling your function**, and when it does, it passes one argument: an object describing what just happened. Your parameter catches it.

This is not magic. It is the ordinary rule about parameters, seen from the other side. When you write `showTotal(45)`, the value 45 lands in whatever the first parameter is called. Here the browser is doing the calling, and it always supplies exactly one argument.

You have seen this from the other side too. When you write `fruits.forEach(function (fruit) { ... })`, you never fill in `fruit` yourself; `forEach` does it on every turn of its loop. The event object arrives the same way.

Because it is a parameter, **you choose the name**. These are all the same:

```js
function handleNextRowClick(event) { }
function handleNextRowClick(e) { }
function handleNextRowClick(whatHappened) { }
```

`event` and `e` are the two conventional choices. Use whichever you like, but be consistent within a project.

And because it is a parameter, **you can leave it out** if you do not need it. A function that ignores an argument it was given is perfectly legal JavaScript, and you have been doing it all along: `fruits.forEach(function (fruit) { ... })` ignores the index and the array that `forEach` also passes. The counter above never used `event`, and it worked fine.

Two properties of that object are worth knowing today, and we will use both:

```js
console.log(event.type);   // "click"
console.log(event.target); // the element that was clicked
```

The rest of it can wait. It is a big object with a lot in it, and you can always log it and have a look.

### 1.5 Returning a value from a handler is pointless

Try this:

```js
function handleNextRowClick() {
  return rowCount + 1;
}
```

Nothing happens. Not "nothing visible happens", but genuinely nothing: the value goes nowhere.

`return` hands a value back **to whoever called the function**. The browser called this one, and the browser has no use for your number. There is no variable on the other side waiting to catch it.

This is worth sitting with, because it is the clearest illustration of the shift in Part 1. In ordinary code you write functions that work out an answer and give it back. Handlers are not like that. A handler is a function you write **for its effects**: it changes some data, or it changes the page. Nobody is collecting a result.

### 1.6 Listeners stack

`addEventListener` adds. It does not replace. Add two listeners for `click` and both run, in the order you added them.

```js
resetButton.addEventListener('click', saveProgress);
resetButton.addEventListener('click', clearCount);
```

Both run on every click. That is often exactly what you want: two unrelated pieces of code can each listen to the same button without knowing about each other.

It also means that if you register a listener inside a function that runs more than once, you quietly get duplicates, and the handler starts firing two, three, four times per click. This is one of the more baffling bugs to meet, and Part 3 shows a structure that avoids it entirely.

There is one small piece of protection built in. The browser refuses to add **the exact same function** twice for the same event:

```js
function beep() {
  console.log('beep');
}

button.addEventListener('click', beep);
button.addEventListener('click', beep);
// one "beep" per click, not two
```

But that only works because both lines refer to the same named function. Two anonymous functions that happen to contain identical code are still two different functions:

```js
button.addEventListener('click', function () {
  console.log('beep');
});

button.addEventListener('click', function () {
  console.log('beep');
});
// two "beep"s per click
```

Which is one more reason to name your handlers.

### Exercise 1: predict, then run

**Goal:** to be certain about what `addEventListener` does with the function you give it.

**Time:** about 20 minutes.

**Brief**

Set up this HTML:

```html
<p id="row-count">Row 0</p>
<button id="next-row">Next row</button>
```

and this JavaScript, with `rowCount` and `showRowCount` from 1.1 in place:

```js
function increaseRowCount() {
  rowCount = rowCount + 1;
  showRowCount();
}

const nextRowButton = document.querySelector('#next-row');

// Line A
nextRowButton.addEventListener('click', increaseRowCount());

// Line B
nextRowButton.addEventListener('click', increaseRowCount);

// Line C
nextRowButton.addEventListener('click', increaseRowCount);

// Line D
nextRowButton.addEventListener('Click', increaseRowCount);

// Line E
nextRowButton.addEventListener('click', function () {
  increaseRowCount();
});
```

1. Before running anything, write down your answers. On paper or in a comment, not in your head, because once you have seen the answer you will convince yourself you knew it.

   - What does the display show immediately after the page loads, before any click?
   - How much does the counter go up by on the first click? Give a number.
   - Which of the five lines contribute to that number, and which do nothing?
   - Is there anything in the console?

2. Run it and compare.
3. Fix the file so that one click adds exactly one row, keeping only the lines that deserve to stay.
4. Add a Reset button that sets the count back to zero, using a named handler.

**Level 2 process:** add a listener that logs `event.type` and `event.target.id`, then click the button and read the output. Then change the parameter's name from `event` to `clicky` and confirm it still works.

<details>
<summary><strong>Solution 1</strong></summary>

**Predictions.**

The display shows **Row 1** immediately after loading, before any click. That is line A running the function on the spot.

One click adds **2**. The contributing lines are B and E. Taking each in turn:

- **Line A** runs `increaseRowCount` immediately, then registers its return value, which is `undefined`. Nothing is registered. It contributes 0 per click, but it does change the display once at load.
- **Line B** registers the function correctly. Contributes 1.
- **Line C** tries to register the very same function for the very same event, so the browser ignores it. Contributes 0.
- **Line D** registers correctly, but for an event called `Click`. Event names are case sensitive and there is no such event, so this listener will wait forever. Contributes 0. **No error is reported**, which is what makes this one nasty.
- **Line E** registers a brand new anonymous function, which is a different function from the one in line B, so it is added as well. Contributes 1.

The console is empty. Nothing here is an error as far as the browser is concerned.

**The fixed file:**

```js
'use strict';

let rowCount = 0;

function showRowCount() {
  const display = document.querySelector('#row-count');
  display.textContent = 'Row ' + rowCount;
}

function handleNextRowClick() {
  rowCount = rowCount + 1;
  showRowCount();
}

function handleResetClick() {
  rowCount = 0;
  showRowCount();
}

document.querySelector('#next-row').addEventListener('click', handleNextRowClick);
document.querySelector('#reset-rows').addEventListener('click', handleResetClick);

// Draw the display once at startup, so the page and the variable agree from the start
showRowCount();
```

That last line is a small habit worth picking up now. If `rowCount` starts at 0 and the HTML says "Row 0", they agree by luck. Draw the display from the variable at startup and they agree by design.

</details>

---

## Part 2: One listener instead of twenty

One button is easy. Lists are where event code either stays simple or turns into a mess, and the difference is one technique.

Here is the page we will work with for the rest of the lesson: a Norwegian phrasebook for visitors. Each phrase can be revealed and can be starred.

```html
<ul id="phrasebook">
  <li class="phrase" data-id="p1">
    <p class="norwegian">Kan du hjelpe meg?</p>
    <p class="english">Can you help me?</p>
    <button data-action="reveal"><span class="label">Show</span></button>
    <button data-action="star">Star</button>
  </li>
  <li class="phrase" data-id="p2">
    <p class="norwegian">Hvor er stasjonen?</p>
    <p class="english">Where is the station?</p>
    <button data-action="reveal"><span class="label">Show</span></button>
    <button data-action="star">Star</button>
  </li>
</ul>
```

Those `data-id` and `data-action` attributes are ordinary HTML attributes with a name of your own choosing, as long as it begins with `data-`. They are how you attach a small piece of your own information to an element, and you read them with `getAttribute` exactly like `src` or `href`. We will lean on them shortly.

The obvious approach is to select all the buttons and give each one a listener:

```js
const buttons = document.querySelectorAll('#phrasebook button');

for (const button of buttons) {
  button.addEventListener('click', handleButtonClick);
}
```

This works. It is also the approach the regular lessons use, and there is nothing wrong with it for a fixed list. But it has two problems that appear the moment your page becomes interesting.

**Problem one: it only covers the buttons that existed when that loop ran.** Add a new phrase later with `createElement`, and its buttons are dead. No listener was ever attached to them, because the loop finished long ago. This is the single most common "why does the new one not work" question in front-end development.

**Problem two: forty phrases means eighty listeners**, all of them the same function, all needing to be attached again every time you rebuild the list. And rebuilding the list is exactly what Part 3 is going to do.

There is a better way, and it depends on understanding what actually happens when you click.

### 2.1 A click hits more than one element

When you click that `<span class="label">Show</span>`, you did not just click a span. You also clicked the button it sits in, the `<li>` around that, the `<ul>` around that, the `<body>`, and the document.

The browser agrees with you. It fires the click event on the innermost element first, then on its parent, then on that element's parent, and so on outwards. This is called **bubbling**, because the event rises up through the tree like a bubble in water.

You can watch it happen. Add these four listeners and click the button:

```js
document.querySelector('.label').addEventListener('click', function () {
  console.log('the span');
});

document.querySelector('#phrasebook button').addEventListener('click', function () {
  console.log('the button');
});

document.querySelector('#phrasebook').addEventListener('click', function () {
  console.log('the list');
});

document.body.addEventListener('click', function () {
  console.log('the body');
});
```

One click, four logs, in this order:

```text
the span
the button
the list
the body
```

Inside first, then outwards. That is the whole rule.

### 2.2 `event.target` is where the click landed

Bubbling means one event reaches several listeners, so the event object has to record where it started. That is `event.target`, and it is always the deepest element, no matter which listener is reading it.

Put a listener on the whole list:

```js
document.querySelector('#phrasebook').addEventListener('click', function (event) {
  console.log(event.target);
});
```

Click the word "Show" and you get the `<span>`, not the button, and not the list.

Note the trap hiding in there. Even if you put the listener directly on the button, `event.target` would still be the span, because that is what the user's finger actually landed on. A button containing an icon or a `<span>` will hand you the icon or the span. Code that reads `event.target.getAttribute('data-action')` breaks the moment somebody puts an icon inside a button, and it breaks in a way that looks completely random, because it depends on which pixel was clicked.

We are about to fix that properly.

### 2.3 Event delegation

Since every click on a phrase bubbles up to the list, **one listener on the list can handle all of them**. This is called event delegation: you delegate the job to a shared ancestor.

```js
function handlePhrasebookClick(event) {
  // 1. From wherever the click landed, walk up and find a button.
  const button = event.target.closest('button');

  // 2. If there is no button above it, the user clicked something else. Leave.
  if (!button) {
    return;
  }

  // 3. Now find which phrase this button belongs to.
  const item = button.closest('.phrase');

  console.log('action:', button.getAttribute('data-action'));
  console.log('phrase:', item.getAttribute('data-id'));
}

document.querySelector('#phrasebook').addEventListener('click', handlePhrasebookClick);
```

Three moving parts, and each one earns its place.

**`closest()` solves the span problem.** You met it in the previous lesson: it starts at an element and walks upwards through its ancestors, returning the first one that matches, or `null`. `event.target.closest('button')` means "the button I clicked, wherever inside it I happened to land". If the user clicked the span, it finds the button. If the user clicked the button itself, `closest` returns that button, because it also checks the element you started from.

**The guard clause is not optional.** Your listener is on the whole list, so it receives every click on the list, including clicks on the phrase text and on the empty space between items. Those have no button above them, so `closest` returns `null`. Without `if (!button) return;` the next line crashes with `Cannot read properties of null`.

That two-line guard, "if this is not for me, leave immediately", is one of the most useful shapes in all of programming. Get used to writing it.

**The `data-` attributes say what the button means.** Rather than working out what to do from the button's text, which changes, or from its position, which also changes, each button carries its own label:

```html
<button data-action="reveal">Show</button>
```

and the handler reads it with `button.getAttribute('data-action')`. You can then branch:

```js
if (button.getAttribute('data-action') === 'reveal') {
  // ...
}

if (button.getAttribute('data-action') === 'star') {
  // ...
}
```

The pay-off is worth stating plainly. **One listener. It works for phrases that do not exist yet.** Add a new `<li>` to that list at any point, with any number of buttons, and it works immediately, because the listener is not on the buttons. It is on the list, and the list was there all along.

### 2.4 Best practice: use a real button

While we are here, a rule that will save you and your users a lot of grief.

If something is clickable, make it a `<button>`. Not a `<div>` with a click listener, not a `<span>`, not an `<a href="#">`.

A real `<button>` gives you, for free and without a single line of JavaScript:

- keyboard focus, so a user can reach it with the Tab key
- activation with both Enter and the space bar
- the correct announcement in a screen reader ("Show, button")
- a `disabled` state that actually stops clicks
- the browser's own focus outline, which many users rely on

A `<div>` with a click listener has none of that. It is invisible to keyboard users, which means the feature is invisible to them too. Rebuilding all of it by hand takes several extra attributes, keyboard listeners for two different keys, and focus styles, and people still get it wrong.

Use `<button type="button">` inside forms, so it does not accidentally submit them. Outside a form, a plain `<button>` is fine.

### Exercise 2: the tool shed board

**Goal:** to handle a whole list of buttons with a single listener.

**Time:** about 25 minutes.

**Brief**

A village allotment has a shed with shared tools. Set up this HTML:

```html
<ul id="tool-board">
  <li class="tool" data-id="t1">
    <span class="tool-name">Wheelbarrow</span>
    <button data-action="take"><span class="label">Take out</span></button>
    <button data-action="report">Report a fault</button>
  </li>
  <li class="tool" data-id="t2">
    <span class="tool-name">Hedge trimmer</span>
    <button data-action="take"><span class="label">Take out</span></button>
    <button data-action="report">Report a fault</button>
  </li>
  <li class="tool" data-id="t3">
    <span class="tool-name">Long ladder</span>
    <button data-action="take"><span class="label">Take out</span></button>
    <button data-action="report">Report a fault</button>
  </li>
</ul>
<p id="shed-log"></p>
```

1. Attach **exactly one** listener, to `#tool-board`. Your file must contain one `addEventListener` call for the board and no loop over the buttons.
2. When a button is clicked, write a line into `#shed-log` saying which action was chosen and which tool it was, using the tool's name from the page, for example: `Wheelbarrow: taken out.`
3. Clicking the tool name, or the gap between items, must do nothing at all and must not produce an error.
4. Test the span problem on purpose. Click exactly on the words "Take out", which are inside a span, and confirm you still get the right answer.
5. Now prove the point. Run the following in the console, then click the new button. It should work without you touching your JavaScript.

```js
const item = document.createElement('li');
item.classList.add('tool');
item.setAttribute('data-id', 't4');

const name = document.createElement('span');
name.classList.add('tool-name');
name.textContent = 'Rotavator';

const button = document.createElement('button');
button.setAttribute('data-action', 'take');
button.textContent = 'Take out';

item.appendChild(name);
item.appendChild(button);
document.querySelector('#tool-board').appendChild(item);
```

**Level 2 process:** add a third button per tool with `data-action="details"`, but write no new listener and no new registration. How many lines did you have to change to support a new action?

<details>
<summary><strong>Solution 2</strong></summary>

```js
'use strict';

function handleToolBoardClick(event) {
  // Walk up from wherever the click landed to the button, if there is one
  const button = event.target.closest('button');
  if (!button) {
    return;
  }

  // Then up again to the list item, so we know which tool it was
  const item = button.closest('.tool');
  if (!item) {
    return;
  }

  const toolName = item.querySelector('.tool-name').textContent;
  const action = button.getAttribute('data-action');
  const log = document.querySelector('#shed-log');

  if (action === 'take') {
    log.textContent = toolName + ': taken out.';
  }

  if (action === 'report') {
    log.textContent = toolName + ': fault reported.';
  }

  if (action === 'details') {
    log.textContent = toolName + ': details requested.';
  }
}

document.querySelector('#tool-board').addEventListener('click', handleToolBoardClick);
```

Points to notice:

- `event.target.closest('button')` gives the right answer whether the user clicked the span or the button, so step 4 needs no special handling.
- The second guard, on `item`, is not strictly needed with this markup, but it costs one line and it means the function cannot crash if somebody later puts a button elsewhere on the board.
- Reading the action into a variable once, rather than calling `getAttribute` in every `if`, is a small readability win. It also gives you one obvious thing to log when a button does nothing.
- The Rotavator works because the listener is on the board, not on any button. The board existed when the listener was attached, and it still exists now.

**Level 2 answer.** Supporting a new action took **one `if` block in the JavaScript and one attribute in the HTML**. No new listener, no new registration, nothing to remember to attach. That ratio is why delegation is worth learning early.

</details>

---

## Part 3: A handler changes data, render draws the page

This is the part that matters most.

You now know how to catch a click. The question is what the handler should actually do, and there is a good answer and a bad one.

### 3.1 The way that stops working

Here is the shape almost everyone writes first. It is not a straw man; it is what the phrasebook looks like if you write it the obvious way.

```js
function handleRevealClick(event) {
  const button = event.target.closest('button');
  const item = button.closest('.phrase');
  const english = item.querySelector('.english');

  if (button.textContent === 'Show') {
    english.classList.remove('is-hidden');
    button.textContent = 'Hide';
  } else {
    english.classList.add('is-hidden');
    button.textContent = 'Show';
  }
}
```

It works. Click it a few times and it does the right thing. But look at what it depends on.

**The truth about this phrase is stored in the page, in English prose.** Whether the translation is showing is recorded as the word "Show" or "Hide" inside a button. Change that button's text to "Reveal" and the logic silently inverts. Translate the page into Norwegian and it breaks. Put an icon inside the button and `textContent` picks up the icon too, and it breaks again.

**Now add a second feature.** Say you want a "Hide all" button, and a counter showing how many are revealed. The counter has to work out its number by reading the page and counting buttons that say "Hide". "Hide all" has to loop over every item, set each class, and fix each button's text. Each new feature has to know about every existing feature's details. Three features means six ways for them to disagree, and eventually one of them forgets to update the counter and you have a page that contradicts itself.

The problem is not that the code is untidy. It is that **there is no single place where the truth lives**.

### 3.2 The loop

Put the truth in ordinary JavaScript, and the page becomes a picture of it.

```text
                 the user does something
                           |
                           v
     the handler changes the data (and only the data)
                           |
                           v
        render(state) draws the whole page from the data
```

That is the entire pattern. In code, a handler looks like this:

```js
function handleSomething(event) {
  // work out what the user meant
  // change state
  render(state);
}
```

Three rules make it work:

1. **Handlers change `state`, and then call `render`.** They do not set text, or classes, or styles.
2. **`render` reads `state` and draws.** It never reads anything back out of the page.
3. **Anything you need to know is in `state`.** If you find yourself asking the page a question, that thing belonged in `state`.

You wrote `render(state)` in the previous lesson and called it by hand from the bottom of the file. Nothing about it changes here. The only difference is what causes it to be called.

### 3.3 The phrasebook, done properly

`index.html`:

```html
<main>
  <h1>Norwegian phrasebook</h1>

  <div id="controls">
    <button data-filter="all">All phrases</button>
    <button data-filter="starred">Starred only</button>
  </div>

  <p id="summary"></p>
  <ul id="phrasebook"></ul>
</main>
```

The list is empty. Everything in it will be generated.

`style.css`:

```css
body { font-family: system-ui, sans-serif; margin: 2rem; max-width: 36rem; }

#phrasebook { list-style: none; padding: 0; }

.phrase {
  border-left: 4px solid #ddd;
  padding: 0.5rem 1rem;
  margin-bottom: 0.75rem;
}

.norwegian { font-weight: 600; margin: 0; }
.english { margin: 0.25rem 0 0.5rem; color: #555; }
.english.is-hidden { color: #aaa; font-style: italic; }

.is-starred { border-left-color: #b45309; background: #fffbeb; }
.empty { color: #666; font-style: italic; }
```

`app.js`, in four pieces.

**Piece one: the state.** All of it, in one place, at the top.

```js
'use strict';

const state = {
  filter: 'all',
  phrases: [
    { id: 'p1', norwegian: 'Kan du hjelpe meg?',  english: 'Can you help me?',      revealed: false, starred: false },
    { id: 'p2', norwegian: 'Hvor er stasjonen?',  english: 'Where is the station?', revealed: false, starred: true  },
    { id: 'p3', norwegian: 'Hva koster det?',     english: 'What does it cost?',    revealed: false, starred: false },
    { id: 'p4', norwegian: 'Snakker du engelsk?', english: 'Do you speak English?', revealed: false, starred: false }
  ]
};
```

Read that and you know everything that is true about the page. That is the test of a good `state`.

**Piece two: small helpers that know nothing about the page.**

```js
function getVisiblePhrases(state) {
  if (state.filter === 'starred') {
    return state.phrases.filter(function (phrase) {
      return phrase.starred;
    });
  }

  return state.phrases;
}

function findPhrase(state, id) {
  return state.phrases.find(function (phrase) {
    return phrase.id === id;
  });
}
```

Neither one mentions `document`. They are plain array work of the kind you did in Module 3, and you can test them in the console without a page in front of you.

**Piece three: one function that turns one phrase into one element.**

```js
function createPhraseItem(phrase) {
  const item = document.createElement('li');
  item.classList.add('phrase');
  item.setAttribute('data-id', phrase.id);

  const norwegian = document.createElement('p');
  norwegian.classList.add('norwegian');
  norwegian.textContent = phrase.norwegian;

  const english = document.createElement('p');
  english.classList.add('english');

  const revealButton = document.createElement('button');
  revealButton.setAttribute('data-action', 'reveal');

  // The button's text and the paragraph both come from the same true/false value
  if (phrase.revealed) {
    english.textContent = phrase.english;
    revealButton.textContent = 'Hide';
  } else {
    english.textContent = 'Translation hidden';
    english.classList.add('is-hidden');
    revealButton.textContent = 'Show';
  }

  const starButton = document.createElement('button');
  starButton.setAttribute('data-action', 'star');

  if (phrase.starred) {
    starButton.textContent = 'Starred';
    starButton.setAttribute('aria-pressed', 'true');
    item.classList.add('is-starred');
  } else {
    starButton.textContent = 'Star';
    starButton.setAttribute('aria-pressed', 'false');
  }

  item.appendChild(norwegian);
  item.appendChild(english);
  item.appendChild(revealButton);
  item.appendChild(starButton);

  return item;
}
```

One phrase in, one element out. Nothing is added to the page here, so you can call it in the console and inspect the result before you trust it.

Notice which way round the dependency runs: the button's text is worked out **from** `phrase.revealed`, never the other way round. And notice that the JavaScript never sets a colour. It adds a class, and the stylesheet decides what that looks like. That is the separation of concerns from Lesson 4.4 doing real work rather than being a slogan.

The `aria-pressed` attribute tells a screen reader that this button is a switch and whether it is currently on. It is one `setAttribute` call and it is the difference between a usable page and a confusing one.

**Piece four: render.**

```js
function render(state) {
  const list = document.querySelector('#phrasebook');
  const summary = document.querySelector('#summary');

  if (!list || !summary) {
    console.log('Phrasebook markup is missing, nothing rendered.');
    return;
  }

  // Empty the list, then build it again from the data.
  // This is our own markup, not anything a user typed, so innerHTML is safe here.
  list.innerHTML = '';

  const visible = getVisiblePhrases(state);

  if (visible.length === 0) {
    const empty = document.createElement('li');
    empty.classList.add('empty');
    empty.textContent = 'No starred phrases yet.';
    list.appendChild(empty);
    summary.textContent = 'Nothing to show.';
    return;
  }

  for (const phrase of visible) {
    list.appendChild(createPhraseItem(phrase));
  }

  const starred = state.phrases.filter(function (phrase) {
    return phrase.starred;
  });

  summary.textContent = visible.length + ' phrases shown, ' + starred.length + ' starred in total.';
}
```

**And now the handlers, which are the short bit.**

```js
function handlePhrasebookClick(event) {
  const button = event.target.closest('button');
  if (!button) {
    return;
  }

  const item = button.closest('.phrase');
  if (!item) {
    return;
  }

  const phrase = findPhrase(state, item.getAttribute('data-id'));
  const action = button.getAttribute('data-action');

  if (action === 'reveal') {
    phrase.revealed = !phrase.revealed;
  }

  if (action === 'star') {
    phrase.starred = !phrase.starred;
  }

  render(state);
}

function handleControlsClick(event) {
  const button = event.target.closest('button');
  if (!button) {
    return;
  }

  state.filter = button.getAttribute('data-filter');
  render(state);
}

document.querySelector('#phrasebook').addEventListener('click', handlePhrasebookClick);
document.querySelector('#controls').addEventListener('click', handleControlsClick);

render(state);
```

Look at what the handlers do and do not do. They flip a boolean or set a string, and then ask for a redraw. Not one line of them touches text, classes or styles. If the display is ever wrong, the bug is in `render` or in `state`, and you can find out which by typing `state` into the console and reading it.

`phrase.revealed = !phrase.revealed;` is the standard way to flip a true or false value, using the NOT operator from Module 2. It reads as "revealed becomes not-revealed".

The two `if` blocks are deliberately not an `if/else`. A click is either a reveal or a star, and writing them as two independent checks means adding a third action later is one more block rather than a rethink.

### 3.4 Why delegation and rendering fit together so well

There is something slightly alarming happening in that code, and it is worth naming.

When you click "Show", `render` runs, `list.innerHTML = ''` throws away every list item, and the loop builds new ones. **The button you clicked no longer exists.** It was destroyed by the very click you made on it.

If you had attached a listener to each button, that would be a disaster. All those listeners would go into the bin along with their buttons, and you would have to attach them all again at the end of every render, every time, without forgetting. That is where the duplicate-listener bug from 1.6 usually comes from: people attach again without clearing up first.

With delegation, none of this arises. The listener is on `#phrasebook`, which `render` never replaces; only its children are replaced. Attach once at startup, and it keeps working for every element you ever build. Click "Show", then click "Hide" on the rebuilt button, and it responds exactly as you would hope.

Two techniques, learnt separately, that turn out to need each other.

### Exercise 3: the tool shed board, with data

**Goal:** to move a working feature from "poking the page" to "changing data and redrawing".

**Time:** about 45 minutes.

**Brief**

Continue with the tool shed from Exercise 2, but throw away the HTML list. Your `index.html` should now contain only:

```html
<main>
  <h1>Tool shed</h1>
  <div id="controls">
    <button data-filter="all">All tools</button>
    <button data-filter="available">Available only</button>
  </div>
  <p id="shed-summary"></p>
  <ul id="tool-board"></ul>
</main>
```

Start from this state:

```js
const state = {
  filter: 'all',
  tools: [
    { id: 't1', name: 'Wheelbarrow',   outWith: null,    broken: false },
    { id: 't2', name: 'Hedge trimmer', outWith: 'Marit', broken: false },
    { id: 't3', name: 'Long ladder',   outWith: null,    broken: true  },
    { id: 't4', name: 'Rotavator',     outWith: null,    broken: false }
  ]
};
```

`outWith` holds the name of whoever has the tool, or `null` if it is in the shed.

**Level 1 process**

1. Write `createToolItem(tool)` returning one `<li>` with the class `tool`, a `data-id` attribute, the tool's name, a line saying either "In the shed" or "Out with Marit", and two buttons carrying `data-action="take"` and `data-action="fault"`.
2. The take button should read "Take out" when the tool is in, and "Bring back" when it is out. Give the item the class `is-out` when it is out and `is-broken` when it is broken, and do all the colouring in CSS.
3. Write `render(state)` that empties the board, filters by `state.filter` (where `available` means not out and not broken), builds one item per visible tool, handles the empty case, and writes a summary such as `4 tools, 1 out, 1 needing repair.`
4. Write one delegated handler for the board. Taking out a tool sets `outWith` to `'You'`; bringing it back sets `outWith` to `null`; the fault button flips `broken`.
5. Write one delegated handler for the controls that sets `state.filter`.
6. Call `render(state)` once at the bottom.

**Rules:** your handlers must not contain the words `textContent`, `classList` or `style`. If you need one of those, the line belongs in `createToolItem` instead. `render` must not read anything out of the page.

**Level 2 process:** a broken tool should not be takeable. Add that rule, and decide where it goes: in the handler, in `createToolItem`, or both. Write one sentence on why.

<details>
<summary><strong>Solution 3</strong></summary>

```js
'use strict';

const state = {
  filter: 'all',
  tools: [
    { id: 't1', name: 'Wheelbarrow',   outWith: null,    broken: false },
    { id: 't2', name: 'Hedge trimmer', outWith: 'Marit', broken: false },
    { id: 't3', name: 'Long ladder',   outWith: null,    broken: true  },
    { id: 't4', name: 'Rotavator',     outWith: null,    broken: false }
  ]
};

// --- helpers that know nothing about the page ---

function isAvailable(tool) {
  return tool.outWith === null && tool.broken === false;
}

function getVisibleTools(state) {
  if (state.filter === 'available') {
    return state.tools.filter(isAvailable);
  }

  return state.tools;
}

function findTool(state, id) {
  return state.tools.find(function (tool) {
    return tool.id === id;
  });
}

// --- one tool in, one element out ---

function createToolItem(tool) {
  const item = document.createElement('li');
  item.classList.add('tool');
  item.setAttribute('data-id', tool.id);

  const name = document.createElement('span');
  name.classList.add('tool-name');
  name.textContent = tool.name;

  const status = document.createElement('p');
  status.classList.add('status');

  const takeButton = document.createElement('button');
  takeButton.setAttribute('data-action', 'take');

  if (tool.outWith === null) {
    status.textContent = 'In the shed';
    takeButton.textContent = 'Take out';
  } else {
    status.textContent = 'Out with ' + tool.outWith;
    takeButton.textContent = 'Bring back';
    item.classList.add('is-out');
  }

  // A broken tool sitting in the shed cannot be taken out at all
  if (tool.broken && tool.outWith === null) {
    takeButton.setAttribute('disabled', 'disabled');
  }

  const faultButton = document.createElement('button');
  faultButton.setAttribute('data-action', 'fault');

  if (tool.broken) {
    faultButton.textContent = 'Mark as repaired';
    item.classList.add('is-broken');
  } else {
    faultButton.textContent = 'Report a fault';
  }

  item.appendChild(name);
  item.appendChild(status);
  item.appendChild(takeButton);
  item.appendChild(faultButton);

  return item;
}

// --- render ---

function render(state) {
  const board = document.querySelector('#tool-board');
  const summary = document.querySelector('#shed-summary');

  if (!board || !summary) {
    console.log('Tool board markup is missing, nothing rendered.');
    return;
  }

  board.innerHTML = '';

  const visible = getVisibleTools(state);

  if (visible.length === 0) {
    const empty = document.createElement('li');
    empty.classList.add('empty');
    empty.textContent = 'No tools match that filter.';
    board.appendChild(empty);
    summary.textContent = 'Nothing to show.';
    return;
  }

  for (const tool of visible) {
    board.appendChild(createToolItem(tool));
  }

  const out = state.tools.filter(function (tool) {
    return tool.outWith !== null;
  });

  const broken = state.tools.filter(function (tool) {
    return tool.broken;
  });

  summary.textContent = state.tools.length + ' tools, ' + out.length + ' out, '
    + broken.length + ' needing repair.';
}

// --- handlers: change the data, then redraw ---

function handleBoardClick(event) {
  const button = event.target.closest('button');
  if (!button) {
    return;
  }

  const item = button.closest('.tool');
  if (!item) {
    return;
  }

  const tool = findTool(state, item.getAttribute('data-id'));
  const action = button.getAttribute('data-action');

  if (action === 'take') {
    if (tool.broken && tool.outWith === null) {
      return;
    }

    if (tool.outWith === null) {
      tool.outWith = 'You';
    } else {
      tool.outWith = null;
    }
  }

  if (action === 'fault') {
    tool.broken = !tool.broken;
  }

  render(state);
}

function handleControlsClick(event) {
  const button = event.target.closest('button');
  if (!button) {
    return;
  }

  state.filter = button.getAttribute('data-filter');
  render(state);
}

document.querySelector('#tool-board').addEventListener('click', handleBoardClick);
document.querySelector('#controls').addEventListener('click', handleControlsClick);

render(state);
```

**Level 2 answer: both, and that is correct rather than sloppy.** `createToolItem` sets the `disabled` attribute, which is the honest thing to show the user: a button that cannot be pressed should look and behave like one, and a disabled button does not fire click events at all, so the browser enforces the rule for you. The check in the handler is a safety net for the case where the state changes some other way, or where somebody later removes the `disabled` line. Showing a rule in the interface and enforcing it in the logic are two different jobs, and doing only the first is how bad data gets into real systems.

</details>

---

## Part 4: Forms are just events with better manners

A form is the oldest interactive thing on the web, and the browser already knows a great deal about how one should behave. Most form bugs come from fighting that rather than using it.

We will build a repair booking form for a bike shop.

```html
<form id="booking-form" novalidate>
  <div class="field">
    <label for="bike">Bike make and model</label>
    <input type="text" id="bike" name="bike" />
    <p class="error" id="bike-error"></p>
  </div>

  <div class="field">
    <label for="day">Drop-off day</label>
    <select id="day" name="day">
      <option value="">Choose a day</option>
      <option value="monday">Monday</option>
      <option value="thursday">Thursday</option>
      <option value="saturday">Saturday</option>
    </select>
    <p class="error" id="day-error"></p>
  </div>

  <div class="field">
    <label for="notes">Anything we should know?</label>
    <textarea id="notes" name="notes" maxlength="100"></textarea>
    <p id="notes-count"></p>
  </div>

  <div class="field">
    <input type="checkbox" id="lock-removed" name="lock-removed" />
    <label for="lock-removed">I have removed my lock</label>
    <p class="error" id="lock-removed-error"></p>
  </div>

  <button type="submit">Book the repair</button>
</form>

<p id="confirmation"></p>
```

Two details in that markup before we write any JavaScript.

**Every input has a `<label for="...">` pointing at its `id`.** This is not decoration. It makes the label clickable, which enlarges the target area, and it is what a screen reader announces when the user reaches the field. A form with unlabelled inputs is unusable for some people. It costs nothing to get right.

**`novalidate` on the form** switches off the browser's own error messages, so that we can write our own and watch them working. In a real project you would normally keep the browser's built-in checks as a first line of defence and add your own on top. For learning, having exactly one thing running is clearer.

### 4.1 Listen for `submit` on the form, not `click` on the button

This is the single most important line in the section.

```js
// Do this
document.querySelector('#booking-form').addEventListener('submit', handleBookingSubmit);

// Not this
document.querySelector('button[type="submit"]').addEventListener('click', handleBookingSubmit);
```

They look equivalent. They are not.

A form can be submitted in several ways: clicking the submit button, pressing Enter in a text field, or pressing Enter or space on the focused button. All of them fire `submit` on the form. Only one of them fires `click` on the button.

If you listen for the click, then a user who fills in the fields and presses Enter, as most people do, gets either nothing at all or a page reload that throws their work away. You will not notice, because you will test it by clicking.

Listen to the form. The browser has already worked out what counts as submitting.

### 4.2 `preventDefault`, and what "default" means

Some events have a built-in browser behaviour attached to them, which happens in addition to your listener. The technical name is the default action.

- `submit` on a form: package up the values and send them to a server, then load the response, which for a form with no `action` means reloading the current page.
- `click` on a link: navigate to the other page.
- `click` on a checkbox: tick or untick it.

That form reload is why you sometimes see the page flash and reset the instant you press the button. Nothing is broken; the browser is doing its oldest and most reliable job. We just do not want it here, because there is no server and we are handling everything in JavaScript.

```js
function handleBookingSubmit(event) {
  event.preventDefault();
  // now the page will not reload, and the rest of the function can run
}
```

`preventDefault` cancels the browser's built-in behaviour for this one event. It does **not** stop your other listeners running, and it does not stop the event bubbling. It only says: do not do the thing you were going to do on your own.

Put it as the first line of a submit handler, every time, and get into the habit of asking "what would the browser do here by itself?" whenever a click seems to undo your work.

### 4.3 Reading what the user typed

Every form control has a `value` property, with one important exception.

```js
const bike = document.querySelector('#bike').value;               // text input
const day = document.querySelector('#day').value;                 // select: the chosen option's value
const notes = document.querySelector('#notes').value;             // textarea
const lockRemoved = document.querySelector('#lock-removed').checked;  // checkbox: true or false
```

A checkbox has a `value` too, but it is almost never what you want. `checked` is the true or false you are after.

Three things that catch people out:

**`value` is always a string.** Always. Even for `<input type="number">`.

```js
const quantity = document.querySelector('#quantity').value;  // "3", not 3
console.log(quantity + 1);          // "31"
console.log(Number(quantity) + 1);  // 4
```

If a total on your page has ever come out as `05` or `31`, this is why.

**An empty field is an empty string, not `null`.** So the check for "did they fill this in" is `=== ''`.

**Whitespace counts.** A field containing three spaces is not empty. `.trim()` from Module 3 removes whitespace from both ends, and you should call it on any text the user typed before you check or store it.

```js
const bike = document.querySelector('#bike').value.trim();
```

For the select, note that the placeholder option has `value=""` on purpose. That makes "they did not choose anything" the same check as an empty text field.

### 4.4 Validation is a question about data

Here is the tempting way to validate, and it is the way the regular lesson shows:

```js
if (bike === '') {
  document.querySelector('#bike-error').textContent = 'Please tell us which bike it is.';
  isValid = false;
}

if (day === '') {
  document.querySelector('#day-error').textContent = 'Please choose a day.';
  isValid = false;
}
```

It works, and for two fields it is perfectly readable. But it tangles two separate jobs together: deciding whether the data is acceptable, and putting messages on the page. Follow the same instinct as Part 3 and split them.

**Job one: read the form into a plain object.**

```js
function readBooking() {
  return {
    bike: document.querySelector('#bike').value.trim(),
    day: document.querySelector('#day').value,
    notes: document.querySelector('#notes').value.trim(),
    lockRemoved: document.querySelector('#lock-removed').checked
  };
}
```

**Job two: decide whether that object is acceptable.** This function takes data and returns data. It never touches the page.

```js
function validateBooking(booking) {
  const errors = [];

  if (booking.bike === '') {
    errors.push({ field: 'bike', message: 'Please tell us which bike it is.' });
  } else if (booking.bike.length < 3) {
    errors.push({ field: 'bike', message: 'That looks too short to be a bike name.' });
  }

  if (booking.day === '') {
    errors.push({ field: 'day', message: 'Please choose a drop-off day.' });
  }

  if (booking.lockRemoved === false) {
    errors.push({ field: 'lock-removed', message: 'We cannot accept a bike with a lock on it.' });
  }

  return errors;
}
```

An array of problems, built with `push`, exactly as you built arrays in Module 2. An empty array means the booking is fine. That is why there is no `isValid` flag to keep in step with anything: `errors.length === 0` is the answer, worked out from the errors themselves.

The `else if` on the bike field is deliberate. If the field is empty, "too short" is true as well, and telling somebody two things about one empty box is unhelpful. One problem, one message.

**Job three: show the errors.** This is a small `render` for the error messages, and it follows the same rule as any render: it draws every field from the data, including the ones with nothing wrong, so old messages clear themselves.

```js
function renderErrors(errors) {
  const fields = ['bike', 'day', 'lock-removed'];

  for (const field of fields) {
    const problem = errors.find(function (error) {
      return error.field === field;
    });

    const box = document.querySelector('#' + field + '-error');
    const input = document.querySelector('#' + field);

    if (problem) {
      box.textContent = problem.message;
      input.classList.add('is-invalid');
    } else {
      box.textContent = '';
      input.classList.remove('is-invalid');
    }
  }
}
```

That loop over all the fields is the part worth copying. A version that only writes the messages it has will leave last time's errors sitting on the page after they have been fixed, which is confusing and looks broken. Draw every field, every time.

**And the handler, which is now almost boring:**

```js
function clearBookingForm() {
  document.querySelector('#bike').value = '';
  document.querySelector('#day').value = '';
  document.querySelector('#notes').value = '';
  document.querySelector('#lock-removed').checked = false;
  document.querySelector('#notes-count').textContent = '';
}

function handleBookingSubmit(event) {
  event.preventDefault();

  const booking = readBooking();
  const errors = validateBooking(booking);
  renderErrors(errors);

  const confirmation = document.querySelector('#confirmation');

  if (errors.length > 0) {
    confirmation.textContent = '';
    return;
  }

  confirmation.textContent = 'Booked: ' + booking.bike + ', dropping off on ' + booking.day + '.';

  clearBookingForm();
  renderErrors([]);
}

document.querySelector('#booking-form').addEventListener('submit', handleBookingSubmit);
```

Note that `value` is not read-only. Assigning to it puts text into the field, which is how `clearBookingForm` empties everything. Calling `renderErrors([])` afterwards clears the error styling too, since emptying the fields does not remove messages we wrote ourselves.

### 4.5 `input`, `change` and which to use

Clicks are not the only events forms fire. Two more are worth knowing.

**`input`** fires on every single keystroke, and also on paste, and on dragging text into a field. Use it for anything that should keep up with the user as they type.

```js
const notes = document.querySelector('#notes');

notes.addEventListener('input', function (event) {
  const used = event.target.value.length;
  document.querySelector('#notes-count').textContent = used + ' of 100 characters used.';
});
```

Here `event.target` is the textarea, so the handler does not need to select it again. That works for any event, and it is often tidier than repeating the selector.

**`change`** fires when the user has finished: for a text field, when they click away, and for a checkbox or a select, immediately on choosing. For dropdowns and checkboxes, `change` is the natural choice.

The distinction matters for one specific kindness. Validating a field with `input` means shouting "That is too short!" at somebody who has typed the first letter of their answer. A common compromise is to validate on submit, and only then, once a field has been marked as wrong, use `input` to clear the error as soon as they fix it.

### Exercise 4: sign a tool out properly

**Goal:** to handle a form submission, validate what came in, and feed the result into the state you already have.

**Time:** about 40 minutes.

**Brief**

Add a form to the tool shed from Exercise 3, so that taking a tool out records who took it.

```html
<form id="signout-form" novalidate>
  <div class="field">
    <label for="borrower">Your name</label>
    <input type="text" id="borrower" name="borrower" />
    <p class="error" id="borrower-error"></p>
  </div>

  <div class="field">
    <label for="tool">Which tool?</label>
    <select id="tool" name="tool">
      <option value="">Choose a tool</option>
    </select>
    <p class="error" id="tool-error"></p>
  </div>

  <div class="field">
    <input type="checkbox" id="checked-condition" name="checked-condition" />
    <label for="checked-condition">I have checked it for damage</label>
    <p class="error" id="checked-condition-error"></p>
  </div>

  <button type="submit">Sign it out</button>
</form>
```

**Level 1 process**

1. Fill the `<select>` from `state.tools` inside `render`, with one `<option>` per tool that is currently available. Keep the "Choose a tool" placeholder as the first option, and give each option the tool's `id` as its value.
2. Listen for `submit` on the form and stop the page reloading.
3. Write `readSignout()` returning an object with the trimmed borrower name, the chosen tool id, and the checkbox as a boolean.
4. Write `validateSignout(state, signout)` returning an array of problems. The name must be at least two characters. A tool must be chosen. The condition box must be ticked. As a fourth check, the chosen tool must still be available.
5. Write `renderSignoutErrors(errors)` covering all three fields every time.
6. On success, set that tool's `outWith` to the borrower's name, empty the form, and call `render(state)`.

**Rules:** `validateSignout` must not contain the word `document`. Your submit handler must not set any text or classes directly.

**Level 2 process:** after a failed submission, clear a field's error as soon as the user starts fixing it, using the `input` event on the borrower field. Then explain in one sentence why doing that validation on `input` from the very beginning would be unkind.

<details>
<summary><strong>Solution 4</strong></summary>

```js
// --- reading and validating: no DOM knowledge below this line ---

function validateSignout(state, signout) {
  const errors = [];

  if (signout.borrower.length < 2) {
    errors.push({ field: 'borrower', message: 'Please give your name.' });
  }

  if (signout.toolId === '') {
    errors.push({ field: 'tool', message: 'Please choose a tool.' });
  } else {
    const tool = findTool(state, signout.toolId);

    if (!tool || !isAvailable(tool)) {
      errors.push({ field: 'tool', message: 'Sorry, that tool is no longer available.' });
    }
  }

  if (signout.checkedCondition === false) {
    errors.push({ field: 'checked-condition', message: 'Please check the tool for damage first.' });
  }

  return errors;
}

// --- the DOM side ---

function readSignout() {
  return {
    borrower: document.querySelector('#borrower').value.trim(),
    toolId: document.querySelector('#tool').value,
    checkedCondition: document.querySelector('#checked-condition').checked
  };
}

function renderSignoutErrors(errors) {
  const fields = ['borrower', 'tool', 'checked-condition'];

  for (const field of fields) {
    const problem = errors.find(function (error) {
      return error.field === field;
    });

    const box = document.querySelector('#' + field + '-error');
    const input = document.querySelector('#' + field);

    if (problem) {
      box.textContent = problem.message;
      input.classList.add('is-invalid');
    } else {
      box.textContent = '';
      input.classList.remove('is-invalid');
    }
  }
}

// Called from inside render(state), so the dropdown always matches the board
function renderToolOptions(state) {
  const select = document.querySelector('#tool');
  if (!select) {
    return;
  }

  // Rebuilding the options throws away whatever the user had chosen,
  // so remember it first and put it back afterwards if it is still on offer
  const chosenBefore = select.value;

  select.innerHTML = '';

  const placeholder = document.createElement('option');
  placeholder.value = '';
  placeholder.textContent = 'Choose a tool';
  select.appendChild(placeholder);

  for (const tool of state.tools) {
    if (isAvailable(tool)) {
      const option = document.createElement('option');
      option.value = tool.id;
      option.textContent = tool.name;
      select.appendChild(option);
    }
  }

  // If the tool they picked is still available, keep it selected.
  // If it has gone, the box falls back to the placeholder on its own.
  select.value = chosenBefore;
}

function clearSignoutForm() {
  document.querySelector('#borrower').value = '';
  document.querySelector('#tool').value = '';
  document.querySelector('#checked-condition').checked = false;
}

function handleSignoutSubmit(event) {
  event.preventDefault();

  const signout = readSignout();
  const errors = validateSignout(state, signout);
  renderSignoutErrors(errors);

  if (errors.length > 0) {
    return;
  }

  const tool = findTool(state, signout.toolId);
  tool.outWith = signout.borrower;

  clearSignoutForm();
  render(state);
}

document.querySelector('#signout-form').addEventListener('submit', handleSignoutSubmit);

// Level 2: clear the error the moment the user starts fixing it
document.querySelector('#borrower').addEventListener('input', function (event) {
  if (event.target.value.trim().length >= 2) {
    document.querySelector('#borrower-error').textContent = '';
    event.target.classList.remove('is-invalid');
  }
});
```

Add `renderToolOptions(state);` inside `render(state)`, near the end. That keeps one rule intact: after any change to the state, everything on the page is redrawn from it, including the dropdown. Sign a tool out and it disappears from the list of choices without you writing a single extra line to make that happen.

That convenience has a price, and it is worth seeing it. Redrawing a control the user has already touched throws away what they had done with it: without the two `chosenBefore` lines, taking a tool from the board would silently empty the dropdown the user had just chosen from. Text they have typed goes the same way, which is why the borrower input is never redrawn by `render`. The general rule is: redraw the parts that come from the data, and leave alone the parts the user is holding. Where the two overlap, as they do in this dropdown, save the value and put it back.

The fourth validation check deserves a word, because on this page you cannot actually make it fire. Every route to a tool becoming unavailable goes through `render`, which rebuilds the dropdown and drops the option, so the choice clears itself before you can submit it. That makes the check a backstop rather than a working feature here, and it is still worth writing: it costs three lines, it survives somebody later changing how the dropdown is drawn, and from Module 5 onwards your data will be coming from somewhere else entirely and can go stale between the moment you draw a list and the moment the user acts on it. Never trust that a list drawn a minute ago is still true.

**Level 2 answer.** Validating the name on `input` from the start means the message "Please give your name" appears the instant somebody types the first letter of their name, and stays there while they type the second. The error is technically correct and practically insulting. Errors belong after an attempt, not during one.

</details>

---

## Part 5: The tidy-up, and what it is called

Everything in this part has been in the code all along. Here are the names.

### The three layers

A web page has three jobs, and each has a file.

- **HTML** says *what the content is*. A heading, a list, a button, a form.
- **CSS** says *what it looks like*. Colours, spacing, layout.
- **JavaScript** says *what it does*. Responds to the user, changes the data, redraws.

Keeping them separate is called **separation of concerns**, and the reason is practical rather than aesthetic: when they are tangled, you cannot change one without reading all three.

Two habits do most of the work.

**No behaviour in the HTML.** You will see this in older code and in a lot of tutorials:

```html
<!-- Do not do this -->
<button onclick="increaseRowCount()">Next row</button>
```

It works, and it is worse in several ways. The function has to be a global, or the browser cannot find it. You cannot search your HTML to find out what a page does. You cannot add a second handler. And your behaviour is now scattered across every file that contains a button. `addEventListener` in the JavaScript file keeps the behaviour in one findable place.

**No appearance in the JavaScript.** This is the one that takes discipline:

```js
// Mixing presentation into the behaviour layer
item.style.backgroundColor = '#fffbeb';
item.style.borderLeftColor = '#b45309';

// Naming a state, and letting CSS decide what that looks like
item.classList.add('is-starred');
```

Every example in this lesson uses the second form. The advantage shows up the day somebody wants the starred colour changed: it is one line in the stylesheet, rather than a hunt through the JavaScript for hard-coded colours. It also means a designer can work without touching your logic, and a dark-mode variant costs you nothing.

The class names above start with `is-`: `is-starred`, `is-out`, `is-broken`, `is-invalid`. That is a common convention for classes that represent a state rather than a kind of thing, and it makes them easy to spot in both files.

### Readable code, in practice

The regular lesson lists good habits. Here are the two that this lesson's structure enforces on its own.

**Names that say what a thing is.** `handleBookingSubmit` tells you what it is for and when it runs. `doStuff` and `f2` do not. Functions that create something start with `create`, functions that draw start with `render`, functions that respond to events start with `handle`. Once you follow a convention like that, you can find any function in a file without reading it.

**Comments that say why, not what.** This comment is noise:

```js
// Select the board
const board = document.querySelector('#tool-board');
```

This one earns its place:

```js
// Rebuilding the options throws away what the user had chosen, so save it first
const chosenBefore = select.value;
```

### Keeping globals out

A global variable is one declared outside every function, where any code anywhere can reach it and change it. The trouble is that when a value goes wrong you have the whole file as a suspect list, and if a second script uses the same name, one silently overwrites the other.

Everything in this lesson has lived at the top level of the file, which technically makes `state` and the handlers global. For a small, single-file exercise that is fine, and pretending otherwise would be dishonest. But there is a simple wrapper that fixes it when the file grows, and it uses only things you already know:

```js
'use strict';

function setUpPhrasebook() {
  const state = {
    // ...
  };

  function render(state) {
    // ...
  }

  function handlePhrasebookClick(event) {
    // ...
  }

  document.querySelector('#phrasebook').addEventListener('click', handlePhrasebookClick);

  render(state);
}

setUpPhrasebook();
```

Everything is now inside one function, so nothing leaks out. The listeners still work perfectly well after `setUpPhrasebook` has finished, because the browser kept a reference to the handlers, and those handlers can still see `state`. Lesson 4.4 calls this a closure. You do not need to understand the mechanism today; you need to know that this arrangement is safe, and that the function scope you learnt in Module 2 is the reason it works.

The other guard is the `'use strict';` you have had at the top all along. Without it, a typo creates a variable:

```js
function handleReset() {
  rowCont = 0;    // typo: should be rowCount
}
```

In sloppy mode, that quietly invents a global called `rowCont`, your reset does nothing, and there is no error to follow. In strict mode you get `ReferenceError: rowCont is not defined` and the exact line number. One line at the top of the file, in exchange for never losing an hour to a typo.

### The shape of a file

Once a file has more than a handful of functions, a consistent order helps enormously. Everything in this lesson follows this one:

```js
'use strict';

// 1. State: everything true about this page
const state = {};

// 2. Helpers: plain logic, no DOM
function isAvailable(tool) {}

// 3. Builders: data in, elements out, nothing appended
function createToolItem(tool) {}

// 4. Render: reads state, draws the page
function render(state) {}

// 5. Handlers: change state, then call render
function handleBoardClick(event) {}

// 6. Wiring: attach listeners, draw once
document.querySelector('#tool-board').addEventListener('click', handleBoardClick);
render(state);
```

Sections 2 and 3 never mention events. Section 5 never mentions `textContent`. When you are hunting a bug, that tells you which sixth of the file to read.

---

## When nothing happens: a checklist

Your handler does not run, no error appears, and the page just sits there. Work down this list in order.

**1. Is the file loading?** Put `console.log('app.js running');` on the first line. If you do not see it, the problem is the script tag, not your code.

**2. Did you write `()` when handing over the function?** The signature is: it works once when the page loads, then never again, with no error. See 1.2.

**3. Is the event name spelt exactly right, in lower case?** `'Click'`, `'onclick'` and `'clik'` all register a listener that waits forever without complaining. It is `'click'`, `'submit'`, `'input'`, `'change'`, `'keydown'`.

**4. Did the element exist when you attached the listener?** If you built it with `createElement` afterwards, or if `render` has replaced it since, your listener went with the old element. Use delegation on a container that survives.

**5. Is your selector finding anything?** `document.querySelector('#tool-boad')` returns `null`, and `null.addEventListener` throws immediately. Read the error: it names the line.

**6. Is the page reloading and hiding your work?** If your console output flashes and vanishes, or the form clears itself, you are missing `event.preventDefault()` in a submit handler. Tick "Preserve log" in the console to see output that survives a reload.

**7. Is `event.target` the element you assumed?** If the button contains a span or an icon, `event.target` is the span. Use `event.target.closest('button')`.

**8. Is the button disabled?** Disabled buttons do not fire click events at all. That is usually what you want, but it does look like a broken listener.

**9. Are you changing state but not redrawing?** Log the state at the end of your handler. If the data is right and the page is wrong, you forgot to call `render`.

**10. Is the handler running more than once?** If a counter jumps by two, you have registered two listeners. Look for a registration line inside a function that runs more than once.

---

## Self study task: the village cinema seat picker

**Goal:** to build a complete interactive page from scratch, combining delegation, state and rendering, and a validated form.

**Time:** about 60 to 90 minutes.

**Brief**

The village cinema wants a booking page. The screen has three rows of four seats. Some are already taken. A visitor picks up to four free seats, sees a running total, and confirms with a short form. Everything happens in the browser; there is no server.

Use this as your starting state:

```js
const state = {
  maxSeats: 4,
  pricePerSeat: 120,
  selected: [],
  seats: [
    { id: 'A1', row: 'A', number: 1, taken: false },
    { id: 'A2', row: 'A', number: 2, taken: true  },
    { id: 'A3', row: 'A', number: 3, taken: false },
    { id: 'A4', row: 'A', number: 4, taken: false },
    { id: 'B1', row: 'B', number: 1, taken: false },
    { id: 'B2', row: 'B', number: 2, taken: false },
    { id: 'B3', row: 'B', number: 3, taken: true  },
    { id: 'B4', row: 'B', number: 4, taken: true  },
    { id: 'C1', row: 'C', number: 1, taken: false },
    { id: 'C2', row: 'C', number: 2, taken: false },
    { id: 'C3', row: 'C', number: 3, taken: false },
    { id: 'C4', row: 'C', number: 4, taken: false }
  ]
};
```

`selected` holds the ids of the seats currently chosen, and it starts empty.

**Level 1 process**

1. Write `index.html` with only the shell: a heading, an empty `<div id="seat-plan">`, an empty `<p id="seat-summary">`, the booking form below, and an empty `<p id="confirmation">`. Use `defer` on the script tag and `'use strict';` at the top of the JavaScript.
2. Write the plain helpers first, with no DOM code in them: `isSelected(state, seatId)`, `findSeat(state, seatId)`, and `getRowNames(state)` returning `['A', 'B', 'C']` worked out from the data rather than typed in.
3. Write `createSeatButton(state, seat)` returning a real `<button type="button">` with the seat id as its text, a `data-seat-id` attribute, the `disabled` attribute when the seat is taken, and the classes `is-taken` and `is-selected` where they apply. All colours in CSS.
4. Write `renderSeatPlan(state)` that empties the plan, builds one `<div class="row">` per row, fills each with that row's seat buttons, and writes the summary: `No seats chosen yet. You may choose up to 4.` when nothing is picked, otherwise something like `A1, B1 - 2 of 4 seats, 240 kr.`
5. Write `toggleSeat(state, seatId)`, which changes `state.selected` and returns a message string: empty when all is well, or an explanation when the seat is taken or the limit has been reached. No DOM code in it.
6. Attach **one** click listener to `#seat-plan`. The handler finds the button with `closest`, calls `toggleSeat`, redraws, and puts any returned message into an error paragraph.

**Level 2 process**

7. Add the booking form: a text input for the name, a select offering "Collect at the desk" and "Show on my phone", and a submit button. Label every control.
8. Write `validateBooking(state, details)` returning an array of problems: the name must be at least two characters, a collection method must be chosen, and at least one seat must be selected. No DOM code in it.
9. Write `renderErrors(errors)` covering every field every time, and a submit handler that prevents the default, validates, and on success writes a confirmation such as `Thank you, Marit. B1, C1 booked for 240 kr.`
10. On a successful booking, mark those seats as taken in the state, empty `selected`, clear the form, and redraw. The booked seats should now be unclickable.

**Level 3 process**

11. Check the whole page with the keyboard alone. Tab to a seat, press Enter and space, tab to the form, submit with Enter from inside the text field. Everything should work, and everything should have a visible focus outline. If any of it does not, the cause is almost certainly an element that should have been a `<button>`.
12. Add a "Clear selection" button that empties `selected`. Put it outside `#seat-plan` and give it its own listener, then explain in one sentence why it could not have been handled by the seat-plan listener.
13. Make the summary always list the seats in seat order, regardless of the order the user clicked them, without changing the order of `state.selected`.

<details>
<summary><strong>Solution: self study task</strong></summary>

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Bygdekinoen booking</title>
    <link rel="stylesheet" href="style.css" />
    <script src="app.js" defer></script>
  </head>
  <body>
    <main>
      <h1>Bygdekinoen</h1>
      <p class="screen">Screen this way</p>

      <div id="seat-plan"></div>
      <p id="seat-summary"></p>
      <p class="error" id="seats-error"></p>

      <button type="button" id="clear-seats">Clear selection</button>

      <form id="booking-form" novalidate>
        <div class="field">
          <label for="name">Name for the booking</label>
          <input type="text" id="name" name="name" />
          <p class="error" id="name-error"></p>
        </div>

        <div class="field">
          <label for="collection">How would you like your tickets?</label>
          <select id="collection" name="collection">
            <option value="">Choose one</option>
            <option value="desk">Collect at the desk</option>
            <option value="phone">Show on my phone</option>
          </select>
          <p class="error" id="collection-error"></p>
        </div>

        <button type="submit">Confirm booking</button>
      </form>

      <p id="confirmation"></p>
    </main>
  </body>
</html>
```

`style.css`:

```css
body { font-family: system-ui, sans-serif; margin: 2rem; max-width: 32rem; }

.screen {
  background: #eee;
  text-align: center;
  padding: 0.5rem;
  border-radius: 0.25rem;
  color: #666;
}

.row { display: flex; align-items: center; gap: 0.5rem; margin-bottom: 0.5rem; }
.row-label { width: 4rem; color: #666; font-size: 0.9rem; }

.seat {
  width: 3rem;
  padding: 0.5rem 0;
  border: 1px solid #999;
  border-radius: 0.25rem;
  background: #fff;
  cursor: pointer;
  font: inherit;
}

.seat.is-selected { background: #166534; border-color: #166534; color: #fff; }
.seat.is-taken { background: #eee; color: #aaa; cursor: not-allowed; }
.seat:focus-visible { outline: 3px solid #1d4ed8; outline-offset: 2px; }

#seat-summary { font-weight: 600; }
.error { color: #b91c1c; min-height: 1.2rem; margin: 0.25rem 0; }
.field { margin-bottom: 1rem; }
label { display: block; margin-bottom: 0.25rem; }
```

`app.js`:

```js
'use strict';

// ---------------------------------------------------------------
// 1. State
// ---------------------------------------------------------------

const state = {
  maxSeats: 4,
  pricePerSeat: 120,
  selected: [],
  seats: [
    { id: 'A1', row: 'A', number: 1, taken: false },
    { id: 'A2', row: 'A', number: 2, taken: true  },
    { id: 'A3', row: 'A', number: 3, taken: false },
    { id: 'A4', row: 'A', number: 4, taken: false },
    { id: 'B1', row: 'B', number: 1, taken: false },
    { id: 'B2', row: 'B', number: 2, taken: false },
    { id: 'B3', row: 'B', number: 3, taken: true  },
    { id: 'B4', row: 'B', number: 4, taken: true  },
    { id: 'C1', row: 'C', number: 1, taken: false },
    { id: 'C2', row: 'C', number: 2, taken: false },
    { id: 'C3', row: 'C', number: 3, taken: false },
    { id: 'C4', row: 'C', number: 4, taken: false }
  ]
};

// ---------------------------------------------------------------
// 2. Helpers: plain logic, no DOM anywhere below this comment
// ---------------------------------------------------------------

function isSelected(state, seatId) {
  for (const id of state.selected) {
    if (id === seatId) {
      return true;
    }
  }

  return false;
}

function findSeat(state, seatId) {
  return state.seats.find(function (seat) {
    return seat.id === seatId;
  });
}

// Worked out from the data, so adding a row D needs no code change
function getRowNames(state) {
  const rows = [];

  for (const seat of state.seats) {
    let alreadyThere = false;

    for (const row of rows) {
      if (row === seat.row) {
        alreadyThere = true;
      }
    }

    if (!alreadyThere) {
      rows.push(seat.row);
    }
  }

  return rows;
}

// Walking the seats in board order gives us seat order for free,
// without touching state.selected
function getSelectedInSeatOrder(state) {
  const chosen = [];

  for (const seat of state.seats) {
    if (isSelected(state, seat.id)) {
      chosen.push(seat.id);
    }
  }

  return chosen;
}

function getTotalPrice(state) {
  return state.selected.length * state.pricePerSeat;
}

// Changes state.selected and returns a message for the user, or ''
function toggleSeat(state, seatId) {
  const seat = findSeat(state, seatId);

  if (!seat || seat.taken) {
    return 'That seat is already taken.';
  }

  if (isSelected(state, seatId)) {
    state.selected = state.selected.filter(function (id) {
      return id !== seatId;
    });

    return '';
  }

  if (state.selected.length >= state.maxSeats) {
    return 'You can choose at most ' + state.maxSeats + ' seats.';
  }

  state.selected.push(seatId);
  return '';
}

function validateBooking(state, details) {
  const errors = [];

  if (details.name.length < 2) {
    errors.push({ field: 'name', message: 'Please give us a name for the booking.' });
  }

  if (details.collection === '') {
    errors.push({ field: 'collection', message: 'Please choose how you want your tickets.' });
  }

  if (state.selected.length === 0) {
    errors.push({ field: 'seats', message: 'Choose at least one seat before booking.' });
  }

  return errors;
}

// ---------------------------------------------------------------
// 3. Builders: data in, elements out, nothing appended
// ---------------------------------------------------------------

function createSeatButton(state, seat) {
  const button = document.createElement('button');

  // type="button" so it cannot submit anything by accident
  button.setAttribute('type', 'button');
  button.classList.add('seat');
  button.setAttribute('data-seat-id', seat.id);
  button.textContent = seat.id;

  let label = 'Row ' + seat.row + ', seat ' + seat.number;

  if (seat.taken) {
    button.setAttribute('disabled', 'disabled');
    button.classList.add('is-taken');
    label = label + ', taken';
  }

  if (isSelected(state, seat.id)) {
    button.classList.add('is-selected');
    button.setAttribute('aria-pressed', 'true');
  } else {
    button.setAttribute('aria-pressed', 'false');
  }

  // Tells a screen reader which seat this is, rather than just "A1"
  button.setAttribute('aria-label', label);

  return button;
}

function createRow(state, rowName) {
  const rowElement = document.createElement('div');
  rowElement.classList.add('row');
  rowElement.setAttribute('data-row', rowName);

  const label = document.createElement('span');
  label.classList.add('row-label');
  label.textContent = 'Row ' + rowName;
  rowElement.appendChild(label);

  for (const seat of state.seats) {
    if (seat.row === rowName) {
      rowElement.appendChild(createSeatButton(state, seat));
    }
  }

  return rowElement;
}

// ---------------------------------------------------------------
// 4. Render: reads state, draws the page
// ---------------------------------------------------------------

function renderSeatPlan(state) {
  const plan = document.querySelector('#seat-plan');
  const summary = document.querySelector('#seat-summary');

  if (!plan || !summary) {
    console.log('Seat plan markup is missing, nothing rendered.');
    return;
  }

  plan.innerHTML = '';

  for (const rowName of getRowNames(state)) {
    plan.appendChild(createRow(state, rowName));
  }

  if (state.selected.length === 0) {
    summary.textContent = 'No seats chosen yet. You may choose up to ' + state.maxSeats + '.';
    return;
  }

  summary.textContent = getSelectedInSeatOrder(state).join(', ') + ' - '
    + state.selected.length + ' of ' + state.maxSeats + ' seats, '
    + getTotalPrice(state) + ' kr.';
}

function renderErrors(errors) {
  const fields = ['name', 'collection', 'seats'];

  for (const field of fields) {
    const problem = errors.find(function (error) {
      return error.field === field;
    });

    const box = document.querySelector('#' + field + '-error');

    if (box && problem) {
      box.textContent = problem.message;
    }

    if (box && !problem) {
      box.textContent = '';
    }
  }
}

function clearBookingForm() {
  document.querySelector('#name').value = '';
  document.querySelector('#collection').value = '';
}

// ---------------------------------------------------------------
// 5. Handlers: change state, then redraw
// ---------------------------------------------------------------

function handleSeatPlanClick(event) {
  const button = event.target.closest('button.seat');
  if (!button) {
    return;
  }

  const message = toggleSeat(state, button.getAttribute('data-seat-id'));

  renderSeatPlan(state);
  document.querySelector('#seats-error').textContent = message;
}

function handleClearClick() {
  state.selected = [];
  renderSeatPlan(state);
  document.querySelector('#seats-error').textContent = '';
}

function handleBookingSubmit(event) {
  event.preventDefault();

  const details = {
    name: document.querySelector('#name').value.trim(),
    collection: document.querySelector('#collection').value
  };

  const errors = validateBooking(state, details);
  renderErrors(errors);

  const confirmation = document.querySelector('#confirmation');

  if (errors.length > 0) {
    confirmation.textContent = '';
    return;
  }

  const seatList = getSelectedInSeatOrder(state).join(', ');
  const total = getTotalPrice(state);

  // Mark the booked seats as taken, then start a fresh selection
  for (const seatId of state.selected) {
    findSeat(state, seatId).taken = true;
  }

  state.selected = [];

  confirmation.textContent = 'Thank you, ' + details.name + '. '
    + seatList + ' booked for ' + total + ' kr.';

  clearBookingForm();
  renderSeatPlan(state);
}

// ---------------------------------------------------------------
// 6. Wiring: attach once, draw once
// ---------------------------------------------------------------

document.querySelector('#seat-plan').addEventListener('click', handleSeatPlanClick);
document.querySelector('#clear-seats').addEventListener('click', handleClearClick);
document.querySelector('#booking-form').addEventListener('submit', handleBookingSubmit);

renderSeatPlan(state);
```

**Answers to the Level 3 questions.**

*Question 12.* Delegation only catches events that bubble up through the element you are listening on, and a button outside `#seat-plan` never passes through it, so the seat-plan listener would never see the click.

*Question 13.* `getSelectedInSeatOrder` does not sort anything. It walks `state.seats`, which is already in board order, and picks out the ones that are selected. The order you want is already sitting in your data, so the only work is reading it in the right order. Reaching for a sorting method here would be solving a problem you do not have, and it would risk rearranging `state.selected` behind the user's back.

**What to check when it is running.**

- Clicking a taken seat does nothing and produces no error, because a disabled button does not fire click events at all. The message in `toggleSeat` is a backstop, not the main defence.
- Choosing a fifth seat leaves the first four alone and shows the limit message.
- After a successful booking, the seats you just booked are grey and unclickable, the summary is back to its starting sentence, and the form is empty.
- Every seat can be reached with Tab and pressed with both Enter and space, because it is a real button and you wrote no keyboard code at all.

</details>

---

## What you should be able to say afterwards

If the lesson worked, these should feel obvious rather than clever.

- You hand a function to `addEventListener` without brackets, because you are giving it the function itself, not the result of running it. It is the same arrangement as passing a callback to `forEach`.
- The browser calls your handler and passes it one argument, which is why a parameter you never filled in has something in it.
- Returning a value from a handler achieves nothing, because nobody is collecting it.
- A click happens on the deepest element first and then bubbles outwards, which is why one listener on a container can serve every button inside it.
- `event.target` is where the click landed, which may be a span inside your button, so reach for `closest()`.
- A guard clause at the top of a delegated handler is not optional.
- A handler changes the data and calls `render`. It does not set text, classes or styles.
- Listen for `submit` on the form, not `click` on the button, and call `preventDefault()` first.
- `value` is always a string, and an empty field is `''`.
- Validation is a function that takes data and returns problems. Showing those problems is a different function.
- If something is clickable, it is a `<button>`.

The next module moves on to getting real data from the internet rather than typing it into an array at the top of your file. When it arrives, you will put it in `state` and call `render`, and this whole structure will still be standing.
