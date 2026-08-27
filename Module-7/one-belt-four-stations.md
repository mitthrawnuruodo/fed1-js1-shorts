# Extra lesson: One belt, four stations

Module 7 gives you four things that all act on the same list: search, sort, pagination and rendering. Each one is explained well on its own. What the module says only once, in a note near the end, is the thing that actually decides whether your project works:

> Filter, then sort, then paginate, then render.

That single line is the spine of the whole week. Get it right and the Module 7 Task is nearly free. Get it wrong and you get the two bugs that everybody gets: the page numbers are wrong after a search, and searching from page 4 shows an empty screen.

So this lesson is about the order, and about where the app keeps what it knows. We will build a small thing badly first, feel why it hurts, and then rebuild it.

If you did the extra lesson for 7.1 and 7.2, this is the other road. There, all four jobs happened in Chicago and you only wrote the request. Here the dataset is small enough to hold, so all four come back to you. You gain sorting, which the museum would not do for you. You also gain every way of getting the four jobs in the wrong order, which is what the rest of this lesson is about.

## The three sentences

Everything below is these three sentences worked out in code.

1. **Every station takes a list and hands back a list.**
2. **Only the last station is allowed to touch the page.**
3. **The screen is a photograph of the state, not a diary of what happened.**

The picture to hold on to is a conveyor belt in a sorting room. The full crate of records goes on at one end. It passes stations, each of which takes the pile it is handed and passes on a pile. At the far end there is a window, and whatever is on the belt at that moment is what people outside can see.

## Part 1: The tangle

Make a folder with `index.html` and `script.js`.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Sandvik Bird Group</title>
  </head>
  <body>
    <h1>Sandvik Bird Group logbook</h1>

    <label for="search">Search:</label>
    <input type="search" id="search" placeholder="e.g. tit, finches" />

    <label for="order">Order by:</label>
    <select id="order">
      <option value="name">Name</option>
      <option value="sightings">Sightings</option>
      <option value="first">First seen</option>
    </select>

    <button id="next">Next page</button>

    <div id="results"></div>

    <script src="script.js"></script>
  </body>
</html>
```

The data goes at the top of `script.js`. It is a made-up logbook from a made-up local bird group, so do not take the numbers to a pub quiz.

```js
const birds = [
  { name: 'Fieldfare', family: 'Thrushes', firstSeen: 1998, sightings: 412, ringed: true },
  { name: 'Redwing', family: 'Thrushes', firstSeen: 2001, sightings: 388, ringed: true },
  { name: 'Blackbird', family: 'Thrushes', firstSeen: 1996, sightings: 901, ringed: true },
  { name: 'Goldcrest', family: 'Kinglets', firstSeen: 2004, sightings: 156, ringed: false },
  { name: 'Wren', family: 'Wrens', firstSeen: 1999, sightings: 274, ringed: false },
  { name: 'Dipper', family: 'Dippers', firstSeen: 2007, sightings: 63, ringed: true },
  { name: 'Siskin', family: 'Finches', firstSeen: 2002, sightings: 340, ringed: true },
  { name: 'Brambling', family: 'Finches', firstSeen: 1997, sightings: 512, ringed: true },
  { name: 'Bullfinch', family: 'Finches', firstSeen: 2010, sightings: 128, ringed: false },
  { name: 'Nuthatch', family: 'Nuthatches', firstSeen: 2012, sightings: 77, ringed: false },
  { name: 'Treecreeper', family: 'Treecreepers', firstSeen: 2005, sightings: 91, ringed: false },
  { name: 'Great Tit', family: 'Tits', firstSeen: 1995, sightings: 1204, ringed: true },
  { name: 'Blue Tit', family: 'Tits', firstSeen: 1995, sightings: 1130, ringed: true },
  { name: 'Coal Tit', family: 'Tits', firstSeen: 2000, sightings: 486, ringed: false },
];
```

Now the obvious way to write the rest. Each control does its own job, start to finish.

```js
const searchInput = document.querySelector('#search');
const orderSelect = document.querySelector('#order');
const nextButton = document.querySelector('#next');
const results = document.querySelector('#results');

let page = 1;

function draw(list) {
  results.innerHTML = '';
  list.forEach((bird) => {
    const row = document.createElement('p');
    row.textContent = bird.name;
    results.appendChild(row);
  });
}

searchInput.addEventListener('input', () => {
  page = 1;
  const term = searchInput.value.toLowerCase().trim();
  const found = birds.filter((bird) => bird.name.toLowerCase().includes(term));
  const sorted = [...found].sort((a, b) => a.name.localeCompare(b.name));
  draw(sorted.slice(0, 5));
});

orderSelect.addEventListener('change', () => {
  const term = searchInput.value.toLowerCase().trim();
  const found = birds.filter((bird) => bird.name.toLowerCase().includes(term));
  const sorted = [...found].sort((a, b) => b.sightings - a.sightings);
  draw(sorted.slice((page - 1) * 5, (page - 1) * 5 + 5));
});

nextButton.addEventListener('click', () => {
  page = page + 1;
  const term = searchInput.value.toLowerCase().trim();
  const found = birds.filter((bird) => bird.name.toLowerCase().includes(term));
  const sorted = [...found].sort((a, b) => a.name.localeCompare(b.name));
  draw(sorted.slice((page - 1) * 5, (page - 1) * 5 + 5));
});

draw([...birds].sort((a, b) => a.name.localeCompare(b.name)).slice(0, 5));
```

Run it. It works, roughly. Search works, Next works, the dropdown works.

Now do this, and do it before reading on:

**Click Next once, then change the dropdown to Sightings.**

The list reorders by sightings, but look at which birds you get. You are on page 2, so you get birds 6 to 10 of the sightings order, not the top 5. That is arguably correct. But now go back and read the three listeners. The dropdown listener always sorts by sightings no matter what you picked. The search listener always sorts by name and always shows page 1. The Next listener always sorts by name. The same four lines of logic appear three times, each version slightly different from the others, and nothing keeps them in step.

Now the real question. **Add a "ringed birds only" checkbox.** Do not write it yet, just count. How many places do you have to edit?

<details>
<summary>How many edits</summary>

Three listeners, plus the initial `draw` call at the bottom, so four places. And you have to remember to reset `page` in the new checkbox listener, which two of the three existing listeners forget to do anyway.

If you did write it, you almost certainly ended up with a fifth copy of the filter-sort-slice sequence.

</details>

That is the shape of the problem. It is not that the code is wrong. It is that there is no single place where "what should be on screen right now" is worked out.

## Part 2: Every station takes a list and hands back a list

Delete everything below the data except the element selectors. We are going to build the belt.

A station is a function with a strict contract: it takes an array and returns a **new** array. It does not touch the page. It does not read anything global. Hand it the same list twice and it gives you the same answer twice.

Write the first one yourself before looking. It should keep only the birds whose name **or** family contains the search term, and it should hand back everything unchanged when the term is empty.

```js
/**
 * Keeps only the birds whose name or family contains the search term.
 * @param {Array<Object>} list The list coming down the belt.
 * @param {string} term The search term, already lower case and trimmed.
 * @returns {Array<Object>} A new, shorter list.
 */
function bySearch(list, term) {
  // your code here
}
```

<details>
<summary>Solution</summary>

```js
function bySearch(list, term) {
  if (term === '') {
    return list;
  }

  return list.filter((bird) => {
    const name = bird.name.toLowerCase();
    const family = bird.family.toLowerCase();
    return name.includes(term) || family.includes(term);
  });
}
```

The early return for an empty term matters. Without it you would still get the right answer, because every string contains `''`, but writing the guard makes the intention obvious: no search means no narrowing.

</details>

Here are the other two stations.

```js
/**
 * Puts the birds in the requested order.
 * @param {Array<Object>} list The list coming down the belt.
 * @param {string} order One of 'name', 'sightings' or 'first'.
 * @returns {Array<Object>} A new, sorted list.
 */
function inOrder(list, order) {
  const copy = [...list];

  if (order === 'name') {
    copy.sort((a, b) => a.name.localeCompare(b.name));
  }

  if (order === 'sightings') {
    copy.sort((a, b) => b.sightings - a.sightings);
  }

  if (order === 'first') {
    copy.sort((a, b) => a.firstSeen - b.firstSeen);
  }

  return copy;
}

/**
 * Takes one page worth of birds off the belt.
 * @param {Array<Object>} list The list coming down the belt.
 * @param {number} page The page number, counting from 1.
 * @param {number} perPage How many birds fit on a page.
 * @returns {Array<Object>} A new list, no longer than perPage.
 */
function onePage(list, page, perPage) {
  const start = (page - 1) * perPage;
  return list.slice(start, start + perPage);
}
```

`inOrder` copies with `[...list]` before sorting because `sort` changes the array it is given. The module warns you about this once. It is worth saying again, because the bug it causes is horrible: your "original" list quietly stops being in its original order, and every later sort starts from wherever the last one left off.

<details>
<summary>Rabbit hole: why "a new list" is the whole trick</summary>

These three functions have a property with a name: they are **pure**. Same input, same output, no side effects.

Pure functions are the reason the belt works. Because `bySearch` cannot change `birds`, you can run it a hundred times without the master list drifting. Because it cannot touch the page, you can call it from anywhere without accidentally redrawing something. And because it only depends on its two arguments, you can test it in the console with a made-up array and know the answer is trustworthy:

```js
console.log(bySearch([{ name: 'Rook', family: 'Crows' }], 'crow'));
```

That is not possible with the tangled version, where the filtering logic is welded to an event listener and can only be run by typing into a box.

</details>

## Part 3: One place that runs the belt

Now the whole point of the exercise. One function, and it is nearly all you will ever need to read to understand the app:

```js
function update() {
  const found = bySearch(birds, state.term);
  const sorted = inOrder(found, state.order);
  const visible = onePage(sorted, state.page, state.perPage);

  drawBirds(visible);
  drawPager(sorted.length);
}
```

Look at the last line for a moment, because it is where the module's second classic bug lives.

`drawPager` is handed `sorted.length`, **not** `visible.length`. By the time the belt reaches `onePage`, the information the pager needs is gone: five birds on the belt tells you nothing about whether there are 5 or 500 behind them. The page count has to be read at the last station where the full filtered list still exists.

This is also why the order of the stations is not a matter of taste. You cannot paginate first, because until you have filtered and sorted you do not yet know which birds are on page 1.

## Part 4: The screen is a photograph of the state

`update()` reads from something called `state`. That is the second half of the idea.

In the tangled version the app's knowledge was scattered. `page` was a variable. The search term lived in the input box. The sort order lived in the dropdown. Three places, and no way to see all of it at once.

Instead, put everything the app knows in one object:

```js
// --- STATE ---

const state = {
  term: '',
  order: 'name',
  page: 1,
  perPage: 5,
};
```

Now every listener does exactly two things: change the state, then run the belt.

```js
searchInput.addEventListener('input', () => {
  state.term = searchInput.value.toLowerCase().trim();
  state.page = 1;
  update();
});

orderSelect.addEventListener('change', () => {
  state.order = orderSelect.value;
  update();
});

pager.addEventListener('click', (event) => {
  if (event.target.tagName === 'BUTTON') {
    state.page = Number(event.target.dataset.page);
    update();
  }
});
```

`state.page = 1` in the search listener is the first classic bug, fixed. If someone is on page 3 and types a search that matches three birds, page 3 of three birds is nothing at all, and they get "No birds match" for a search that clearly matched. Resetting the page is not a special case to remember. It is part of what "a new search" means.

And the windows. Two functions, and they are the only ones in the file allowed to touch the page:

```js
/**
 * Draws the birds that are currently visible.
 * @param {Array<Object>} list The birds to show.
 */
function drawBirds(list) {
  results.innerHTML = '';

  if (list.length === 0) {
    results.textContent = 'No birds match that search.';
    return;
  }

  list.forEach((bird) => {
    const row = document.createElement('p');
    row.textContent = `${bird.name} (${bird.family}) - first seen ${bird.firstSeen}, ${bird.sightings} sightings`;
    results.appendChild(row);
  });
}

/**
 * Draws one button per page.
 * @param {number} total How many birds there are before the page slice.
 */
function drawPager(total) {
  const pages = Math.ceil(total / state.perPage);
  pager.innerHTML = '';

  for (let i = 1; i <= pages; i++) {
    const button = document.createElement('button');
    button.textContent = i;
    button.dataset.page = i;

    if (i === state.page) {
      button.classList.add('active');
    }

    pager.appendChild(button);
  }
}
```

Notice that neither of them decides anything. They are told what to draw and they draw it. That is sentence three: the screen is a photograph of the state. Nothing is patched, nothing is nudged, nothing is toggled. Every redraw starts from scratch and produces exactly the picture the state describes.

Swap the `#next` button in your HTML for a pager container, and add the missing selector:

```html
    <div id="results"></div>
    <div id="pager"></div>
```

```js
const pager = document.querySelector('#pager');
```

Then call `update()` once at the bottom of the file to draw the first picture. That is the whole app.

## Part 5: The payoff

Back to the feature that cost four edits before. Add a checkbox to the HTML:

```html
    <label for="ringed">
      <input type="checkbox" id="ringed" /> Ringed birds only
    </label>
```

Now, three additions. One field in the state:

```js
  ringedOnly: false,
```

One new station:

```js
/**
 * Keeps only the ringed birds, if the visitor asked for that.
 * @param {Array<Object>} list The list coming down the belt.
 * @param {boolean} only Whether to keep only ringed birds.
 * @returns {Array<Object>} A new list.
 */
function byRinged(list, only) {
  if (only === false) {
    return list;
  }

  return list.filter((bird) => bird.ringed);
}
```

One extra line in the belt, plus the selector and the listener:

```js
const ringedBox = document.querySelector('#ringed');

function update() {
  const found = bySearch(birds, state.term);
  const ringed = byRinged(found, state.ringedOnly);
  const sorted = inOrder(ringed, state.order);
  const visible = onePage(sorted, state.page, state.perPage);

  drawBirds(visible);
  drawPager(sorted.length);
}

ringedBox.addEventListener('change', () => {
  state.ringedOnly = ringedBox.checked;
  state.page = 1;
  update();
});
```

Nothing else changed. Not `drawBirds`, not `drawPager`, not the search listener, not the pager. Sorting still works on the filtered list, pagination still counts the right number of pages, and the search still combines with it correctly, because all of that was decided once in `update()` and none of it had to be decided again.

That is what you have bought. New features arrive as one state field, one station and one listener, no matter how many features are already there.

## Exercises

### Exercise 1: Build a station

Add a family filter. Put this dropdown in your HTML above the results:

```html
    <label for="family">Family:</label>
    <select id="family">
      <option value="">All families</option>
      <option value="Thrushes">Thrushes</option>
      <option value="Finches">Finches</option>
      <option value="Tits">Tits</option>
      <option value="Kinglets">Kinglets</option>
    </select>
```

Then do the three additions: a `family` field in the state, a `byFamily` station, and a listener. Slot the station into `update()` and check that it still combines correctly with search, sort and pagination.

<details>
<summary>Solution</summary>

```js
// In the state object
  family: '',
```

```js
/**
 * Keeps only the birds in the chosen family.
 * @param {Array<Object>} list The list coming down the belt.
 * @param {string} family The family name, or '' for all families.
 * @returns {Array<Object>} A new list.
 */
function byFamily(list, family) {
  if (family === '') {
    return list;
  }

  return list.filter((bird) => bird.family === family);
}
```

```js
function update() {
  const found = bySearch(birds, state.term);
  const inFamily = byFamily(found, state.family);
  const ringed = byRinged(inFamily, state.ringedOnly);
  const sorted = inOrder(ringed, state.order);
  const visible = onePage(sorted, state.page, state.perPage);

  drawBirds(visible);
  drawPager(sorted.length);
}
```

```js
const familySelect = document.querySelector('#family');

familySelect.addEventListener('change', () => {
  state.family = familySelect.value;
  state.page = 1;
  update();
});
```

Note `===` rather than `includes` here. The value comes from a dropdown you control, so it either matches a family exactly or it does not. Searching is fuzzy; choosing from a list is not.

</details>

### Exercise 2: Break the belt on purpose

Three sabotages. For each one, **write down what you think will happen before you run it**, then make the change, then put it back. Guessing wrong here is more useful than guessing right.

1. In `update()`, change `drawPager(sorted.length)` to `drawPager(visible.length)`.
2. In `update()`, swap the last two stations so pagination happens before sorting.
3. In the search listener, delete the line `state.page = 1;`.

<details>
<summary>What actually happens, and why</summary>

**Sabotage 1.** With 14 birds and 5 per page you should get 3 page buttons. You get 1. `visible.length` is at most 5, and `Math.ceil(5 / 5)` is 1, so the pager insists there is only ever one page and you can never leave it. If your filtered list happened to have exactly 3 items you would get 1 page, which looks right, which is what makes this bug so annoying to find.

This is the "page numbers are wrong after a search" bug in its purest form.

**Sabotage 2.** Sorting by name, page 1, you should see: Blackbird, Blue Tit, Brambling, Bullfinch, Coal Tit. Instead you see: Blackbird, Fieldfare, Goldcrest, Redwing, Wren.

Those are the first five birds in the original array, alphabetised among themselves. You have sorted the page instead of sorting the list. Every page is internally tidy and the sequence as a whole is nonsense. Sorting has to see everything, so it has to run before anything throws most of the data away.

This is exactly the problem that made sorting impossible against the museum API in the 7.1 and 7.2 extra lesson. There you were handed twelve records out of 132,681 and could not sort them meaningfully, because twelve records is a page. The difference is that there it was imposed on you and here you did it to yourself. The rule is the same either way: you cannot sort what you do not hold.

**Sabotage 3.** Click to page 3, then search for "tit". Three birds match, but you are still on page 3, so `onePage` slices from index 10 of a three-item list and returns an empty array, and `drawBirds` reports "No birds match that search."

The message is a lie, and it is a lie that will send you looking at `bySearch`, which is not broken. That is worth remembering: when the belt is wrong, the station that reports the problem is usually not the station that caused it.

</details>

### Exercise 3: Widen the search

At the moment `bySearch` looks at the name and the family. Make it match the year of the first sighting too, so that typing `199` finds every bird first recorded in the 1990s.

The wrinkle: `firstSeen` is a number, and `includes` is a string method.

<details>
<summary>Solution</summary>

```js
function bySearch(list, term) {
  if (term === '') {
    return list;
  }

  return list.filter((bird) => {
    const name = bird.name.toLowerCase();
    const family = bird.family.toLowerCase();
    const year = String(bird.firstSeen);
    return name.includes(term) || family.includes(term) || year.includes(term);
  });
}
```

`String(bird.firstSeen)` turns `1998` into `'1998'` so `includes` has something to work with. No `toLowerCase` needed, since digits have no case.

Checks: `199` gives Fieldfare, Blackbird, Wren, Brambling, Great Tit and Blue Tit. `1995` gives Great Tit and Blue Tit. `200` gives the six birds first seen between 2000 and 2009.

Nothing outside this function changed. That is the point of the contract: a station can be rewritten freely as long as it still takes a list and hands back a list.

</details>

## Self study task

Extend the logbook with the things a real one would need. Each level stands on its own, so stop wherever you like.

### Level 1: Tell the visitor what they are looking at

Add a line above the results reading something like `Showing 5 of 14 records`. It must reflect the filters, so searching for "tit" should say `Showing 3 of 3 records` and not mention 14.

Do it without adding any logic to `drawBirds`.

<details>
<summary>Solution</summary>

```html
    <p id="count"></p>
```

```js
const count = document.querySelector('#count');

/**
 * Reports how many records are on screen out of how many matched.
 * @param {number} shown How many birds are on the current page.
 * @param {number} total How many birds matched the filters.
 */
function drawCount(shown, total) {
  if (total === 0) {
    count.textContent = 'No records match.';
    return;
  }

  count.textContent = `Showing ${shown} of ${total} records`;
}
```

```js
function update() {
  const found = bySearch(birds, state.term);
  const ringed = byRinged(found, state.ringedOnly);
  const sorted = inOrder(ringed, state.order);
  const visible = onePage(sorted, state.page, state.perPage);

  drawBirds(visible);
  drawPager(sorted.length);
  drawCount(visible.length, sorted.length);
}
```

A third window onto the same belt. It needs both figures, and both are available in `update()` at the moment it is called, which is exactly why the belt is written as a sequence of named variables rather than one long chain.

This version says "Showing 1 of 1 records", which is not English. Fixing that is a nice small job if you want it, and it belongs inside `drawCount` and nowhere else.

</details>

### Level 2: Remember the settings

Make the search term, the order and the checkbox survive a refresh, using `localStorage`. Do not save `page`: someone coming back tomorrow should start at the top.

Be careful about the first ever visit, when there is nothing saved, and about the case where what is saved is not valid JSON.

<details>
<summary>Solution</summary>

```js
/**
 * Saves the parts of the state that should outlive a refresh.
 */
function saveState() {
  localStorage.setItem('birdBelt', JSON.stringify(state));
}

/**
 * Restores saved settings into the state, if there are any.
 */
function loadState() {
  const saved = localStorage.getItem('birdBelt');

  if (saved === null) {
    return;
  }

  try {
    const parsed = JSON.parse(saved);
    state.term = parsed.term;
    state.order = parsed.order;
    state.ringedOnly = parsed.ringedOnly;
  } catch (error) {
    console.log('Could not read the saved settings, starting fresh.');
  }
}
```

Call `saveState()` at the end of `update()`, so anything that changes the state gets saved without you having to remember it in each listener.

Then at the bottom of the file, before the first `update()`, restore the settings and put them back into the controls:

```js
loadState();

searchInput.value = state.term;
orderSelect.value = state.order;
ringedBox.checked = state.ringedOnly;

update();
```

Those three lines matter. The state and the controls have to agree when the page loads, or you will see a filtered list with an empty search box.

Test the failure path properly. Open the console and run `localStorage.setItem('birdBelt', 'not json at all')`, then refresh. You should see your message and a working app with default settings, not a blank page. Cause the error yourself; do not take my word for it.

</details>

### Level 3: A second window onto the same belt

Add a small summary panel that reports, **for the filtered set rather than the visible page**, the earliest year in the records and which bird has the most sightings. Searching "finches" should summarise the finches, not all fourteen birds.

Use a plain loop. You have not met `reduce` yet and you do not need it.

<details>
<summary>Solution</summary>

```js
/**
 * Describes the filtered set of birds in one sentence.
 * @param {Array<Object>} list The birds that matched the filters.
 * @returns {string} A sentence for the summary panel.
 */
function summarise(list) {
  if (list.length === 0) {
    return 'Nothing to summarise.';
  }

  let earliest = list[0].firstSeen;
  let busiest = list[0];

  list.forEach((bird) => {
    if (bird.firstSeen < earliest) {
      earliest = bird.firstSeen;
    }

    if (bird.sightings > busiest.sightings) {
      busiest = bird;
    }
  });

  return `Earliest record: ${earliest}. Most sightings: ${busiest.name} (${busiest.sightings}).`;
}
```

```html
    <p id="summary"></p>
```

```js
const summary = document.querySelector('#summary');

// At the end of update()
  summary.textContent = summarise(sorted);
```

Seeding `earliest` and `busiest` from `list[0]` is why the empty check comes first. There is no sensible earliest year in an empty list, and `list[0].firstSeen` on an empty array would throw.

Pass `sorted`, not `visible`. The summary describes what the visitor searched for, not the five records that happen to be on screen. Three windows now read from the belt at three different points, and none of them knows the others exist.

</details>

<details>
<summary>Rabbit hole: is this not a lot of ceremony for fourteen birds?</summary>

Yes, honestly. For fourteen birds and two controls, the tangled version in Part 1 is shorter and you would not go to prison for shipping it.

The belt earns its keep at the point where features start combining. Two controls give you one pair to keep in step. Five controls give you ten pairs. The tangled version's cost grows with the number of pairs; the belt's cost grows with the number of controls, one station each, and the pairs take care of themselves because there is only one place where the combining happens.

Which is a long way of saying: the structure is not there to make this app good. It is there so the app is still workable at feature seven. Your Module 7 Task is feature five.

</details>

<details>
<summary>Rabbit hole: what this has to do with React</summary>

If you carry on to the later courses, you will meet a library that is built almost entirely on the three sentences above. It gives you a place to keep state, it insists that drawing is a function of that state, and it redraws from scratch whenever the state changes.

The main thing it adds is doing the redrawing efficiently, so that `results.innerHTML = ''` followed by rebuilding everything does not actually rebuild everything. That is a real problem worth solving, but it is a performance problem, not an idea. The idea is the part you have just written by hand, and having written it by hand is worth a great deal when the framework version starts doing things you did not ask for.

</details>
