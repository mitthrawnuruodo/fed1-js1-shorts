# Extra lesson: Flat paper, and the date that came back wrong

**Estimated time:** about 2 hours for the core path, plus 45 to 60 minutes for the self study task.  
**Prerequisites:** Modules 1 to 5.

The regular lessons show you `localStorage`, then `JSON`, then `Date`, then `Intl`, each one properly and in order. This lesson does something different: it builds one small app, badly, and lets it break. The break is the point. Almost everyone who saves data in a browser hits this exact bug once, usually at eleven o'clock at night, and spends an hour convinced the computer is lying to them.

Two sentences carry the whole lesson.

**The first:** web storage is a shelf that only holds paper. Everything you put on it is flattened into text on the way in, and JSON is the folding and the unfolding.

**The second:** a date is a number wearing a costume. The number survives the fold. The costume does not.

Everything below is those two sentences, worked out in code.

---

## Part 1 - The shelf only holds paper

Open any page in your browser, open the console, and run these three lines.

```js
localStorage.setItem('count', 3);

const count = localStorage.getItem('count');

console.log(count + 1);
```

You saved the number `3`. You asked for it back. You added `1` to it.

The console says `31`.

Not `4`. Thirty-one.

Nothing has gone wrong, and nothing is broken. You have just met the single most important fact about web storage, and you met it the hard way, which is the way it sticks:

> `localStorage` stores strings. Only strings. Always strings.

When you handed it the number `3`, it did not politely refuse. It quietly turned it into the text `"3"` and shelved that instead. When you asked for it back, you got text. And `"3" + 1` in JavaScript is `"31"`, because a string plus anything is a longer string.

Check it yourself:

```js
localStorage.setItem('count', 3);

const count = localStorage.getItem('count');

console.log(count);        // 3
console.log(typeof count); // string
```

The `console.log(count)` line is a little bit of a liar here. It prints `3`, which looks like a number. `typeof` is the one telling the truth.

It gets worse with objects. Try this:

```js
const book = { title: 'Dune', pages: 412 };

localStorage.setItem('book', book);

console.log(localStorage.getItem('book')); // [object Object]
```

Your book is gone. Not corrupted, not partly saved - gone. `localStorage` needed text, so it asked JavaScript to turn the object into text, and JavaScript's default answer for any plain object is the useless string `[object Object]`. The title and the page count were never written down at all.

So the shelf only holds paper. Which raises the obvious question: how do you get an object onto a shelf that only holds paper?

You fold it flat first.

### Folding and unfolding

`JSON.stringify` folds. It takes a value and gives you back a string that describes it.

```js
const book = { title: 'Dune', pages: 412 };

const folded = JSON.stringify(book);

console.log(folded);        // {"title":"Dune","pages":412}
console.log(typeof folded); // string
```

`JSON.parse` unfolds. It takes such a string and gives you back a real, working value.

```js
const folded = '{"title":"Dune","pages":412}';

const unfolded = JSON.parse(folded);

console.log(unfolded.title);  // Dune
console.log(unfolded.pages);  // 412
console.log(typeof unfolded); // object
```

Note what came back out of `pages`: the number `412`, not the text `"412"`. JSON writes down what kind of thing each value was, so unfolding restores it. That is the difference between JSON and just turning things into text with `+`.

Put the two halves together and the shelf works properly:

```js
const book = { title: 'Dune', pages: 412 };

localStorage.setItem('book', JSON.stringify(book));

const stored = JSON.parse(localStorage.getItem('book'));

console.log(stored.pages + 1); // 413
```

`413`. The shelf still only holds paper. We just learnt to fold.

> **Tip: watch the shelf while you work.** Open your browser's developer tools and find the **Application** tab (Chrome and Edge) or **Storage** tab (Firefox). Under Local Storage you will see every key you have saved and its exact value, as text. Keep it open for the rest of this lesson. When something looks wrong, looking at what is actually on the shelf will usually tell you why in about four seconds.

### Exercise 1 - Predict, then check

Do the predicting first. Write your guesses down, on paper or in a comment. Guessing and being wrong is what makes this stick.

**Goal:** to be certain about what survives a trip through storage and what does not.

**Steps**

1. For each of the five values below, predict two things: what `getItem` will return, and what `typeof` will say about it. Assume each one is stored with `setItem` directly, with no `JSON.stringify`.

   - the number `42`
   - the boolean `true`
   - the array `[1, 2, 3]`
   - the string `'hello'`
   - the object `{ a: 1 }`

2. Now write the code and check. For each value, store it, read it back, and log both the value and its `typeof`.
3. Pick the two that surprised you most and store them again, this time with `JSON.stringify` on the way in and `JSON.parse` on the way out. Log the `typeof` again.
4. Write yourself a one-line comment explaining the difference.

<details>
<summary>Solution</summary>

```js
// Step 2: everything stored directly comes back as a string

localStorage.setItem('a', 42);
localStorage.setItem('b', true);
localStorage.setItem('c', [1, 2, 3]);
localStorage.setItem('d', 'hello');
localStorage.setItem('e', { a: 1 });

console.log(localStorage.getItem('a'), typeof localStorage.getItem('a'));
// 42 string

console.log(localStorage.getItem('b'), typeof localStorage.getItem('b'));
// true string

console.log(localStorage.getItem('c'), typeof localStorage.getItem('c'));
// 1,2,3 string

console.log(localStorage.getItem('d'), typeof localStorage.getItem('d'));
// hello string

console.log(localStorage.getItem('e'), typeof localStorage.getItem('e'));
// [object Object] string
```

The array is the sneaky one. `[1, 2, 3]` becomes the text `"1,2,3"`, which looks almost right and is completely useless. You cannot call `.length` on it and get `3`, and you cannot loop over it and get numbers.

```js
// Step 3: the same two values, folded properly

localStorage.setItem('c', JSON.stringify([1, 2, 3]));
localStorage.setItem('e', JSON.stringify({ a: 1 }));

const list = JSON.parse(localStorage.getItem('c'));
const item = JSON.parse(localStorage.getItem('e'));

console.log(list, typeof list, list.length); // [1, 2, 3] object 3
console.log(item, typeof item, item.a);      // {a: 1} object 1
```

```js
// Step 4
// setItem flattens anything into text; JSON.stringify flattens it into text
// that remembers what it used to be, so JSON.parse can rebuild it.
```

Note that `typeof` says `object` for the array. That is correct and slightly annoying: in JavaScript, arrays are a kind of object. `Array.isArray(list)` is how you tell them apart when you need to.

</details>

---

## Part 2 - Something worth saving

Reading about storage is dull. Let us build the smallest useful thing that needs it.

A **lending shelf**: a list of things you have lent to people, so that six months from now you can remember who has your good scissors.

Create a folder with two files.

**index.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Lending shelf</title>
  </head>
  <body>
    <h1>Lending shelf</h1>

    <form id="lend-form">
      <label for="item">What did you lend?</label>
      <input type="text" id="item" required />

      <label for="borrower">Who has it?</label>
      <input type="text" id="borrower" required />

      <button type="submit">Add to shelf</button>
    </form>

    <ul id="shelf-list"></ul>

    <script src="script.js"></script>
  </body>
</html>
```

**script.js**

```js
const form = document.querySelector('#lend-form');
const itemInput = document.querySelector('#item');
const borrowerInput = document.querySelector('#borrower');
const list = document.querySelector('#shelf-list');

let shelf = [];

function save() {
  localStorage.setItem('lendingShelf', JSON.stringify(shelf));
}

function load() {
  const saved = localStorage.getItem('lendingShelf');

  if (saved) {
    shelf = JSON.parse(saved);
  }
}

function render() {
  list.textContent = '';

  shelf.forEach(function (entry) {
    const listItem = document.createElement('li');
    listItem.textContent = entry.item + ' - ' + entry.borrower;
    list.appendChild(listItem);
  });
}

form.addEventListener('submit', function (event) {
  event.preventDefault();

  const entry = {
    item: itemInput.value,
    borrower: borrowerInput.value
  };

  shelf.push(entry);
  save();
  render();

  itemInput.value = '';
  borrowerInput.value = '';
});

load();
render();
```

Open it, add two or three things, then reload the page.

They are still there.

That is genuinely the whole trick, and it is worth pausing on, because it is the first time anything you have written has outlived the page. Every JavaScript variable you have made so far died the moment the tab reloaded. This one did not.

Look at the shape of it. Four small functions and nothing clever:

- `save` folds the whole array flat and puts it on the shelf.
- `load` takes it off the shelf and unfolds it.
- `render` draws whatever is in the array.
- The submit handler changes the array, then saves, then renders.

`load()` runs once at the bottom of the file, before the first `render()`. That order matters: fill the array first, draw second.

> **Tip: `if (saved)` is doing real work.** On a brand new browser there is nothing under the key `lendingShelf`, and `getItem` returns `null` for keys that do not exist. Handing `null` to `JSON.parse` is not a good time. The `if` skips the unfolding entirely on a first visit, and `shelf` keeps its starting value of `[]`.

> **Tip: how to wipe the shelf while you are developing.** You will want this within about ten minutes. In the console, run `localStorage.removeItem('lendingShelf')` and reload. Or right-click the key in the Application tab and delete it. Avoid `localStorage.clear()` out of habit - it wipes everything the site has stored, which on a real project is a rude surprise.

### Exercise 2 - Giving things back

Right now, things go onto the shelf and never come off.

**Goal:** to change the array, save, and redraw, in that order.

**Steps**

1. In `render`, create a button for each entry with the text `Returned`, and append it to the list item alongside the text.
2. Give the button a click listener that removes that entry from `shelf`.
3. Make sure the change survives a reload.

The `filter` method from Module 3 is the neat way to remove one item from an array. Remember that `filter` gives you a **new** array rather than changing the old one, so you will need to assign the result back to `shelf`.

<details>
<summary>Solution</summary>

```js
function render() {
  list.textContent = '';

  shelf.forEach(function (entry, index) {
    const listItem = document.createElement('li');
    listItem.textContent = entry.item + ' - ' + entry.borrower + ' ';

    const returnedButton = document.createElement('button');
    returnedButton.textContent = 'Returned';

    returnedButton.addEventListener('click', function () {
      shelf = shelf.filter(function (entryToCheck, indexToCheck) {
        return indexToCheck !== index;
      });

      save();
      render();
    });

    listItem.appendChild(returnedButton);
    list.appendChild(listItem);
  });
}
```

The click listener keeps hold of the `index` from the `forEach` it was created inside, which is why each button knows which entry it belongs to.

The order at the end of the listener is the same order as in the submit handler, and it is always the same order: **change the array, save it, redraw it.** If you forget `save()`, the item vanishes from the page and comes straight back on reload, which is a confusing bug to look at and a very easy one to cause.

</details>

---

## Part 3 - Adding time, and breaking everything

The shelf is more useful if it remembers *when* you lent each thing.

JavaScript's tool for a moment in time is the `Date` object. Ask for one with no arguments and you get right now:

```js
const now = new Date();

console.log(now); // something like Wed Aug 26 2026 11:12:00 GMT+0200
```

A `Date` knows how to answer questions about itself. There is a whole family of these methods, all beginning with `get`:

```js
const now = new Date();

console.log(now.getDate());     // day of the month, 1 to 31
console.log(now.getMonth());    // month, 0 to 11 - January is 0
console.log(now.getFullYear()); // e.g. 2026
console.log(now.getHours());    // 0 to 23
```

`getMonth()` counting from zero is a genuine trap that has shipped a great many off-by-one bugs. January is `0`, December is `11`. There is no good reason for it. It is just how it is, and the way to survive it is to expect it.

So let us stamp each entry with the moment it was lent. Two small changes.

In the submit handler:

```js
const entry = {
  item: itemInput.value,
  borrower: borrowerInput.value,
  lentAt: new Date()
};
```

And in `render`:

```js
listItem.textContent =
  entry.item + ' - ' + entry.borrower + ' (day ' + entry.lentAt.getDate() + ')';
```

Save, reload the page, add an item.

It works. The day number appears. Everything is fine.

**Now reload the page.**

The list is empty, and the console has this in it:

```
Uncaught TypeError: entry.lentAt.getDate is not a function
```

Do not fix it yet. Sit with it for a second, because this error is the most useful thing in this lesson.

Nothing about your code changed between the working run and the broken one. The only thing that happened in between is that your data went onto the shelf and came back off it. And it came back different.

Go and look. Open the Application tab and read the value under `lendingShelf`:

```
[{"item":"Scissors","borrower":"Ola","lentAt":"2026-08-26T09:12:00.000Z"}]
```

There is your date, and there are the quotation marks around it. It is a **string**.

### Why

JSON is a very small language. It can describe exactly six kinds of value, and that is the entire list:

- strings
- numbers
- `true` and `false`
- `null`
- arrays
- objects

There is no date on that list. There is no date in JSON at all.

So when `JSON.stringify` reached your `Date` object and found something it had no way to write down, it asked the `Date` for a text version of itself, and the `Date` handed over its ISO string: `2026-08-26T09:12:00.000Z`. Sensible. Readable. Sortable. Text.

And when `JSON.parse` read that back, it saw a string that looked like a date and did the only honest thing available to it: it gave you a string. `JSON.parse` cannot know that this particular text used to be a `Date` and that other text is just text. `"2026-08-26"` might be a date, or it might be a filename, or a password. Guessing would be worse than not guessing.

Strings do not have a `getDate` method. Hence the error.

**A date is a number wearing a costume. The costume did not survive the fold.**

### Two ways out

**The first: put the costume back on.** If you know a particular field ought to be a date, rebuild it after unfolding. `new Date()` accepts an ISO string and gives you a working `Date` object back.

```js
function load() {
  const saved = localStorage.getItem('lendingShelf');

  if (saved) {
    shelf = JSON.parse(saved);

    shelf.forEach(function (entry) {
      entry.lentAt = new Date(entry.lentAt);
    });
  }
}
```

Reload. It works. This is called **reviving**, and you will need it whenever the data comes from somewhere you do not control, such as a server. More on that in Part 6.

**The second: never send the costume in the first place.** Underneath, a `Date` is only a number - the count of milliseconds since midnight on 1 January 1970, UTC. That moment is called the Unix epoch, and it is the zero point every JavaScript date is measured from.

You can get that number directly, without an object anywhere near it:

```js
console.log(Date.now()); // e.g. 1787735520000 - a big, unfriendly, extremely useful number
```

And numbers, unlike dates, are on JSON's list of six. A number goes onto the shelf as a number and comes back as a number, unchanged, every time.

So store the number:

```js
const entry = {
  item: itemInput.value,
  borrower: borrowerInput.value,
  lentAt: Date.now()
};
```

Undo the reviving loop in `load` - we do not need it any more - and build a `Date` only at the moment you actually want to display something:

```js
function render() {
  list.textContent = '';

  shelf.forEach(function (entry) {
    const lentDate = new Date(entry.lentAt);

    const listItem = document.createElement('li');
    listItem.textContent =
      entry.item + ' - ' + entry.borrower + ' (day ' + lentDate.getDate() + ')';

    list.appendChild(listItem);
  });
}
```

Note that `new Date()` takes a timestamp number just as happily as it takes an ISO string. Same object, either way in.

**Which should you use?** For data you create yourself, store the number. It is smaller, it is unambiguous, it needs no repair, and it never has this bug. For data that arrives from elsewhere already carrying ISO strings, revive it. Both live in real codebases, and knowing why each exists is more useful than a rule.

We will carry on with the number.

> **Wipe the shelf now.** Your existing entries still have ISO strings in `lentAt` from the earlier version, and mixing the two formats will produce a second confusing bug on top of the first. `localStorage.removeItem('lendingShelf')`, reload, and add fresh items.

<details>
<summary>Rabbit hole: how did stringify know to ask the Date for text?</summary>

`JSON.stringify` follows one rule when it meets an object it does not recognise: if that object has a method called `toJSON`, call it and write down whatever it returns.

`Date` has one. It returns the ISO string. That is the whole mechanism.

```js
const now = new Date();

console.log(now.toJSON());               // 2026-08-26T09:12:00.000Z
console.log(now.toISOString());          // the same string
console.log(JSON.stringify({ at: now })); // {"at":"2026-08-26T09:12:00.000Z"}
```

You can put a `toJSON` method on your own objects and `stringify` will use it. It is rarely worth doing, but it explains why dates get this one small privilege and other objects do not.

Notice that the mechanism only runs in one direction. There is nothing on the way back that corresponds to it, which is precisely why you have to revive by hand.

</details>

<details>
<summary>Rabbit hole: other things JSON quietly loses</summary>

The date is the one that bites people, but it is not alone. Because JSON only knows six kinds of value, anything else has to be dropped or changed.

```js
const awkward = {
  name: 'test',
  nothing: undefined,
  doSomething: function () { return 1; },
  notANumber: NaN,
  tooBig: Infinity,
  nested: { fine: true }
};

console.log(JSON.stringify(awkward));
// {"name":"test","notANumber":null,"tooBig":null,"nested":{"fine":true}}
```

Three things happened there:

- `undefined` values are **dropped entirely**. The key vanishes.
- Functions are **dropped entirely**, for the same reason.
- `NaN` and `Infinity` both become `null`, because JSON's numbers have no way to express them.

`null` itself is fine and survives untouched. It is on the list.

The practical lesson: what comes back out of storage is data, not behaviour. If your objects have methods on them, the methods are not coming back.

</details>

### Exercise 3 - Repair the broken save

Someone has written a small "recently played" list for a music site. It saves fine and crashes on reload with `playedAt.getHours is not a function`.

```js
let history = [];

function saveHistory() {
  localStorage.setItem('playHistory', JSON.stringify(history));
}

function loadHistory() {
  const saved = localStorage.getItem('playHistory');

  if (saved) {
    history = JSON.parse(saved);
  }
}

function addPlay(trackName) {
  history.push({ track: trackName, playedAt: new Date() });
  saveHistory();
}

function showHistory() {
  history.forEach(function (play) {
    console.log(play.track + ' at ' + play.playedAt.getHours() + ':00');
  });
}
```

**Goal:** to fix it twice, once each way, and to be able to say which you would ship.

**Steps**

1. Fix it by reviving in `loadHistory`.
2. Then fix it instead by never storing a `Date` at all.
3. Confirm each version survives `loadHistory()` followed by `showHistory()`.
4. Write one sentence on which you would keep and why.

<details>
<summary>Solution</summary>

**Fix 1 - revive on load**

```js
function loadHistory() {
  const saved = localStorage.getItem('playHistory');

  if (saved) {
    history = JSON.parse(saved);

    history.forEach(function (play) {
      play.playedAt = new Date(play.playedAt);
    });
  }
}
```

Nothing else changes. `addPlay` still stores a `Date`, and `showHistory` still calls `getHours` on one.

**Fix 2 - store the number**

```js
function addPlay(trackName) {
  history.push({ track: trackName, playedAt: Date.now() });
  saveHistory();
}

function showHistory() {
  history.forEach(function (play) {
    const playedDate = new Date(play.playedAt);
    console.log(play.track + ' at ' + playedDate.getHours() + ':00');
  });
}
```

`loadHistory` is left exactly as it was, which is the point: there is nothing to repair, because nothing broke.

**Step 4**

Fix 2. The data is ours, so we get to choose its shape, and a shape that cannot break is better than a shape we remember to repair. Fix 1 has a failure mode that Fix 2 does not: add one more date field later, forget to add it to the reviving loop, and the bug is back in a new spot.

</details>

---

## Part 4 - Sums with time

Two weeks is a fair loan. After that you are allowed to send a slightly pointed message.

Working out a due date sounds like it should need a calendar. It does not, because we stored a number. Two weeks is just an amount of milliseconds you can add.

Milliseconds are a small unit, so the number is large. Build it from parts you can read, rather than typing `86400000` and hoping:

```js
const DAY_IN_MS = 24 * 60 * 60 * 1000; // hours * minutes * seconds * milliseconds
```

Then a due date is one line of arithmetic:

```js
function dueDate(entry) {
  return entry.lentAt + 14 * DAY_IN_MS;
}
```

And "how long left" is a subtraction. Comparing two moments is the thing timestamps are genuinely excellent at, because comparing two numbers is easy and comparing two calendars is not:

```js
function daysLeft(entry) {
  const difference = dueDate(entry) - Date.now();

  return Math.ceil(difference / DAY_IN_MS);
}
```

`Math.ceil` rounds up, so anything still owing today counts as a whole day rather than `0.3` of one. If the loan is overdue the difference goes negative, and so does the answer, which is exactly what we want.

Wire it into `render`:

```js
shelf.forEach(function (entry) {
  const remaining = daysLeft(entry);

  let status = remaining + ' days left';

  if (remaining < 0) {
    status = 'overdue by ' + Math.abs(remaining) + ' days';
  }

  const listItem = document.createElement('li');
  listItem.textContent = entry.item + ' - ' + entry.borrower + ' (' + status + ')';
  list.appendChild(listItem);
});
```

Everything you have just added is arithmetic on plain numbers. No `Date` object appears anywhere in Part 4. That is the practical argument for storing the number: the moment you want to *compare* or *measure* time rather than *display* it, a number is the easier thing to hold.

> **Tip: testing this without waiting two weeks.** You do not need to. Add an item, then in the console change its timestamp by hand and redraw:
>
> ```js
> shelf[0].lentAt = Date.now() - 20 * 24 * 60 * 60 * 1000;
> save();
> render();
> ```
>
> That item was now lent twenty days ago and is six days overdue. Being able to fake the clock like this is a real testing technique, not a trick for the lesson.

<details>
<summary>Rabbit hole: fourteen days of milliseconds is not always fourteen days</summary>

`14 * DAY_IN_MS` is exactly 1,209,600,000 milliseconds. That is fourteen twenty-four-hour periods.

Most of the year, fourteen twenty-four-hour periods and "the same time of day, a fortnight later" are the same thing. Twice a year, in countries with daylight saving, they are not. One day in spring is 23 hours long and one day in autumn is 25. Add a fixed pile of milliseconds across one of those and your due date lands an hour out.

For "roughly two weeks" that is completely fine and nobody will notice. For anything where the clock time must hold, the other approach is to let the calendar do the counting:

```js
const due = new Date(entry.lentAt);
due.setDate(due.getDate() + 14);
```

`setDate` is a **setter** - it changes the `Date` object rather than returning a new one - and it handles rollover for you. Set the date to the 40th of a 31-day month and it becomes the 9th of the next month. Add days across a daylight saving change and the clock time stays put.

Which to use is a judgement call about what "two weeks later" means in your particular app. Both are correct answers to slightly different questions.

</details>

### Exercise 4 - Sort out the shelf

**Goal:** to compare timestamps and count things.

**Steps**

1. Write a function `overdueCount()` that returns how many entries on the shelf are past their due date.
2. Show that number in a heading above the list, in the form `2 items overdue`. Add an empty `<h2 id="summary"></h2>` to your HTML and fill it from `render`.
3. Write a function `oldestLoan()` that returns the entry that has been out the longest, or `null` if the shelf is empty.

For step 3, `filter` will not help you - it selects many, and you want one. A plain loop that keeps track of the best answer so far is the honest tool here.

<details>
<summary>Solution</summary>

```js
const summary = document.querySelector('#summary');

function overdueCount() {
  const overdue = shelf.filter(function (entry) {
    return daysLeft(entry) < 0;
  });

  return overdue.length;
}

function oldestLoan() {
  if (shelf.length === 0) {
    return null;
  }

  let oldest = shelf[0];

  shelf.forEach(function (entry) {
    if (entry.lentAt < oldest.lentAt) {
      oldest = entry;
    }
  });

  return oldest;
}
```

And in `render`, after the loop:

```js
summary.textContent = overdueCount() + ' items overdue';
```

The comparison in `oldestLoan` is worth a second look: `entry.lentAt < oldest.lentAt`. Those are two plain numbers, and a **smaller** timestamp means **further in the past**, because every timestamp counts upwards from 1970. So the oldest loan is the smallest number.

Starting `oldest` at `shelf[0]` rather than at something like `0` is deliberate. There is no sensible "empty" value to start a comparison from here, so we start with a real entry and let the loop improve on it. The early `return null` is what makes that safe.

You will also notice it says `1 items overdue` when there is one. That is an `if` and two words away from being right, and it is worth doing, because this is exactly the sort of thing users notice and developers stop seeing.

</details>

---

## Part 5 - Showing it to a human

Our list currently says things like `(day 26)`, which is not a date so much as a fragment of one. Time to display it properly.

The tempting approach is to build the string yourself out of getters:

```js
const lentDate = new Date(entry.lentAt);

const bad =
  lentDate.getDate() + '/' + (lentDate.getMonth() + 1) + '/' + lentDate.getFullYear();

console.log(bad); // 26/8/2026
```

That works, technically. It also has a `+ 1` in it that you will forget, no leading zeros, and a `/` separator that is wrong in Norway, wrong in Japan, and means something different in the United States. `26/8/2026` and `8/26/2026` are the same day written two ways, and there is no way for a reader to tell which convention a site is using except by guessing.

The browser already knows all of this. Every browser ships with a full set of formatting rules for every language and region it supports, and `Intl` is how you ask for them.

```js
const lentDate = new Date('2026-08-26T09:12:00Z');

const uk = new Intl.DateTimeFormat('en-GB', { dateStyle: 'medium' });

console.log(uk.format(lentDate)); // 26 Aug 2026
```

Two pieces:

- The **locale**, `'en-GB'` - a language tag and usually a region. `'nb-NO'` is Norwegian Bokmal as used in Norway, `'en-US'` is American English, `'ja-JP'` is Japanese.
- The **options**, `{ dateStyle: 'medium' }` - how much detail you want. The four choices are `'short'`, `'medium'`, `'long'` and `'full'`, and each one means something slightly different in each locale, which is the entire point.

Try the same instant in two places:

```js
const lentDate = new Date('2026-08-26T09:12:00Z');

const uk = new Intl.DateTimeFormat('en-GB', { dateStyle: 'long' });
const no = new Intl.DateTimeFormat('nb-NO', { dateStyle: 'long' });

console.log(uk.format(lentDate)); // 26 August 2026
console.log(no.format(lentDate)); // 26. august 2026
```

Note the differences you did not have to know about: the full stop after the day, and the lower-case month name, because Norwegian does not capitalise months and English does. Nobody wrote that rule into your app. It came with the browser.

Notice also that we made the formatter **once** and then called `format` on it. That ordering is deliberate. Building a formatter is the expensive part - the browser has to go and load the rules for that locale - and calling `format` afterwards is cheap. Make it once, outside your loop, and reuse it:

```js
const dateFormatter = new Intl.DateTimeFormat('en-GB', { dateStyle: 'medium' });

function render() {
  list.textContent = '';

  shelf.forEach(function (entry) {
    const lentDate = new Date(entry.lentAt);
    const remaining = daysLeft(entry);

    let status = remaining + ' days left';

    if (remaining < 0) {
      status = 'overdue by ' + Math.abs(remaining) + ' days';
    }

    const listItem = document.createElement('li');
    listItem.textContent =
      entry.item +
      ' - ' +
      entry.borrower +
      ' - lent ' +
      dateFormatter.format(lentDate) +
      ' (' +
      status +
      ')';

    list.appendChild(listItem);
  });

  summary.textContent = overdueCount() + ' items overdue';
}
```

The shape of the app is now the thing worth noticing:

> The timestamp is stored as a number, compared as a number, and turned into words at the last possible moment, in one place.

Nothing in the file except that one `format` call cares what language the user speaks. If you had built display strings and stored *those*, every one of those decisions would be baked into the shelf, and changing your mind would mean rewriting saved data.

### Exercise 5 - Let the user choose

**Goal:** to store a preference, apply it on load, and see why formatters belong outside the loop.

**Steps**

1. Add two buttons to your HTML: `English` and `Norsk`.
2. Clicking one should redraw every date on the page in that locale.
3. The choice should survive a reload.
4. This one is a plain string, not an object. Store it without `JSON.stringify` and note how much less work that is.

<details>
<summary>Solution</summary>

In `index.html`, above the list:

```html
<button id="lang-en">English</button>
<button id="lang-no">Norsk</button>
```

In `script.js`:

```js
const englishButton = document.querySelector('#lang-en');
const norwegianButton = document.querySelector('#lang-no');

let locale = 'en-GB';
let dateFormatter = new Intl.DateTimeFormat(locale, { dateStyle: 'medium' });

function setLocale(newLocale) {
  locale = newLocale;
  dateFormatter = new Intl.DateTimeFormat(locale, { dateStyle: 'medium' });

  localStorage.setItem('shelfLocale', locale);

  render();
}

englishButton.addEventListener('click', function () {
  setLocale('en-GB');
});

norwegianButton.addEventListener('click', function () {
  setLocale('nb-NO');
});
```

And in `load`, alongside the shelf:

```js
function load() {
  const saved = localStorage.getItem('lendingShelf');

  if (saved) {
    shelf = JSON.parse(saved);
  }

  const savedLocale = localStorage.getItem('shelfLocale');

  if (savedLocale) {
    locale = savedLocale;
    dateFormatter = new Intl.DateTimeFormat(locale, { dateStyle: 'medium' });
  }
}
```

Two things to take from this.

`locale` and `dateFormatter` are declared with `let` rather than `const`, because both get replaced when the user chooses. A formatter is fixed to one locale once it is built, so switching language means building a new one.

Those two lines **replace** the `const dateFormatter` line from Part 5. Leaving both in gives you `SyntaxError: Identifier 'dateFormatter' has already been declared`, and a script with a syntax error in it does not run at all, so the whole page will look dead.

And the preference needed no `JSON.stringify` and no `JSON.parse`, because `'nb-NO'` is already a string, and the shelf only ever wanted a string. Folding is only for things that are not paper yet.

</details>

<details>
<summary>Rabbit hole: what if you do not know the user's locale?</summary>

Pass `undefined` as the locale and the browser uses whatever the user has set in their operating system or browser:

```js
const theirs = new Intl.DateTimeFormat(undefined, { dateStyle: 'full' });

console.log(theirs.format(new Date()));
```

This is usually the right default for a public site: it is already correct for most visitors and it needs no buttons. Offer an explicit choice on top of it when you have a reason - people using a borrowed machine, or living in one country in another country's language.

To see what the browser thinks you want, `navigator.language` will tell you.

`toLocaleDateString()` on a `Date` does much the same job in one call and is fine for a one-off. It builds a formatter internally every time you call it, though, so in a loop over a hundred items you have built a hundred formatters. That is the case for `Intl.DateTimeFormat` and a variable.

</details>

---

## Part 6 - The same trip, in reverse

One more turn, and it closes the loop with Module 5.

Everything so far has been data you made, folded, and unfolded yourself. Most real data arrives from a server instead - and a server sends JSON, because JSON is what everything agrees on.

Which means server data has exactly the same six-value limitation, and exactly the same consequence.

Here is a response body of the kind a lending app's backend might send. Paste it in as it is; no network required:

```js
const returnedFromServer = [
  { item: 'Cake tin', borrower: 'Sara', returnedAt: '2026-05-02T17:30:00Z' },
  { item: 'Tent pegs', borrower: 'Ida', returnedAt: '2026-06-19T08:05:00Z' },
  { item: 'Allen keys', borrower: 'Jonas', returnedAt: '2026-07-30T14:45:00Z' }
];

returnedFromServer.forEach(function (record) {
  console.log(record.returnedAt.getFullYear());
});
```

Same error as Part 3. Same reason. The server could not send you a `Date` either, so it sent text - and the text it sent is the ISO format, for the same reason your own `stringify` produced ISO earlier. It is the format everyone settled on.

The fix is the reviving you already know:

```js
returnedFromServer.forEach(function (record) {
  const returnedDate = new Date(record.returnedAt);

  console.log(record.item + ' came back on ' + dateFormatter.format(returnedDate));
});
```

Read the `Z` on the end of those strings. It means the time is in UTC, the world's shared clock, rather than in whoever's local time. That is why servers use it: a moment written in UTC means one unambiguous instant everywhere, and `new Date()` converts it to the reader's own time zone automatically when it displays. A server that sent `17:30` with no `Z` would be sending a puzzle.

So the round trip is symmetrical, and it is worth holding onto as one shape:

> Dates go out as text, and come back as text. Somebody, at some point, has to put the costume back on. That somebody is you.

<details>
<summary>Rabbit hole: storage as a cache</summary>

Once you can store data and stamp it with a time, you can avoid asking the server for things you already have.

The pattern is short and it uses every single thing in this lesson:

```js
const ONE_HOUR = 60 * 60 * 1000;

async function getFilms() {
  const cached = localStorage.getItem('films');

  if (cached) {
    const box = JSON.parse(cached);

    if (Date.now() - box.savedAt < ONE_HOUR) {
      console.log('using the copy we already have');
      return box.data;
    }
  }

  console.log('fetching a fresh copy');

  const response = await fetch('https://ghibliapi.vercel.app/films');
  const data = await response.json();

  localStorage.setItem('films', JSON.stringify({ savedAt: Date.now(), data: data }));

  return data;
}
```

The trick is the wrapper object. Rather than storing the data on its own, we store `{ savedAt, data }`, so the copy carries the time it was made. Without the timestamp there is no way to tell a fresh copy from a stale one, and a cache with no expiry is just a bug with good intentions.

Run it twice and watch the console messages change. Wait an hour, or fake it by editing `savedAt` in the console, and it fetches again.

</details>

---

## One loose end, deliberately left

Try this in the console, on your lending shelf page:

```js
localStorage.setItem('lendingShelf', 'this is not JSON');
```

Now reload.

The page is blank and the console says something about unexpected tokens. `JSON.parse` was handed something that is not JSON, and it did the only thing it can do: it threw an error, and the error stopped your script dead before `render` ever ran.

You did that on purpose, so it looks like a silly thing to worry about. It is not. Storage can hold text written by an older version of your own code, by a different tab, by a browser extension, or by a user poking about in the console exactly as you just did. `JSON.parse` sits there in `load` with no protection at all, and any of those will take the whole page down.

There is a tool for this. It is `try...catch`, it is Lesson 6.4, and it is where the next extra lesson starts.

For now: `localStorage.removeItem('lendingShelf')` and reload.

---

## Self study task: a practice log

Different app, same three moves: store a number, compare numbers, format at the end.

**The idea.** A log for practising something - an instrument, a language, drawing, running. You record a session, and the page tells you how you are doing.

**Level 1**

1. A form with two fields: what you practised, and for how many minutes.
2. Each session is stored as an object with three properties: the activity, the minutes as a number, and `at` as a timestamp from `Date.now()`.
3. Sessions persist across reloads.
4. The list shows each session with its date formatted through `Intl.DateTimeFormat`.

**Level 2**

5. Show a total of minutes practised **today**. This is the interesting one, because "today" is a range, not a moment: a session counts if its timestamp is after midnight this morning. The way to find that boundary is to take the current date and set its clock to zero.

   ```js
   const startOfToday = new Date();
   startOfToday.setHours(0, 0, 0, 0);

   console.log(startOfToday.getTime()); // midnight this morning, as a number
   ```

   `setHours` takes hours, minutes, seconds and milliseconds, so that one line flattens the whole clock. Compare each session's `at` against that number.

6. Show a total for the last seven days as well.
7. Show the date and time of the most recent session, using both `dateStyle` and `timeStyle` in your formatter options.

**Level 3 (recommended)**

8. Add a language toggle as in Exercise 5, and store the choice.
9. Add a "longest gap" figure: the biggest number of whole days between two consecutive sessions. Sessions arrive in the order they were added, so consecutive entries in the array are consecutive in time.
10. Make the page survive rubbish in storage. You have not been taught the tool for this yet, so solve it with what you have: before parsing, check that the stored text at least starts with `[`, and fall back to an empty array if it does not. It is a blunt instrument and it is not what you would ship, which is worth knowing before you meet the proper one.

<details>
<summary>Hints, if you want them</summary>

**On step 5.** `setHours` changes the `Date` object it is called on rather than returning a new one. So make a fresh `new Date()` for it to flatten - do not call it on a date you still need intact.

**On step 6.** Seven days ago is `Date.now() - 7 * DAY_IN_MS`. Whether you want a rolling seven days or the last seven calendar days starting at midnight is a real design decision, and either answer is defensible as long as you know which one you chose.

**On step 7.** `{ dateStyle: 'medium', timeStyle: 'short' }`. The two can be combined freely. What you cannot do is mix `dateStyle` with individual options such as `month` or `weekday` in the same formatter - that throws an error.

**On step 9.** Loop from index `1` rather than `0`, and compare each session's `at` against the one before it. There is no gap before the first session, so there is nothing to measure there. Divide the difference by `DAY_IN_MS` and round down with `Math.floor`, because a gap of 1.8 days is one whole day and a bit.

**On step 10.** `if (saved && saved.charAt(0) === '[')`. Blunt, as advertised: it catches the obvious rubbish and would happily accept `[nonsense`. Note where it fails, because that gap is what `try...catch` is for.

</details>

---

## What to take away

Three things, in the order they will save you time.

**Storage only holds text.** Everything else is flattened on the way in. `JSON.stringify` and `JSON.parse` are how you flatten an object without losing it.

**JSON knows six kinds of value, and a date is not one of them.** Any date you fold comes back as a string. Either store a number instead, or rebuild the date after unfolding, but know which one you are doing.

**Store the number, format at the end.** Timestamps are for comparing and measuring; formatted strings are for reading. Keep one `Intl.DateTimeFormat` in a variable and turn numbers into words in exactly one place in your code.

And the thing underneath all three, which is worth more than the specifics: when data comes back different from how it went in, go and look at what is actually stored. The Application tab would have told you about that ISO string in four seconds. Most storage bugs are not really bugs. They are a mismatch between what you think is on the shelf and what is on the shelf.
