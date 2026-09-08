# Waiting without freezing

**Estimated time:** about 2 hours for the core path, plus 45 to 60 minutes for the self study task.  
**Prerequisites:** Modules 1 to 4. You need variables, `if`, loops, arrays, objects, functions, and enough DOM to select an element, create one, and listen for a click.

## How this lesson is different

The regular lessons walk up a ladder of techniques: callbacks, then callback hell, then promises, then `.then`/`.catch`/`.finally`, then `async`/`await`, then `fetch`, then JSON. That is a sensible order and you should still work through those pages.

This lesson does something else. We build one small thing - a board of Studio Ghibli films - and then swap the plumbing underneath it three times. Callbacks, then a promise you write yourself, then a real network request. The point is that the code around the plumbing barely changes, and that there is nothing magic inside.

Two sentences hold the whole lesson together.

**One: your browser tab has exactly one pen.** Only one line of your JavaScript can be written at a time. `await` is the moment your function puts the pen down and lets the browser use it for something else, and then picks it up again later at that same line.

**Two: `fetch` hands you the envelope before it hands you the letter.** That is why there are two `await`s. And an envelope that says "no such film" is still an envelope that arrived safely, which is why there is one check you have to do yourself.

We will use the unofficial Studio Ghibli API. It is free, it needs no key, no sign-up and no token, so you can spend your attention on the JavaScript rather than on credentials.

---

## Before you start

Make a folder with three files.

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Film board</title>
    <link rel="stylesheet" href="style.css" />
    <script src="app.js" defer></script>
  </head>
  <body>
    <h1>Film board</h1>
    <p id="status">Nothing loaded yet.</p>
    <button id="loadButton" type="button">Load films</button>
    <div id="filmList"></div>
  </body>
</html>
```

`style.css`:

```css
body {
  font-family: sans-serif;
  max-width: 40rem;
  margin: 2rem auto;
}

.film {
  border: 1px solid #ccc;
  border-radius: 6px;
  padding: 0.5rem 1rem;
  margin-bottom: 0.75rem;
}

.film h2 {
  font-size: 1.1rem;
  margin: 0.25rem 0;
}
```

`app.js`:

```js
const statusText = document.querySelector('#status');
const loadButton = document.querySelector('#loadButton');
const filmList = document.querySelector('#filmList');

console.log('app.js is running');
```

**Open the page with Live Server, not by double-clicking the file.** In VS Code, install the Live Server extension, right-click `index.html`, choose "Open with Live Server". Your address bar should say `http://127.0.0.1:5500/...` or similar, not `file:///C:/...`.

This matters from Part 3 onwards. A page opened straight from disk has no proper address of its own, and browsers refuse to let such a page fetch data from a real server. You would get a confusing error that has nothing to do with your code. Get into the habit now, before it can bite you.

---

## Part 1: one pen

Here is the thing that makes asynchronous code necessary in the first place.

Your tab runs your JavaScript with one pen. There is no second pen. While one of your functions is writing, nothing else can write. Not another function of yours, not the browser's own work of redrawing the page, not the code that reacts to your clicks. Everything queues for the pen.

Put this at the bottom of `app.js` and try it.

```js
function waitTheSlowWay(seconds) {
  const stopAt = Date.now() + seconds * 1000;
  while (Date.now() < stopAt) {
    // Do nothing at all. Just keep asking what time it is.
  }
}

loadButton.addEventListener('click', function () {
  statusText.textContent = 'Working...';
  waitTheSlowWay(3);
  statusText.textContent = 'Done.';
});
```

`Date.now()` gives you the current time as a number of milliseconds. The `while` loop keeps checking it until three seconds have gone by. It is a deliberately stupid way to wait, and it is a very good way to see the problem.

### Exercise 1

Before you run it, write down what you expect to see on screen. Then run it and check.

1. Click the button. Does the text ever say `Working...`?
2. While the three seconds are passing, try to select the heading text with your mouse. Try to scroll. Try clicking the button again.
3. Now rewrite the handler so that "Done." appears three seconds after the click, but the page stays alive the whole time. Use `setTimeout`.

<details>
<summary>Solution</summary>

`Working...` never appears. The screen goes straight from `Nothing loaded yet.` to `Done.` after a three second pause, and during that pause the page is completely dead. You cannot select text, you cannot scroll, and your second click just gets remembered and handled afterwards.

Setting `textContent` does not draw anything on screen by itself. It changes the page in memory and leaves a note for the browser saying "this needs redrawing". The redraw needs the pen. Your handler is still holding the pen, all the way through `waitTheSlowWay`, and it only lets go when it reaches the end. By then `textContent` is already `'Done.'`, so that is what gets drawn. The first value was never on screen for even a single frame.

Here is the non-blocking version:

```js
loadButton.addEventListener('click', function () {
  statusText.textContent = 'Working...';
  setTimeout(function () {
    statusText.textContent = 'Done.';
  }, 3000);
});
```

Now `Working...` appears immediately, the page stays responsive, and `Done.` replaces it three seconds later.

The difference is that this handler finishes almost instantly. It sets the text, it asks the browser to run a function later, and then it stops and puts the pen down. The browser is free to redraw, to handle scrolling, to do whatever it likes. Keeping time is not JavaScript's job at all: `setTimeout` hands the stopwatch to the browser, which has its own machinery for timers and does not need the pen to count.

Three seconds later the browser wants your function run. It waits until the pen is free, and then runs it.

</details>

<details>
<summary>Rabbit hole: why does the whole tab freeze, and not just my script?</summary>

Because your script, the page's redrawing, and the handling of your clicks and key presses are all done by the same single worker, in the same queue. Your code is not running "alongside" the browser. It is running as one of the browser's jobs, and while it runs, the other jobs wait their turn.

This is also why a page can go grey and offer to kill the tab. The browser noticed that it has not been able to get the pen back for a long time.

There is a way to get a genuine second worker, called a Web Worker, and there are separate rules about what it is allowed to touch. That is well beyond this course. For everything you will write this year, assume one pen, and write code that gives it back quickly.

</details>

---

## Part 2: build the delivery service yourself

Before you use somebody else's asynchronous function, write one. Once you have built the machinery by hand it stops looking like magic.

Put this near the top of `app.js`, under the three `querySelector` lines. It is standing in for data that will later come over the network.

```js
const localFilms = [
  {
    title: 'Porco Rosso',
    director: 'Hayao Miyazaki',
    release_date: '1992',
    running_time: '93'
  },
  {
    title: "Kiki's Delivery Service",
    director: 'Hayao Miyazaki',
    release_date: '1989',
    running_time: '102'
  },
  {
    title: 'Spirited Away',
    director: 'Hayao Miyazaki',
    release_date: '2001',
    running_time: '124'
  }
];
```

Those property names look odd. `release_date`, not `releaseDate`. That is on purpose: they are the names the real API uses, and later in this lesson we will drop real data straight into the same code. You do not get to choose the shape of other people's data. You read it and use the names it gives you.

### The obvious approach, which does not work

Your first instinct will be to write a function that fetches and returns.

```js
function brokenGetFilms() {
  let result;
  setTimeout(function () {
    result = localFilms;
  }, 1000);
  return result;
}

console.log(brokenGetFilms()); // undefined
```

This logs `undefined`, every single time. Trace it with the pen in mind. The function sets up a timer, reaches `return result` about a microsecond later, and `result` has not been assigned yet. The function is long finished by the time the timer goes off. A value that does not exist yet cannot be returned.

Write this one out and run it. Getting `undefined` here once, on purpose, will save you an hour of confusion later.

### Version one: hand in a function

If the answer cannot come back out through `return`, it has to go somewhere else. So we hand in a function for the answer to be given to.

```js
function fetchFilmsWithCallback(whenDone) {
  setTimeout(function () {
    whenDone(localFilms);
  }, 1000);
}

fetchFilmsWithCallback(function (films) {
  console.log('Got', films.length, 'films');
});
```

That works. `whenDone` is a callback, the same idea you already used with `forEach` and with `addEventListener`, just applied to a value that arrives late.

It also has a shape you will grow to dislike. The result comes out sideways. Anything you want to do with those films has to happen inside that function, and if you then need a second thing that also takes time, it goes inside the first one, and so on inwards. The regular lesson calls the end state callback hell, and it is worth reading that page to see how deep the nesting gets.

### Version two: hand out a receipt

A `Promise` is an object that stands in for a value that has not arrived yet. Think of it as a receipt from a shop: you get it immediately, it is not the goods, and it can later turn into either the goods or an apology.

```js
function fetchFilms() {
  return new Promise(function (resolve) {
    setTimeout(function () {
      resolve(localFilms);
    }, 1000);
  });
}
```

`new Promise` takes one function, and it runs that function immediately. `resolve` is a function handed to you by the Promise. Calling `resolve(value)` is how you say "the receipt can be exchanged now, and here is what for".

Note what changed and what did not. The `setTimeout` is still there. The waiting is still done by the browser. All we have done is wrap the arrangement in an object that we can pass around and return.

Now `fetchFilms()` returns something immediately, so it can be used like a normal value:

```js
const receipt = fetchFilms();
console.log(receipt); // Promise { <pending> }
```

To get at the value, you can ask the receipt to run a function once it is exchanged:

```js
fetchFilms().then(function (films) {
  console.log('Got', films.length, 'films');
});
```

Which, you may notice, looks almost exactly like the callback version. That is fair. The real gain arrives with the next step.

### Version three: await

```js
async function showFilms() {
  const films = await fetchFilms();
  console.log('Got', films.length, 'films');
}

showFilms();
```

Two rules, and they are the whole of `async`/`await`.

**Rule one: `await` may only appear inside a function marked `async`.** It means "put the pen down here, and carry on from this exact line when the value arrives". While the function is paused, the browser has the pen back and the page stays alive.

**Rule two: a function marked `async` always returns a Promise**, whether or not you wrote one. If you `return films` from an async function, the caller does not get the array, it gets a receipt for the array. This catches everybody once.

Here is a trace worth running. Predict the order first.

```js
async function traceFilms() {
  console.log('2: inside, about to await');
  const films = await fetchFilms();
  console.log('4: films arrived,', films.length, 'of them');
}

console.log('1: before calling traceFilms');
traceFilms();
console.log('3: after calling traceFilms');
```

The numbers give it away, but sit with why. `traceFilms()` runs normally as far as the `await`. At that point it puts the pen down and hands control straight back to the line after `traceFilms()`, which is why `3` prints before the films arrive. A second later the value is ready, `traceFilms` picks the pen back up on the line it left off, and `4` prints.

That is the whole trick. An `async` function is an ordinary function that is allowed to pause in the middle.

### Exercise 2

Write `getFilmsFromShelf(shelfName)`.

1. It returns a Promise.
2. After 500 milliseconds, if `shelfName` is `'ghibli'`, it resolves with `localFilms`.
3. Otherwise it rejects with `new Error('No shelf called ' + shelfName)`.
4. Write an async function `report(shelfName)` that logs how many films were found, or logs the error message if there were none. Use `try` and `catch`.
5. Call `report('ghibli')` and `report('pixar')`.

To reject, take the second parameter of the Promise function: `new Promise(function (resolve, reject) { ... })`, then call `reject(someError)`.

<details>
<summary>Solution</summary>

```js
function getFilmsFromShelf(shelfName) {
  return new Promise(function (resolve, reject) {
    setTimeout(function () {
      if (shelfName === 'ghibli') {
        resolve(localFilms);
      } else {
        reject(new Error('No shelf called ' + shelfName));
      }
    }, 500);
  });
}

async function report(shelfName) {
  try {
    const films = await getFilmsFromShelf(shelfName);
    console.log('Found', films.length, 'films on', shelfName);
  } catch (error) {
    console.log('Problem:', error.message);
  }
}

report('ghibli'); // Found 3 films on ghibli
report('pixar'); // Problem: No shelf called pixar
```

Notice that `try`/`catch` works here in exactly the way you would expect it to work on ordinary code. That is the main reason `await` is nicer than `.then`. A rejected Promise that you are awaiting behaves like a thrown error at that line, so one `catch` can cover several awaited steps.

The `.then` spelling of the same thing puts the failure path in a separate function:

```js
getFilmsFromShelf('pixar')
  .then(function (films) {
    console.log('Found', films.length, 'films');
  })
  .catch(function (error) {
    console.log('Problem:', error.message);
  });
```

Both are correct. You will read plenty of code written in both styles, so you need to recognise both, but write new code with `async`/`await`.

One warning worth having now: if nothing catches a rejected Promise, the failure does not crash your program. It goes quiet, and appears in the console as an "uncaught (in promise)" warning that is very easy to scroll past. Every Promise needs somebody to catch it.

</details>

### Exercise 3

Predict the exact output order, then run it.

```js
function later(label, ms) {
  return new Promise(function (resolve) {
    setTimeout(function () {
      resolve(label);
    }, ms);
  });
}

async function run() {
  console.log('B');
  const first = await later('D', 100);
  console.log(first);
  const second = await later('E', 100);
  console.log(second);
}

console.log('A');
run();
console.log('C');
```

Then answer in a sentence: roughly how long does the whole thing take, and why is it not 100 milliseconds?

<details>
<summary>Solution</summary>

The output is `A`, `B`, `C`, `D`, `E`.

`console.log('A')` runs. `run()` is called and runs normally until the first `await`, printing `B`. At the `await` it puts the pen down, so control returns to the main script and `C` prints. About 100 milliseconds later the first Promise resolves, `run` resumes, prints `D`, and immediately hits the second `await` and pauses again. Another 100 milliseconds later it prints `E`.

The whole thing takes roughly 200 milliseconds, because the second `later` call does not even start until the first has finished. `await` means wait here. If step two needs the answer from step one, that is exactly what you want. If the two are independent, you have just made your app twice as slow for no reason, and there is a tool for starting both at once. Look up `Promise.all` when you meet a case that needs it. It is not needed for anything in this course, so do not go hunting for excuses to use it.

</details>

<details>
<summary>Rabbit hole: why do promise callbacks jump the queue?</summary>

Try this:

```js
console.log('start');

setTimeout(function () {
  console.log('timeout, 0 ms');
}, 0);

Promise.resolve().then(function () {
  console.log('promise');
});

console.log('end');
```

The order is `start`, `end`, `promise`, `timeout, 0 ms`. The Promise callback wins, even though the timer was asked for first and asked for zero delay.

There are two queues, not one. Timers, clicks and network events go in the ordinary task queue. Promise callbacks go in a separate microtask queue, and the browser drains the entire microtask queue before it takes even one job from the ordinary queue.

You will not need this to write anything in this course. It is here because it explains an output order that would otherwise look like a bug, and because interviewers are unreasonably fond of asking about it.

</details>

---

## Part 3: swap in the real API

Everything so far has been pretend. Time to point the same code at a real server.

### Look at the data before you write a line of code

Open this in a browser tab:

```
https://ghibliapi.vercel.app/films
```

That is it. That is an API request. You just made one, with the same method (`GET`) that your browser uses for every page it loads. The response is not a web page, it is JSON: a text format that happens to look almost exactly like JavaScript arrays and objects.

Install a JSON viewer extension if the wall of text is hard to read, or open the Network tab in DevTools, reload, and click the request to get a formatted view. Then look at one film object and find the property names. You will recognise `title`, `director`, `release_date` and `running_time` from our fake data, and you will find a good deal more besides, including `id`, `description`, `rt_score` and `image`.

Doing this first, every time you meet a new API, is the habit worth taking from this lesson. Read the real data before you write code against it. Half the bugs beginners hit with APIs are a wrong guess about a property name.

The address breaks into two parts:

- the base URL, `https://ghibliapi.vercel.app`, which is the server
- the path, `/films`, which is the thing you want from it

Change the path and you get something else: `/people`, `/locations`, `/species`, `/vehicles`. Add an id to the path and you get one single item instead of the list. Copy any film's `id` from the list and try `/films/` followed by that id. That pattern - a collection at `/things`, one item at `/things/<id>` - is the REST convention, and you will meet it on almost every API you ever touch.

### fetch

```js
const FILMS_URL = 'https://ghibliapi.vercel.app/films';

async function getFilms() {
  const response = await fetch(FILMS_URL);
  const films = await response.json();
  return films;
}
```

Two `await`s. This is the bit that trips everybody up, so here is why.

The first `await` waits for the envelope. When it finishes you have a `Response` object: the server has answered, and you can see the status code and the headers. You do not yet have the contents. On a slow connection the body may still be arriving.

The second `await` reads the contents and turns them into JavaScript. `response.json()` does two jobs: it waits for the rest of the body, and then it parses the JSON text into real arrays and objects. It is a slow job too, so it also hands back a Promise, so it also needs awaiting.

If you forget the second `await`, you will get a Promise where you expected an array, and something will complain that `films.length` is `undefined`. When that happens, look for a missing `await` first.

### The check nobody remembers

`fetch` only rejects when the request could not be made at all: no network, no such domain, request blocked. If the server answers, `fetch` is satisfied. It does not care what the server said.

So a 404 "no such thing" is, as far as `fetch` is concerned, a complete success. The envelope arrived. It just has bad news in it.

That is what `response.ok` is for. It is `true` for any status in the 200s and `false` otherwise.

```js
async function getFilms() {
  const response = await fetch(FILMS_URL);

  if (!response.ok) {
    throw new Error('The server answered with status ' + response.status);
  }

  const films = await response.json();
  return films;
}
```

Throwing here rather than returning `undefined` means the caller's `catch` handles a bad status and a dead network in the same place, with the same code.

Worth knowing the rough families of status codes:

- 200s: fine
- 300s: it moved, look over there
- 400s: your fault - 404 not found, 401 not logged in, 403 not allowed
- 500s: their fault, the server fell over

### Wiring it to the button

```js
async function loadFilms() {
  statusText.textContent = 'Loading...';
  loadButton.disabled = true;

  try {
    const films = await getFilms();
    statusText.textContent = 'Loaded ' + films.length + ' films.';
    console.log(films[0]);
  } catch (error) {
    statusText.textContent = 'Could not load the films. ' + error.message;
  } finally {
    loadButton.disabled = false;
  }
}

loadButton.addEventListener('click', loadFilms);
```

Delete or comment out the Part 1 click handler first, or you will have two handlers fighting over the same button.

Three things in there are habits rather than syntax.

**Say something before you wait.** The user gets `Loading...` the moment they click. A page that appears to do nothing for two seconds looks broken.

**Disable the button while you are working.** Otherwise an impatient user fires off five requests.

**`finally` runs either way.** Whether the try block succeeded or the catch block ran, the button gets re-enabled. Anything that must happen regardless of the outcome goes there.

One last detail: `loadFilms` is an `async` function, and we handed it to `addEventListener` like any other. That is fine. The browser calls it, it returns a Promise, and the browser ignores the Promise entirely. Which is exactly why the `try`/`catch` inside it is not optional - nobody else is going to catch anything for you.

### Exercise 4

Break it on purpose. This is the fastest way to learn what the error handling is actually for.

Open the Network tab in DevTools and keep it open. For each of the following, predict whether your `catch` block runs, or whether `!response.ok` catches it instead, and what appears on screen. Then try it, and look at what the Network tab shows.

1. Misspell the domain: `https://ghibliapi.vercel.appp/films`.
2. Keep the domain, misspell the path: `https://ghibliapi.vercel.app/filmzzz`.
3. Put the URL back, and instead set the Network tab's throttling dropdown to "Offline", then click the button.

<details>
<summary>Solution</summary>

**1 and 3 land in your `catch` block.** There is no server to answer, so `fetch` itself rejects with a `TypeError`, usually saying something unhelpful like "Failed to fetch". Browsers keep the message vague on purpose, to avoid leaking information about the user's network. The Network tab shows the request in red with no status code at all, because no response ever came back.

Your screen shows `Could not load the films. Failed to fetch`, which is not a wonderful thing to show a user. In real work you would show your own wording here, and keep the technical text for the console.

**2 goes down the `!response.ok` path**, assuming the server answers with a 404. The request was a complete success at the network level: it reached the server, and the server replied. It just replied "I have nothing at that address". The Network tab shows the request with a status code, and `response.ok` is `false`.

Look carefully at what you actually get for number 2, though, because this is a real lesson in itself. Some APIs answer a bad path with a 404 and a helpful message. Some answer with a 404 and an HTML error page, which then blows up inside `response.json()` with a parse error, and lands in your `catch` after all. Some cheerfully answer 200 with an empty object, in which case neither of your checks fires and your code sails on with nonsense data.

The general rule: a status code is what the server chose to say about itself. It is a strong hint, not a guarantee. Check the status, and where it matters, check that the data has the shape you expected too.

</details>

<details>
<summary>Rabbit hole: CORS, and why the file:// warning was serious</summary>

At some point you will get an error in the console with the word CORS in it, and it will look like your fetch is broken. It is not. Your request went out and a response came back. The browser then read the response's headers, did not find permission to hand the data to your page, and threw the data away without showing it to you.

The reason this exists: your browser holds your logged-in sessions. Without a rule like this, any page you visited could quietly fetch your webmail in the background and read the reply. So the browser only lets a page read a response from a different origin if that server explicitly says it is allowed. An origin is the combination of protocol, domain and port, so `http://127.0.0.1:5500` and `https://ghibliapi.vercel.app` are different origins.

The Ghibli API sends the header that grants that permission to everyone, which is why this lesson works at all. Many APIs do not, and are meant to be called from a server rather than from a browser.

A page opened as `file:///...` has no real origin, so servers cannot grant it permission, and the request fails before it starts. That is the whole reason for the Live Server instruction at the top.

There is nothing you can do in your own JavaScript to change any of this. The permission belongs to the server you are calling. If you hit a CORS wall on some other API, the answer is either an API key and a different endpoint, or a small server of your own that makes the call. Both are JS2 material.

</details>

---

## Part 4: keep the data, render from it

We can load films. Now put them on the page, using the DOM work from Module 4.

Add one variable near the top of `app.js`, next to the other constants:

```js
let allFilms = [];
```

And a render function. This is the same pattern as the previous lesson: empty the container, then build it from an array.

```js
function renderFilms(films) {
  filmList.innerHTML = '';

  for (const film of films) {
    const card = document.createElement('div');
    card.classList.add('film');

    const heading = document.createElement('h2');
    heading.textContent = film.title;
    card.appendChild(heading);

    const facts = document.createElement('p');
    facts.textContent =
      film.director + ' - ' + film.release_date + ' - ' + film.running_time + ' min';
    card.appendChild(facts);

    filmList.appendChild(card);
  }
}
```

Try it against the fake data first: `renderFilms(localFilms);` at the bottom of the file. It works, because we chose the fake data's property names to match the real thing.

Now update `loadFilms` to keep the data and draw it:

```js
async function loadFilms() {
  statusText.textContent = 'Loading...';
  loadButton.disabled = true;

  try {
    allFilms = await getFilms();
    renderFilms(allFilms);
    statusText.textContent = 'Loaded ' + allFilms.length + ' films.';
  } catch (error) {
    statusText.textContent = 'Could not load the films. ' + error.message;
  } finally {
    loadButton.disabled = false;
  }
}
```

That one line, `allFilms = await getFilms();`, is the second habit worth taking from this lesson.

**Fetch once. Keep the array. Render from the array.**

Every filter, search, count and sort from here on happens on `allFilms`, in memory, instantly. Going back to the network for something you already have is slow, it is rude to whoever runs the server, and it makes your app fall over when the connection is poor. Ask the network for data. Ask your variables for everything else.

### Exercise 5

Add a search box that filters the films by title as the user types.

1. Add `<input type="search" id="search" placeholder="Search titles" />` to `index.html`, just above the `filmList` div.
2. Select it in `app.js`.
3. Listen for the `input` event, which fires on every keystroke, unlike `change`.
4. In the handler, filter `allFilms` down to the films whose title contains what was typed, and pass the result to `renderFilms`. Ignore case, so that `totoro` finds `My Neighbor Totoro`.
5. Make sure that clearing the box shows everything again.

You will want `toLowerCase()` and `includes()`, both of which you met in Module 3.

<details>
<summary>Solution</summary>

```html
<input type="search" id="search" placeholder="Search titles" />
```

```js
const searchBox = document.querySelector('#search');

searchBox.addEventListener('input', function () {
  const term = searchBox.value.toLowerCase();

  const matches = allFilms.filter(function (film) {
    return film.title.toLowerCase().includes(term);
  });

  renderFilms(matches);
});
```

Clearing the box needs no special case. An empty string is contained in every string, so `includes('')` is `true` for everything and all the films come back.

Note that `allFilms` is never changed. `filter` builds a new array and leaves the original alone, which is why you can type, delete, and type something else without losing anything. If you had filtered `allFilms` and assigned the result back to `allFilms`, the first search would have thrown away every film that did not match, permanently. That is a bug that looks like magic when you meet it in the wild.

Also note what is not here: no `fetch`. Twenty-two films are already in memory. Filtering them takes microseconds.

</details>

<details>
<summary>Rabbit hole: what if the list were 20,000 films instead of 22?</summary>

Then you would not download them all, and the search would have to go to the server, which usually means a query string: `/films?search=totoro`. The bit after the `?` is a set of `name=value` pairs, and it is how you pass options to a GET request.

But firing a request on every keystroke would mean seven requests for "totoro", of which six are wasted, and the answers can come back out of order, so the list can end up showing results for "totor" after you have finished typing. The fix is to wait until the user pauses, which is called debouncing, and to ignore answers to questions you no longer care about.

None of that is needed here, and none of it is in this course. It is worth knowing that the reason we can be so casual is that the whole dataset is small, and that "just fetch everything and filter locally" stops being the right answer at some size.

</details>

---

## Self study task: the film picker

**Time:** 45 to 60 minutes. Not submitted and not assessed. This is where the lesson lands.

Extend the film board with a director filter and a detail panel. Everything you need has appeared somewhere above.

### The brief

**Part A: filter by director.**

After the films load, work out the list of directors that actually appear in the data - do not type them in by hand, because then your code would break the day the API adds a film. Build one button per director, plus an "All" button, and put them in a `<div id="directorButtons"></div>` above the search box. Clicking a director's button renders only that director's films. Clicking "All" renders all of them.

To collect the directors without duplicates: start with an empty array, loop over the films, and push each director in only if the array does not already `includes` it.

**Part B: a detail panel.**

Add a `<div id="detail"></div>` below the film list. Make each film card clickable. When a card is clicked, fetch that single film from `https://ghibliapi.vercel.app/films/` plus the film's `id`, and show its `title`, `description` and `rt_score` in the detail panel.

Yes, you already have all of that in `allFilms`, and in a real app you would just use it. Do it over the network anyway: this is the exercise where you write a second `fetch` with its own loading message and its own error handling, and one film at a time is the natural shape for that.

**Part C, if you have time: do not fetch twice.**

If the same film is clicked again, do not go back to the network. Keep an object where the keys are film ids and the values are the film objects you have already fetched, check it before fetching, and add to it after.

### Before you look at the solution

Two things that are easy to get wrong, in case you get stuck:

- The click handler goes on each card, inside the loop in `renderFilms`. Because `film` is declared with `const` in the `for...of` header, each card's handler remembers its own film. This is the one place in this task where it is worth pausing to understand why it works.
- Your detail loader is `async`, and it will be doing its own `fetch`, so it needs its own `try`/`catch`. The `catch` in `loadFilms` cannot help it.

<details>
<summary>Solution</summary>

`index.html`, body:

```html
<h1>Film board</h1>
<p id="status">Nothing loaded yet.</p>
<button id="loadButton" type="button">Load films</button>
<div id="directorButtons"></div>
<input type="search" id="search" placeholder="Search titles" />
<div id="filmList"></div>
<div id="detail"></div>
```

`app.js` in full:

```js
const statusText = document.querySelector('#status');
const loadButton = document.querySelector('#loadButton');
const filmList = document.querySelector('#filmList');
const searchBox = document.querySelector('#search');
const directorButtons = document.querySelector('#directorButtons');
const detail = document.querySelector('#detail');

const FILMS_URL = 'https://ghibliapi.vercel.app/films';

let allFilms = [];
const detailCache = {};

// --- Getting data ---------------------------------------------------

async function getFilms() {
  const response = await fetch(FILMS_URL);

  if (!response.ok) {
    throw new Error('The server answered with status ' + response.status);
  }

  return await response.json();
}

async function getOneFilm(id) {
  const response = await fetch(FILMS_URL + '/' + id);

  if (!response.ok) {
    throw new Error('The server answered with status ' + response.status);
  }

  return await response.json();
}

// --- Drawing --------------------------------------------------------

function renderFilms(films) {
  filmList.innerHTML = '';

  for (const film of films) {
    const card = document.createElement('div');
    card.classList.add('film');

    const heading = document.createElement('h2');
    heading.textContent = film.title;
    card.appendChild(heading);

    const facts = document.createElement('p');
    facts.textContent =
      film.director + ' - ' + film.release_date + ' - ' + film.running_time + ' min';
    card.appendChild(facts);

    card.addEventListener('click', function () {
      showDetail(film.id);
    });

    filmList.appendChild(card);
  }
}

function renderDetail(film) {
  detail.innerHTML = '';

  const heading = document.createElement('h2');
  heading.textContent = film.title;
  detail.appendChild(heading);

  const score = document.createElement('p');
  score.textContent = 'Rotten Tomatoes score: ' + film.rt_score;
  detail.appendChild(score);

  const description = document.createElement('p');
  description.textContent = film.description;
  detail.appendChild(description);
}

function renderDirectorButtons(films) {
  const directors = [];

  for (const film of films) {
    if (!directors.includes(film.director)) {
      directors.push(film.director);
    }
  }

  directorButtons.innerHTML = '';

  const allButton = document.createElement('button');
  allButton.type = 'button';
  allButton.textContent = 'All';
  allButton.addEventListener('click', function () {
    renderFilms(allFilms);
  });
  directorButtons.appendChild(allButton);

  for (const director of directors) {
    const button = document.createElement('button');
    button.type = 'button';
    button.textContent = director;

    button.addEventListener('click', function () {
      const theirs = allFilms.filter(function (film) {
        return film.director === director;
      });
      renderFilms(theirs);
    });

    directorButtons.appendChild(button);
  }
}

// --- Doing things ---------------------------------------------------

async function loadFilms() {
  statusText.textContent = 'Loading...';
  loadButton.disabled = true;

  try {
    allFilms = await getFilms();
    renderDirectorButtons(allFilms);
    renderFilms(allFilms);
    statusText.textContent = 'Loaded ' + allFilms.length + ' films.';
  } catch (error) {
    statusText.textContent = 'Could not load the films. ' + error.message;
  } finally {
    loadButton.disabled = false;
  }
}

async function showDetail(id) {
  if (detailCache[id]) {
    renderDetail(detailCache[id]);
    return;
  }

  detail.textContent = 'Loading that one...';

  try {
    const film = await getOneFilm(id);
    detailCache[id] = film;
    renderDetail(film);
  } catch (error) {
    detail.textContent = 'Could not load that film. ' + error.message;
  }
}

searchBox.addEventListener('input', function () {
  const term = searchBox.value.toLowerCase();

  const matches = allFilms.filter(function (film) {
    return film.title.toLowerCase().includes(term);
  });

  renderFilms(matches);
});

loadButton.addEventListener('click', loadFilms);
```

A few notes on the choices.

`renderDirectorButtons` is called once, from `loadFilms`, and not from `renderFilms`. If it were rebuilt on every render, the buttons would be recreated every time you typed a letter in the search box. Rebuild only what actually changed.

`showDetail` checks the cache before it does anything else, and returns early if it finds something. Note that it does not even show `Loading that one...` in that case, because there is nothing to wait for. An early `return` is often clearer than wrapping the rest of a function in an `else`.

`detailCache[id]` on an id we have never seen gives `undefined`, which is falsy, so the `if` is `false` and we go and fetch. That works, but it is worth knowing the limits of the trick: if a value you were caching could itself be falsy, such as `0` or an empty string, this test would keep re-fetching it forever.

`showDetail` writes its own loading and error messages into `detail`, not into `statusText`. Two things are loading independently, so they get their own places to say so. If they shared one status line they would overwrite each other.

`getOneFilm` and `getFilms` are nearly identical, which should make you itch. Combining them into one function that takes a path is a good five-minute refactor, and a fair thing to try before moving on.

</details>

---

## What to take away

Three sentences, and then you can close the tab.

**Your tab has one pen, and `await` is where a function puts it down.** Anything that takes real time - a timer, a network request - is handled by machinery outside your code, and your job is to hand the work over and get out of the way. If your page freezes, you are holding the pen.

**`fetch` gives you the envelope, then the letter, and does not read the letter for you.** Two `await`s. Then check `response.ok` yourself, because a 404 arrived perfectly successfully. Wrap the lot in `try`/`catch`, and tell the user what is happening while they wait.

**Fetch once, keep the array, render from the array.** The network is for getting data. Your variables are for everything you do with it afterwards.

If you want more, the regular Lesson 5.3 goes on to POST, PUT and DELETE, which is how you send data rather than only asking for it. Lesson 5.4 covers the Network tab properly, and is worth reading slowly: nearly every API bug you will ever have is visible there, in plain sight, if you know how to look.
