# Look at the DOM, then draw it from your data

**Estimated time:** about 2 hours, plus the self study task at the end  
**You will need:** variables, `if`, `for...of`, arrays, objects, functions, and `filter` and `find` from Module 3. Nothing else.  
**Not in this lesson:** events. There is no `addEventListener` anywhere below. That is Lesson 4.3, and everything here is meant to make that lesson easier.

## How this lesson is different

The regular lessons hand you the DOM one method at a time: here is `getElementById`, here is `setAttribute`, here is `appendChild`. Each gets its own small page and its own small exercise. That is a sensible way to meet the tools, and you should still do those lessons.

This one is built around two habits instead of a list of methods.

**Habit one: look at the DOM before you write code against it.** Most DOM bugs are not logic bugs. They happen because the tree in the browser is not the tree you pictured in your head. So we start by breaking that assumption on purpose, in the console, before writing anything.

**Habit two: write one function that draws your data, and call it.** Most beginner DOM code is a long list of one-off instructions: change this text, add that class, hide this div. It works for five minutes and then becomes unmanageable. There is a simpler way to think about it: keep your data in an array, write one function that draws the array on the page, and when the data changes, call that function again.

Everything else here - selecting, creating, classes, attributes - turns up in service of those two habits rather than as a topic in its own right.

One note on scope. Because we are not doing events yet, nothing on these pages reacts to clicks. Where a real page would respond to a button, you will call the function yourself from the console. That is deliberate. Next lesson, a button will call it for you and nothing else in your file will need to change.

---

## Before you start

Make one folder for the whole lesson with three files: `index.html`, `app.js` and `style.css`. You will keep editing the same three files rather than making a new folder for every section.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>DOM lab</title>
    <link rel="stylesheet" href="style.css" />
    <script src="app.js" defer></script>
  </head>
  <body>
    <h1>DOM lab</h1>
  </body>
</html>
```

Notice `defer` on the script tag. It matters more than it looks.

A plain `<script src="app.js"></script>` in the `<head>` runs **before** the browser has finished building the body. Any element you try to select will come back as `null`, and your very first line of DOM code will crash. There are two easy fixes:

1. Put the script tag at the very end of `<body>`, after all the HTML. This is what the regular lessons do.
2. Keep it in the `<head>` and add `defer`, which tells the browser to run the file once the document is fully built.

Either is fine. We will use `defer`. If you ever find that your selections return `null` for elements you can plainly see on the page, this is the first thing to check.

Open `index.html` in the browser and open the developer tools with F12 (or Ctrl+Shift+I, or Cmd+Option+I on a Mac). Keep the Console tab visible for the whole lesson.

---

## Part 1: Your HTML file is not the DOM

Almost everyone starts out believing the DOM is just their HTML file spelled differently. It is not. The HTML file is a **set of instructions for building** a DOM. The browser reads it, makes decisions about it, quietly repairs it where it is broken, and produces a tree of objects. Your JavaScript talks to that tree and only to that tree. Once the page has loaded, the file on disk no longer matters.

You can look at both:

- **Ctrl+U** (View Source) shows the file exactly as it was sent to the browser. It is a text file and it never changes.
- The **Elements** panel in the developer tools shows the DOM as it is right now. It changes as your code runs.

If those two ever disagree, the Elements panel is the one that matters, because that is what your code is working with.

### 1.1 The browser repairs your HTML, and the repairs are real

Put this inside your `<body>`:

```html
<section id="notice">
  <h2>Harbour notice</h2>
  <ul id="services">
    <li>Foot passengers</li>
    <li>Vehicles</li>
  </ul>
  <table id="times">
    <tr><td>08:15</td></tr>
    <tr><td>13:40</td></tr>
  </table>
</section>
```

That table looks perfectly reasonable. Now run this in the console:

```js
const times = document.querySelector('#times');

console.log(times.children[0].tagName);
// "TBODY"

console.log(document.querySelector('#times > tr'));
// null

console.log(document.querySelectorAll('#times tr').length);
// 2
```

There is no `<tbody>` anywhere in your file, but there is one in the DOM. The rules of HTML say a table's rows must sit inside a row group, so the browser adds one for you without saying anything.

The consequence is not just trivia. The child selector `#times > tr` now matches nothing, because `tr` is no longer a child of `table`, it is a grandchild. The descendant selector `#times tr` still works.

This is the most useful thing to take from this part. **Selectors run against the DOM, not against your file.** When a selector that obviously should work finds nothing, print the element and look at what the browser actually built:

```js
console.log(times.outerHTML);
```

`outerHTML` gives you the element and everything inside it, as HTML text. It is the quickest way to see the real tree.

### 1.2 Whitespace is part of the tree

Still in the console:

```js
const services = document.querySelector('#services');

console.log(services.children.length);    // 2
console.log(services.childNodes.length);  // 5
```

Two list items, but five nodes. The three extras are text made of the line breaks and indentation between your tags. They are genuinely part of the tree.

```js
console.log(services.firstChild);          // a text node: a newline and some spaces
console.log(services.firstElementChild);   // <li>Foot passengers</li>
```

The rule is easy to remember: **properties with "Element" in the name skip the text, the ones without it do not.**

| Includes text | Elements only |
| --- | --- |
| `childNodes` | `children` |
| `firstChild` | `firstElementChild` |
| `nextSibling` | `nextElementSibling` |
| `previousSibling` | `previousElementSibling` |

In practice you want the "Element" versions nearly every time. If you have ever used `element.nextSibling` and got back something that was not an element at all, this is why.

### 1.3 Two console tools worth knowing

```js
const services = document.querySelector('#services');

console.log(services);   // shows it as an HTML tag you can expand
console.dir(services);   // shows it as a JavaScript object, with all its properties
```

`console.log` shows the element the way the Elements panel does. `console.dir` shows the object underneath: `id`, `className`, `classList`, `textContent`, `children` and a great many more. Scrolling through that list once is a good use of five minutes, because it makes the point that an element **is just an object with properties you can set**. There is no magic in `element.id = 'main'`. It is the same assignment you have been doing since week one.

One more, which only works in the console and not in your `.js` file: `$0` is a reference to whatever element you last clicked in the Elements panel. Click an element there, switch to the Console, type `$0`, and you have it without writing a selector at all.

### Exercise 1: prove the browser rewrote your HTML

**Goal:** to see for yourself that the DOM differs from the file that produced it.

**Brief**

1. Put the `#notice` section from 1.1 into your `index.html`.
2. Before running anything, write your predictions down. Actually write them down, because guessing in your head does not count - you will adjust the guess the moment you see the answer.

   - `notice.children.length`
   - `notice.childNodes.length`
   - `services.children.length`
   - `services.childNodes.length`
   - Does `document.querySelector('#times > tr')` find anything?
   - `times.children[0].tagName`

3. Now run this in the console and compare.

```js
const notice = document.querySelector('#notice');
const services = document.querySelector('#services');
const times = document.querySelector('#times');

console.log('notice.children.length   =', notice.children.length);
console.log('notice.childNodes.length =', notice.childNodes.length);
console.log('services.children.length   =', services.children.length);
console.log('services.childNodes.length =', services.childNodes.length);
console.log('#times > tr ->', document.querySelector('#times > tr'));
console.log('times.children[0].tagName =', times.children[0].tagName);
console.log(times.outerHTML);
```

4. For every prediction you got wrong, write one sentence explaining why the browser did what it did.
5. Finally, add `<li>Bicycles` to the services list **with no closing tag**, reload, and check `services.children.length` again. Did the missing tag break the list?

<details>
<summary><strong>Solution 1</strong></summary>

```text
notice.children.length   = 3     (h2, ul, table)
notice.childNodes.length = 7     (3 elements plus 4 pieces of whitespace)
services.children.length   = 2   (two li elements)
services.childNodes.length = 5   (2 elements plus 3 pieces of whitespace)
#times > tr -> null
times.children[0].tagName = TBODY
```

Why:

- `childNodes` counts the line breaks and indentation between every pair of tags. Neatly formatted HTML always produces more nodes than it looks like it should.
- `#times > tr` finds nothing because the browser inserted a `<tbody>`, so the rows are children of that row group rather than of the table.
- Adding `<li>Bicycles` with no closing tag still gives you three list items. The browser closes an open `<li>` by itself when it meets the next `<li>` or the closing `</ul>`. Your HTML was invalid and the page worked anyway, which is exactly why invalid HTML survives so long in real projects.

The `outerHTML` looks roughly like this, with a `<tbody>` you never wrote:

```html
<table id="times">
    <tbody><tr><td>08:15</td></tr>
    <tr><td>13:40</td></tr>
  </tbody></table>
```

</details>

---

## Part 2: Finding elements

The regular lesson gives you five ways to select: `getElementById`, `getElementsByTagName`, `getElementsByClassName`, `querySelector` and `querySelectorAll`. They all work, and you should be able to read all of them in other people's code.

For writing new code, this lesson takes a narrower line: **use `querySelector` and `querySelectorAll`.** One syntax, using the CSS selectors you already know. `getElementById` is not wrong, but having one selection tool in your head is worth more than the tiny speed difference.

- `querySelector('...')` gives you the **first** matching element, or `null`.
- `querySelectorAll('...')` gives you a list of **all** matching elements, which may be empty.

### 2.1 Searching inside one element

Here is the part the regular lesson barely mentions, and it changes how you write DOM code.

`querySelector` is not only available on `document`. **Every element has it.** When you call it on an element, it only searches inside that element.

```js
const board = document.querySelector('#board');

// searches the whole page
const allNames = document.querySelectorAll('.route-name');

// searches only inside #board
const boardNames = board.querySelectorAll('.route-name');
```

Why does that matter? Because it lets you write a function that works on **one card at a time**, without giving every single piece of every single card its own id.

```js
function describeRoute(routeElement) {
  const name = routeElement.querySelector('.route-name').textContent;
  const info = routeElement.querySelector('.route-info').textContent;
  return name + ' (' + info + ')';
}
```

That function works on the first card, the fortieth card, and cards that do not exist yet. Compare it with the alternative, where every card needs `id="route-1-name"`, `id="route-2-name"` and so on. Searching inside an element is how you avoid that completely.

### 2.2 Looping over what you found

`querySelectorAll` does not give you a real array. It gives you something array-like called a `NodeList`. It has `length`, you can read `list[0]`, and you can loop over it:

```js
const routes = document.querySelectorAll('.route');

for (const route of routes) {
  console.log(route.textContent);
}

routes.forEach(function (route) {
  console.log(route.textContent);
});
```

Both of those work. What does **not** work is `map`, `filter` and `find`:

```js
routes.filter(function (route) { ... });
// TypeError: routes.filter is not a function
```

When you need those, collect what you want into a real array first, with a loop:

```js
const names = [];

for (const route of routes) {
  names.push(route.querySelector('.route-name').textContent);
}

// names is now a normal array, so everything from Module 3 works on it
console.log(names.filter(function (name) {
  return name.length > 10;
}));
```

That is the pattern for this lesson: **loop over elements, build a normal array, then use the array methods you already know.**

### 2.3 Searching upwards with `closest()`

`querySelector` searches downwards, into an element. `closest()` searches upwards. Starting from the element you call it on, it walks up through the parents and returns the first one that matches, or `null`.

```js
const heading = document.querySelector('.route-name');
const wall = heading.closest('.wall');
```

The regular lesson uses `parentElement` for this, which works but makes you count the levels: `heading.parentElement.parentElement`. That breaks the moment somebody wraps things in one more div. `closest()` says what you actually mean - "the wall section I am inside" - and does not care how many layers there are in between.

### 2.4 `null` is a normal answer

`querySelector` returns `null` when nothing matches. That is not an error, it is an answer. The error comes on the next line, when you try to use it:

```js
const missing = document.querySelector('.does-not-exist');
missing.textContent = 'hello';
// TypeError: Cannot read properties of null (setting 'textContent')
```

You do not need a check on every line. But at the top of a function that reaches into the page, one check turns a confusing crash into a clear message:

```js
function updateStatus(text) {
  const status = document.querySelector('#status');

  if (!status) {
    console.warn('No #status element on this page, nothing updated.');
    return;
  }

  status.textContent = text;
}
```

### Exercise 2: read the route board

**Goal:** to get information out of a page that already exists, without changing anything on it.

**Brief**

Put this in your `index.html`:

```html
<main id="board">
  <section class="wall" data-wall="slab">
    <h2 class="wall-name">Slab</h2>
    <article class="route" data-grade="4" data-setter="Ingrid">
      <h3 class="route-name">Morning Coffee</h3>
      <p class="route-info">Yellow holds</p>
    </article>
    <article class="route" data-grade="5" data-setter="Ola">
      <h3 class="route-name">Chalk Dust</h3>
      <p class="route-info">Blue holds</p>
    </article>
  </section>
  <section class="wall" data-wall="overhang">
    <h2 class="wall-name">Overhang</h2>
    <article class="route" data-grade="7" data-setter="Ingrid">
      <h3 class="route-name">Ceiling Fan</h3>
      <p class="route-info">Red holds</p>
    </article>
    <article class="route" data-grade="6" data-setter="Marit">
      <h3 class="route-name">Short Rope</h3>
      <p class="route-info">Green holds</p>
    </article>
    <article class="route" data-grade="8" data-setter="Ola">
      <h3 class="route-name">Night Shift</h3>
      <p class="route-info">Black holds</p>
    </article>
  </section>
</main>
```

Those `data-grade` and `data-setter` attributes are a normal HTML feature. Any attribute whose name starts with `data-` is yours to invent, and you read it with `getAttribute('data-grade')`.

In `app.js`, log the following, using only `querySelector` and `querySelectorAll`:

1. How many routes there are on the board altogether.
2. How many routes are on the overhang wall. Find the overhang section first, then search **inside it**, rather than writing a cleverer selector.
3. The names of all routes, as one comma-separated string. Build a normal array first, then use `join(', ')`.
4. The names of routes graded 6 or harder. Remember that an attribute is always a string, so you will need `Number(...)` before comparing.
5. Given only the `<h3>` for "Chalk Dust", the name of the wall it sits on. Use `closest()` to get the wall, then search inside the wall for its name.
6. The average grade of all the routes. A running total in a loop is all you need.

<details>
<summary><strong>Solution 2</strong></summary>

```js
// 1. Every route on the board
const allRoutes = document.querySelectorAll('.route');
console.log('Routes in total:', allRoutes.length);
// 5

// 2. Find the section first, then search inside it
const overhang = document.querySelector('[data-wall="overhang"]');
const overhangRoutes = overhang.querySelectorAll('.route');
console.log('Routes on the overhang:', overhangRoutes.length);
// 3

// 3. Loop over the elements and build a normal array of names.
// Inside the loop we search inside one route at a time.
const names = [];

for (const route of allRoutes) {
  const name = route.querySelector('.route-name').textContent;
  names.push(name);
}

console.log('All routes:', names.join(', '));
// Morning Coffee, Chalk Dust, Ceiling Fan, Short Rope, Night Shift

// 4. Attribute values are always strings, so convert before comparing
const hardRoutes = [];

for (const route of allRoutes) {
  const grade = Number(route.getAttribute('data-grade'));

  if (grade >= 6) {
    hardRoutes.push(route.querySelector('.route-name').textContent);
  }
}

console.log('Grade 6 and above:', hardRoutes.join(', '));
// Ceiling Fan, Short Rope, Night Shift

// 5. Start at an element inside the wall and walk upwards with closest()
const chalkDust = document.querySelectorAll('.route-name')[1];
const wall = chalkDust.closest('.wall');
console.log('Chalk Dust is on:', wall.querySelector('.wall-name').textContent);
// Slab

// 6. A running total, exactly as in Module 1
let total = 0;

for (const route of allRoutes) {
  total = total + Number(route.getAttribute('data-grade'));
}

console.log('Average grade:', total / allRoutes.length);
// 6
```

Two things worth noticing. In question 3 there are two searches happening: `document.querySelectorAll` finds the routes, and then inside the loop `route.querySelector` searches one route at a time. That combination - find the group, then search inside each one - is the shape of an enormous amount of real DOM code.

In question 4, forgetting `Number(...)` gives you a string comparison instead of a number comparison. `'10' >= '6'` is `false`, because as text "1" comes before "6". It will look like a mystery until you remember that attributes are always strings.

</details>

---

## Part 3: Build elements in a function

You have two ways to put new content on the page. You can hand the browser a string of HTML and let it read it, or you can build elements and attach them. Both turn up in real code. This part is about knowing which one you are choosing and why.

### 3.1 The string way, and where it goes wrong

```js
const route = { name: 'Ceiling Fan', info: 'Red holds' };

board.innerHTML = '<article class="route">' +
  '<h3 class="route-name">' + route.name + '</h3>' +
  '<p class="route-info">' + route.info + '</p>' +
  '</article>';
```

Short, and you can see the shape of the result. That is a real advantage, and it is why people reach for it. There are two problems.

**Problem one: your text is treated as HTML.** `innerHTML` cannot tell the difference between text and tags.

```js
const name = 'Hold & Cold <best route>';

container.innerHTML = '<h3>' + name + '</h3>';
// The browser reads <best route> as a tag it does not recognise
// and it disappears from the page.
```

Compare that with `textContent`, which never treats anything as HTML:

```js
const heading = document.createElement('h3');
heading.textContent = name;
console.log(heading.outerHTML);
// <h3>Hold &amp; Cold &lt;best route&gt;</h3>
```

The browser has escaped the awkward characters for you, and the page displays exactly what was in the variable. When the text comes from a person or from an API rather than from you, this is the difference between a working page and the security hole the regular lesson demonstrates.

**Problem two: `innerHTML` destroys what was already there.** Setting it throws away everything inside the element and builds it again. Any variable that was pointing at one of the old elements now points at something that is no longer on the page:

```js
const firstRoute = document.querySelector('.route');
board.innerHTML = board.innerHTML + '<article class="route">Another</article>';

firstRoute.classList.add('selected');
// No error. No effect either. That element is not on the page any more.
```

This produces the most annoying kind of bug: the code runs, nothing goes red, and nothing happens.

A reasonable rule of thumb: **`innerHTML` is fine for fixed markup you wrote yourself. Build elements for anything that comes from your data.**

### 3.2 A function that returns an element

Building elements is wordier, and that is why people give up on it. The fix is not to type less, it is to put the wordiness inside a function and never look at it again.

```js
function createRouteCard(route) {
  const card = document.createElement('article');
  card.classList.add('route');
  card.setAttribute('data-grade', route.grade);

  const name = document.createElement('h3');
  name.classList.add('route-name');
  name.textContent = route.name;

  const info = document.createElement('p');
  info.classList.add('route-info');
  info.textContent = route.info;

  card.append(name, info);

  return card;
}
```

One route object goes in, one finished element comes out. Nothing is added to the page inside this function, which is exactly what makes it useful: you can call it in the console and inspect the result before you trust it.

```js
const card = createRouteCard({ name: 'Ceiling Fan', info: 'Red holds', grade: 7 });
console.log(card.outerHTML);
// <article class="route" data-grade="7"><h3 class="route-name">Ceiling Fan</h3><p class="route-info">Red holds</p></article>
```

This is just a function with a parameter and a `return`, from Module 2. There is nothing DOM-specific about the idea. But it is the single most useful habit in this lesson, because now you can make one card, or a hundred, with one line each.

### 3.3 `append` is a friendlier `appendChild`

The regular lesson teaches `appendChild`, which adds one element:

```js
card.appendChild(name);
card.appendChild(info);
```

`append` does the same thing but takes as many as you like, and accepts plain text too:

```js
card.append(name, info);

const summary = document.createElement('p');
summary.append('Total: ', strongElement, ' routes');
```

Use whichever you prefer. `append` usually means fewer lines.

### 3.4 An element can only be in one place

This one catches nearly everybody. An element has exactly one parent. If you add an element somewhere and then add the same element somewhere else, it is not copied, it **moves**.

```js
const badge = document.createElement('span');
badge.textContent = 'NEW';

boxA.append(badge);
boxB.append(badge);

console.log(boxA.outerHTML);  // <div></div>                   the badge left
console.log(boxB.outerHTML);  // <div><span>NEW</span></div>
```

The practical version of this rule: **create the element inside your loop, not before it.** If you build one card above the loop and append it on every pass, you will end up with exactly one card and twenty minutes of confusion.

### 3.5 Classes and attributes

Three small things you will use constantly.

**Use `classList`, not `className`.** Assigning to `className` wipes out every class the element had:

```js
card.classList.add('route');
card.classList.remove('hidden');
```

**`classList.toggle` takes an optional second argument.** With one argument it flips the class on and off. With a second true-or-false argument it forces the class on or off:

```js
// the long way
if (route.closed) {
  card.classList.add('is-closed');
} else {
  card.classList.remove('is-closed');
}

// the same thing
card.classList.toggle('is-closed', route.closed);
```

That second form is worth learning now, because when you draw a page from data you are constantly saying "this class should be there exactly when this is true".

**Your own attributes are just attributes.**

```js
card.setAttribute('data-grade', route.grade);
console.log(card.getAttribute('data-grade'));  // "7", a string, not 7
```

Anything you read back out of an attribute is a string. Wrap it in `Number(...)` before doing sums or a `===` comparison with a number.

### Exercise 3: build a card with a function

**Goal:** to produce a nested element from an object, and check it before putting it on the page.

**Brief**

1. Write a function `createRouteCard(route)` that takes an object like

```js
{ id: 12, name: 'Ceiling Fan', info: 'Red holds', grade: 7, setter: 'Ingrid', closed: true }
```

and returns one `<article>` element containing:

   - the class `route`
   - `data-id` and `data-grade` attributes taken from the object
   - an `<h3 class="route-name">` with the route name
   - a `<p class="route-info">` with the info text
   - a `<p class="setter">` reading "Set by Ingrid"
   - the class `is-closed`, but only when `closed` is `true`. Use the two-argument form of `classList.toggle`.

2. Do **not** put it on the page yet. Log `card.outerHTML` and check it against what you expected.
3. Now make an array of three route objects, loop over it, and append a card for each one to a container `<div id="board"></div>`. Create the card inside the loop.
4. Give one of your routes the name `Hold & Cold <best route>` and check that it appears on the page exactly as written.

<details>
<summary><strong>Solution 3</strong></summary>

```js
function createRouteCard(route) {
  const card = document.createElement('article');
  card.classList.add('route');
  card.setAttribute('data-id', route.id);
  card.setAttribute('data-grade', route.grade);

  const name = document.createElement('h3');
  name.classList.add('route-name');
  name.textContent = route.name;

  const info = document.createElement('p');
  info.classList.add('route-info');
  info.textContent = route.info;

  const setter = document.createElement('p');
  setter.classList.add('setter');
  setter.textContent = 'Set by ' + route.setter;

  card.append(name, info, setter);

  // Second argument: put the class on when closed is true, take it off when false
  card.classList.toggle('is-closed', route.closed);

  return card;
}

const routes = [
  { id: 12, name: 'Ceiling Fan', info: 'Red holds', grade: 7, setter: 'Ingrid', closed: true },
  { id: 13, name: 'Hold & Cold <best route>', info: 'Blue holds', grade: 5, setter: 'Ola', closed: false },
  { id: 14, name: 'Night Shift', info: 'Black holds', grade: 8, setter: 'Ola', closed: false }
];

// Check one card before anything goes on the page
console.log(createRouteCard(routes[0]).outerHTML);

const board = document.querySelector('#board');

if (board) {
  for (const route of routes) {
    // Created inside the loop, so each card is a separate element
    const card = createRouteCard(route);
    board.append(card);
  }
}
```

The awkward name comes out correctly because `textContent` escapes it for you:

```html
<h3 class="route-name">Hold &amp; Cold &lt;best route&gt;</h3>
```

which the browser then shows as `Hold & Cold <best route>`. Had you built that card by joining strings and setting `innerHTML`, `<best route>` would have been read as an unknown tag and would have vanished.

</details>

---

## Part 4: Draw the page from your data

This is the part that matters most.

### 4.1 The rule

**Your data is the truth. The page is a picture of it.**

Everything that is true about your page lives in ordinary JavaScript variables: an array of objects, maybe a couple of extra values. When something changes, you change the data and then draw the picture again. You never keep information only in the page.

In practice that means one function:

```js
function renderBoard(departures) {
  // empty the container, then build it from the array
}
```

and one rule about calling it: after anything changes, call it again.

### 4.2 The habit this replaces

Here is the shape almost everybody writes first. It is not a strawman - you will find it in your own work from last week.

```js
total.textContent = 'Total: 240 kr';

// somewhere else, later
const text = total.textContent;               // "Total: 240 kr"
const value = Number(text.slice(7, -3));      // 240, if you are lucky
total.textContent = 'Total: ' + (value + 95) + ' kr';
```

The total is stored **in the page**, as an English sentence, and has to be picked back out of that sentence before it can be used. Change the wording from "Total" to "Sum" and the arithmetic breaks.

The data-driven version:

```js
let total = 240;

function renderTotal() {
  document.querySelector('#total').textContent = 'Total: ' + total + ' kr';
}

total = total + 95;
renderTotal();
```

The number stays a number. The sentence is built from it. Changing the wording cannot possibly break the sum, because the sum never touches the wording. Once you have seen this difference you will start noticing the first version everywhere.

### 4.3 A worked example: the departure board

A ferry terminal board. Sailings live in an array, the board shows them, and delayed or cancelled sailings are marked.

`index.html`:

```html
<main>
  <h1>Departures</h1>
  <p id="summary"></p>
  <ul id="board"></ul>
</main>
```

`style.css`:

```css
body { font-family: sans-serif; margin: 2rem; }

#board { list-style: none; padding: 0; }

.departure {
  display: flex;
  gap: 1rem;
  padding: 0.5rem 0;
  border-bottom: 1px solid #ddd;
}

.destination { flex: 1; }

.is-delayed .status { color: #b45309; font-weight: bold; }
.is-cancelled { opacity: 0.6; }
.is-cancelled .destination { text-decoration: line-through; }
.empty { color: #666; font-style: italic; }
```

`app.js`, in four pieces.

**Piece one: the data.** All of it, in one place, at the top of the file.

```js
const departures = [
  { id: 'K14', time: '08:15', destination: 'Hirtshals', quay: 1, delayMinutes: 25, cancelled: false },
  { id: 'K28', time: '11:00', destination: 'Hanstholm', quay: 1, delayMinutes: 0,  cancelled: false },
  { id: 'K21', time: '13:40', destination: 'Hirtshals', quay: 2, delayMinutes: 0,  cancelled: false },
  { id: 'K33', time: '16:05', destination: 'Hanstholm', quay: 2, delayMinutes: 0,  cancelled: true }
];
```

**Piece two: a small function that knows nothing about the page.**

```js
function statusText(departure) {
  if (departure.cancelled) {
    return 'Cancelled';
  }

  if (departure.delayMinutes > 0) {
    return 'Delayed ' + departure.delayMinutes + ' min';
  }

  return 'On time';
}
```

Plain `if` statements on an object. Keep as much of your thinking in this shape as you can: it is easy to read, easy to test, and it has no idea a web page exists.

**Piece three: one function that turns one sailing into one element.**

```js
function createDepartureRow(departure) {
  const row = document.createElement('li');
  row.classList.add('departure');
  row.setAttribute('data-id', departure.id);

  const time = document.createElement('span');
  time.classList.add('time');
  time.textContent = departure.time;

  const destination = document.createElement('span');
  destination.classList.add('destination');
  destination.textContent = departure.destination;

  const quay = document.createElement('span');
  quay.classList.add('quay');
  quay.textContent = 'Quay ' + departure.quay;

  const status = document.createElement('span');
  status.classList.add('status');
  status.textContent = statusText(departure);

  row.append(time, destination, quay, status);

  row.classList.toggle('is-delayed', departure.delayMinutes > 0);
  row.classList.toggle('is-cancelled', departure.cancelled);

  return row;
}
```

**Piece four: the function that draws the whole list.**

```js
function renderBoard(list) {
  const board = document.querySelector('#board');

  if (!board) {
    console.warn('No #board on this page, nothing drawn.');
    return;
  }

  // 1. Empty the container so we never draw the same sailing twice
  board.innerHTML = '';

  // 2. Say something sensible when there is nothing to show.
  //    Every list needs this, and it is always forgotten first.
  if (list.length === 0) {
    const empty = document.createElement('li');
    empty.classList.add('empty');
    empty.textContent = 'No sailings to show.';
    board.append(empty);
    return;
  }

  // 3. One row per item
  for (const departure of list) {
    const row = createDepartureRow(departure);
    board.append(row);
  }
}

renderBoard(departures);
```

Three functions, each with one job: decide the status text, build one row, draw the list. If the board is wrong, you know which of the three to look at.

Note that `renderBoard` takes the array as a **parameter** rather than reaching for the `departures` variable directly. That is what lets you draw a different list without changing the function, which is the next section.

### 4.4 Changing the data and drawing again

Nothing here is wired to a button, because events are next lesson. But you can already see the whole point by changing the data and calling the function again. Type this into the console with the page open:

```js
renderBoard(departures.filter(function (departure) {
  return departure.quay === 1;
}));
```

Only the quay 1 sailings appear. Then:

```js
renderBoard([]);
```

The empty message appears. And to change a sailing:

```js
departures[2].delayMinutes = 15;
renderBoard(departures);
```

The board redraws with the new delay, the amber styling and the new status text.

Look at what you did **not** have to do there. You did not find the right row, work out whether it already had the delayed class, remove the old status text and put new text in. You changed a number and drew the board again.

That is the whole benefit, and it grows with the page. A board with twenty rows and six different things that can change still needs one drawing function, not one hundred and twenty little update routines.

Next lesson, `renderBoard(...)` gets called by a button instead of by you, and none of the code above has to change.

### 4.5 What drawing everything again costs

Emptying the container and rebuilding it is simple to think about, and simple is worth a lot. It is not free, though.

When you replace the contents of a container, the old elements are destroyed, and anything the browser was keeping in them goes too: which element the user had clicked into, what they had selected, and anything they had typed into an input inside that container.

For a list of ferry departures none of that matters. For a page with a form on it, redrawing the form while somebody is typing in it would be a rotten experience. The usual answer is to draw the tightest possible area: keep the form outside the container you empty.

This is also, more or less, the problem that React exists to solve. You write code that says "throw it away and draw it again", and underneath, the library works out which few elements actually changed and touches only those. When you meet React later in the programme, this pattern is what it will be built on.

### Exercise 4: extend the board

**Goal:** to work with a drawing function without falling back into one-off page edits.

**Brief**

Starting from the departure board above:

1. Write a function `getDeparturesForQuay(list, quay)` that returns a new array containing only the sailings from that quay. Use `.filter()`. Do not touch the DOM in this function.
2. Write a function `updateSummary(list)` that puts a sentence into `#summary` reading, for example, "4 sailings, 2 disrupted." A sailing counts as disrupted when it is cancelled or delayed.
3. Call `updateSummary` from inside `renderBoard`, so the sentence always matches what is on the board. Make sure it says something sensible when the list is empty.
4. Test everything from the console: draw quay 1 only, draw quay 9 (which does not exist), change a delay and draw again. Check the summary matches the rows every time.

**Rules:** do not read anything back out of the page. `renderBoard` should be safe to call as many times in a row as you like, always with the same result.

<details>
<summary><strong>Solution 4</strong></summary>

```js
function getDeparturesForQuay(list, quay) {
  return list.filter(function (departure) {
    return departure.quay === quay;
  });
}

function updateSummary(list) {
  const summary = document.querySelector('#summary');

  if (!summary) {
    return;
  }

  if (list.length === 0) {
    summary.textContent = 'Nothing scheduled.';
    return;
  }

  const disrupted = list.filter(function (departure) {
    return departure.cancelled || departure.delayMinutes > 0;
  });

  summary.textContent = list.length + ' sailings, ' + disrupted.length + ' disrupted.';
}

function renderBoard(list) {
  const board = document.querySelector('#board');

  if (!board) {
    console.warn('No #board on this page, nothing drawn.');
    return;
  }

  board.innerHTML = '';

  // The summary is drawn from the same array as the rows,
  // so the two can never disagree.
  updateSummary(list);

  if (list.length === 0) {
    const empty = document.createElement('li');
    empty.classList.add('empty');
    empty.textContent = 'No sailings to show.';
    board.append(empty);
    return;
  }

  for (const departure of list) {
    board.append(createDepartureRow(departure));
  }
}

renderBoard(departures);
```

Testing from the console:

```js
renderBoard(getDeparturesForQuay(departures, 1));
// 2 sailings, 1 disrupted.

renderBoard(getDeparturesForQuay(departures, 9));
// Nothing scheduled, and the "No sailings to show." row

departures[1].cancelled = true;
renderBoard(departures);
// 4 sailings, 3 disrupted.
```

The important detail is that `updateSummary` counts the same array that the rows are built from. If you had instead counted the rows on the page, or kept a separate `disruptedCount` variable that something else had to remember to update, the sentence and the list would eventually drift apart. Drawing both from one array makes that impossible.

</details>

---

## Part 5: When nothing happens

A checklist for the worst kind of bug, where your code runs, nothing goes red, and the page just sits there. Work down it in order.

**1. Is the file loading at all?** Put `console.log('app.js running');` on the first line. If you do not see it, the problem is the script tag: wrong path, wrong filename, or a typo in `src`.

**2. Did you select `null`?** Log the result of the selection. `Cannot read properties of null` tells you the line but not which lookup failed if you have several on one line.

**3. Has the page been built yet?** If a selection returns `null` for an element you can see in the Elements panel, your script ran too early. Add `defer`, or move the script tag to the end of `<body>`.

**4. Does your selector match the DOM, or only your file?** Print `container.outerHTML` and read what is really there. Remember the `<tbody>`.

**5. Is the element you are changing still on the page?** If anything set `innerHTML` in between, your variable is pointing at an element that has been thrown away. Select it again.

**6. Did you forget the unit?** `element.style.width = 200` does nothing whatsoever. It has to be `'200px'`. No error is raised, because an invalid style value is simply ignored.

**7. Did `className` eat your classes?** `element.className = 'active'` replaces every class on the element. Use `classList.add` and `classList.remove`.

**8. Did you create the element inside the loop?** If only the last item ended up with the badge, you created one badge outside the loop and moved it round and round.

**9. Is your attribute a string?** `Number(card.getAttribute('data-grade'))` before you compare or add.

**10. Is it the CSS rather than the JavaScript?** Check the Elements panel. If the class is on the element and it still looks wrong, your JavaScript worked and your stylesheet did not.

---

## Self study task: the station schedule board

**Goal:** to build a complete page from an array, using only what is in this lesson.

**Brief**

Radio Skagerrak needs a schedule page. The programmes live in an array. The page lists them, marks whichever one is on air, and says something useful at the top.

Because we have no events yet, "the current time" is simply a variable at the top of your file. You change it by hand and reload the page to see the badge move to a different programme.

Start from this data:

```js
const currentTime = '09:40';

const programmes = [
  { id: 'p1', start: '06:00', end: '09:00', title: 'Early Tide',      presenter: 'Ingvild Rom', genre: 'news',  repeat: false },
  { id: 'p2', start: '09:00', end: '10:30', title: 'The Long Player', presenter: 'Bjorn Aas',   genre: 'music', repeat: false },
  { id: 'p3', start: '10:30', end: '12:00', title: 'Coastal Kitchen', presenter: 'Sara Lunde',  genre: 'talk',  repeat: true  },
  { id: 'p4', start: '12:00', end: '14:00', title: 'Afternoon Drift', presenter: 'Bjorn Aas',   genre: 'music', repeat: false },
  { id: 'p5', start: '14:00', end: '15:00', title: 'Harbour Report',  presenter: 'Ingvild Rom', genre: 'news',  repeat: false }
];
```

The times are written as `'09:40'` on purpose. Because they always have two digits for the hour and use the 24 hour clock, you can compare them with `<` and `>` just like numbers: `'09:00' < '13:40'` is `true`. That trick only works while the format stays exactly like this.

**Level 1 process**

1. Set up a folder with `index.html`, `style.css` and `app.js`, with `defer` on the script tag. The HTML holds only the empty shell: a heading, an empty `<p id="clock">`, an empty `<p id="schedule-summary">` and an empty `<ul id="schedule">`. Everything else is built by your JavaScript.
2. Write `isOnAir(programme, now)` that returns `true` when `now` is at or after the start time and before the end time. No DOM code in this function.
3. Write `createProgrammeItem(programme, now)` that returns one `<li>` containing:
   - the class `programme` and a `data-id` attribute
   - a `<p class="badge">On air now</p>` as the **first** thing inside, but only when the programme is on air
   - a `<p class="slot">` reading "09:00 to 10:30"
   - an `<h2 class="title">` with the title
   - a `<p class="presenter">` reading "With Bjorn Aas"
   - a `<span class="genre">` with the genre, and a `data-genre` attribute
   - a `<p class="repeat">Repeat</p>`, but only for repeats
   - the class `is-on-air` when it is on air and `is-finished` when it has already ended, both set with the two-argument `classList.toggle`
4. Write `renderSchedule(list, now)` that:
   - checks the containers exist before using them
   - puts "Station time: 09:40" into `#clock`
   - empties `#schedule` and builds one item per programme
   - shows a single "Nothing scheduled." item when the array is empty
   - puts a sentence into `#schedule-summary` naming what is on air and how many programmes are still to come
5. Call `renderSchedule(programmes, currentTime)` at the bottom of the file.
6. Write the CSS so the on-air programme is obvious and finished programmes are dimmed. Your JavaScript must not set any colours - it only sets classes.

**Level 2 process**

7. Change `currentTime` to `'13:05'` and reload. The badge should move to Afternoon Drift, and three programmes should now be dimmed.
8. From the console, draw only the music programmes:

```js
renderSchedule(programmes.filter(function (p) { return p.genre === 'music'; }), '13:05');
```

Check that the summary sentence still agrees with what is on screen.

9. Draw an empty array and confirm the "Nothing scheduled." message appears.

**Level 3 process**

10. Give one programme a title containing an ampersand and an angle bracket, and confirm it appears on the page exactly as written.
11. Write two or three sentences answering this: if there were a search box inside `#schedule`, what would go wrong every time you called `renderSchedule`, and where would you move it to?

<details>
<summary><strong>Solution: self study task</strong></summary>

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Radio Skagerrak schedule</title>
    <link rel="stylesheet" href="style.css" />
    <script src="app.js" defer></script>
  </head>
  <body>
    <main>
      <h1>Radio Skagerrak</h1>
      <p id="clock"></p>
      <p id="schedule-summary"></p>
      <ul id="schedule"></ul>
    </main>
  </body>
</html>
```

`style.css`:

```css
body { font-family: sans-serif; margin: 2rem; max-width: 40rem; }

#schedule { list-style: none; padding: 0; }

.programme {
  border-left: 4px solid #ddd;
  padding: 0.5rem 1rem;
  margin-bottom: 0.75rem;
}

.programme .title { margin: 0.25rem 0; font-size: 1.1rem; }
.slot, .presenter { margin: 0; color: #555; font-size: 0.9rem; }

.genre {
  display: inline-block;
  margin-top: 0.5rem;
  font-size: 0.75rem;
  background: #eee;
  border-radius: 1rem;
  padding: 0.1rem 0.6rem;
}

.badge {
  margin: 0;
  font-size: 0.75rem;
  font-weight: bold;
  text-transform: uppercase;
  color: #b91c1c;
}

.repeat { margin: 0.25rem 0 0; font-size: 0.8rem; font-style: italic; color: #666; }

.is-on-air { border-left-color: #b91c1c; background: #fff5f5; }
.is-finished { opacity: 0.5; }
.empty { color: #666; font-style: italic; }
```

`app.js`:

```js
// -----------------------------------------------------------
// The data. Everything true about this page lives up here.
// -----------------------------------------------------------

const currentTime = '09:40';

const programmes = [
  { id: 'p1', start: '06:00', end: '09:00', title: 'Early Tide',      presenter: 'Ingvild Rom', genre: 'news',  repeat: false },
  { id: 'p2', start: '09:00', end: '10:30', title: 'The Long Player', presenter: 'Bjorn Aas',   genre: 'music', repeat: false },
  { id: 'p3', start: '10:30', end: '12:00', title: 'Coastal Kitchen', presenter: 'Sara Lunde',  genre: 'talk',  repeat: true  },
  { id: 'p4', start: '12:00', end: '14:00', title: 'Afternoon Drift', presenter: 'Bjorn Aas',   genre: 'music', repeat: false },
  { id: 'p5', start: '14:00', end: '15:00', title: 'Harbour Report',  presenter: 'Ingvild Rom', genre: 'news',  repeat: false }
];

// -----------------------------------------------------------
// A function that knows nothing about the page.
// The times are two-digit 24 hour strings, so < and > work on
// them just as they would on numbers.
// -----------------------------------------------------------

function isOnAir(programme, now) {
  return programme.start <= now && now < programme.end;
}

// -----------------------------------------------------------
// One programme in, one element out. Nothing is added to the
// page in here, so we can log the result and check it first.
// -----------------------------------------------------------

function createProgrammeItem(programme, now) {
  const onAir = isOnAir(programme, now);

  const item = document.createElement('li');
  item.classList.add('programme');
  item.setAttribute('data-id', programme.id);

  // Appended first, so it appears at the top of the item
  if (onAir) {
    const badge = document.createElement('p');
    badge.classList.add('badge');
    badge.textContent = 'On air now';
    item.append(badge);
  }

  const slot = document.createElement('p');
  slot.classList.add('slot');
  slot.textContent = programme.start + ' to ' + programme.end;

  const title = document.createElement('h2');
  title.classList.add('title');
  title.textContent = programme.title;

  const presenter = document.createElement('p');
  presenter.classList.add('presenter');
  presenter.textContent = 'With ' + programme.presenter;

  const genre = document.createElement('span');
  genre.classList.add('genre');
  genre.setAttribute('data-genre', programme.genre);
  genre.textContent = programme.genre;

  item.append(slot, title, presenter, genre);

  if (programme.repeat) {
    const repeat = document.createElement('p');
    repeat.classList.add('repeat');
    repeat.textContent = 'Repeat';
    item.append(repeat);
  }

  // Put the class on when the condition is true, take it off when it is false
  item.classList.toggle('is-on-air', onAir);
  item.classList.toggle('is-finished', programme.end <= now);

  return item;
}

// -----------------------------------------------------------
// Draw the whole page from the array.
// -----------------------------------------------------------

function renderSchedule(list, now) {
  const schedule = document.querySelector('#schedule');
  const summary = document.querySelector('#schedule-summary');
  const clock = document.querySelector('#clock');

  if (!schedule || !summary || !clock) {
    console.warn('The schedule markup is missing, nothing drawn.');
    return;
  }

  clock.textContent = 'Station time: ' + now;

  schedule.innerHTML = '';

  if (list.length === 0) {
    const empty = document.createElement('li');
    empty.classList.add('empty');
    empty.textContent = 'Nothing scheduled.';
    schedule.append(empty);
    summary.textContent = 'No programmes listed.';
    return;
  }

  for (const programme of list) {
    schedule.append(createProgrammeItem(programme, now));
  }

  // The summary is worked out from the same array as the list,
  // so the two can never disagree.
  const current = list.find(function (programme) {
    return isOnAir(programme, now);
  });

  const upcoming = list.filter(function (programme) {
    return programme.start > now;
  });

  let sentence = list.length + ' programmes listed. ';

  if (current) {
    sentence = sentence + 'On air: ' + current.title + '. ';
  } else {
    sentence = sentence + 'Off air just now. ';
  }

  summary.textContent = sentence + upcoming.length + ' still to come.';
}

renderSchedule(programmes, currentTime);
```

At `'09:40'` the page reads:

```text
Station time: 09:40
5 programmes listed. On air: The Long Player. 3 still to come.
```

Early Tide has the class `programme is-finished`, The Long Player has `programme is-on-air` with the badge at the top, and the other three have only `programme`.

Change `currentTime` to `'13:05'` and reload, and the badge moves to Afternoon Drift while the first three dim.

Drawing only the music programmes from the console gives:

```text
2 programmes listed. On air: Afternoon Drift. 0 still to come.
```

**Level 3, question 11.** A search box inside `#schedule` would be destroyed every time `renderSchedule` ran, because the function empties that container before building it again. Whatever the user had typed would disappear and the box would lose focus. The fix is to keep the box outside `#schedule`, somewhere the drawing function never touches, so that redrawing the list cannot affect it.

</details>

---

## What you should be able to say afterwards

If the lesson worked, these should feel obvious rather than clever:

- The DOM is a tree of objects the browser built from your file, and it is not always the tree you wrote.
- `console.dir` on an element shows you it is just an object with properties.
- `querySelector` works on any element, not only on `document`, and searching inside one element is how you avoid giving everything an id.
- A `NodeList` is not a real array. Loop over it and build one if you need `filter` or `map`.
- Attributes are always strings.
- `textContent` escapes your text for you. Joining strings and using `innerHTML` makes escaping your problem.
- An element has one parent, so appending it somewhere else moves it. Create it inside the loop.
- A function that takes one object and returns one element is worth writing every time.
- The data is the truth and the page is a picture of it. When the data changes, draw it again.

Next lesson, events give you a reason to draw it again. Everything you have written here will still work when they do.
