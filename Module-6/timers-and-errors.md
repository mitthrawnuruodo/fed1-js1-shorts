# A note for later, and the net you left behind

**Estimated time:** about 2 hours for the core path, plus 45 to 60 minutes for the self study task.  
**Prerequisites:** Modules 1 to 5, plus Lessons 6.1 and 6.2.

The regular lessons teach timers, then error handling, as two separate subjects. They are not really two subjects. They are two halves of the same question: **what happens to code that runs later, and who is still around to catch it when it goes wrong?**

Two sentences carry this lesson.

**The first:** `setTimeout` does not pause anything. It writes a note and walks on.

**The second:** `try...catch` wraps a stretch of code, not a stretch of time.

Put those together and you get the trap that this lesson is built around: a `try...catch` sitting right on top of a `setTimeout` catches absolutely nothing, and the reason why is worth more than either feature on its own.

---

## Part 1 - The note, and walking on

Start in the console.

```js
console.log('first');

setTimeout(function () {
  console.log('second');
}, 2000);

console.log('third');
```

The order is `first`, `third`, `second`.

The word "timeout" makes it sound like a pause, and it is not one. Nothing waited. `setTimeout` did its job instantly: it wrote a note that says *run this function in two seconds*, handed the note to the browser, and returned. The next line ran immediately.

That is the whole model, and it is worth having a picture for it:

> `setTimeout(fn, 2000)` means: **write "run fn" on a note, put it in the browser's tray marked "in 2 seconds", and carry on with your day.**

The browser watches the tray. When the two seconds are up, it waits for your code to be doing nothing at all, and only then does it run the function.

### The bit that catches everyone

Try to build a countdown with a loop.

```js
for (let i = 1; i <= 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, 1000);
}
```

You expect `1`, then a pause, `2`, a pause, `3`.

What you get is a one-second pause and then `1 2 3` all at once, in a rush.

Read it again with the note picture in mind. The loop does not pause either. It runs all three iterations immediately, in under a millisecond, and every single one of them writes a note saying *in one second*. One second from **now**, and all three "now"s are the same instant. So all three notes come due at the same moment.

The fix is to write a different time on each note:

```js
for (let i = 1; i <= 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, i * 1000);
}
```

One second, two seconds, three seconds. Now it counts.

> **Tip: the delay is a minimum, not a promise.** The browser will not run your function before the delay is up, but it may well run it after. If your code is busy with something slow when the note comes due, the note waits its turn. `setTimeout(fn, 1000)` means "not sooner than one second", never "exactly one second".

### The receipt

`setTimeout` gives you something back: a number that identifies that particular note.

```js
const noteId = setTimeout(function () {
  console.log('You will not see this.');
}, 3000);

clearTimeout(noteId);
```

Nothing is logged. `clearTimeout` finds the note in the tray and tears it up.

Think of the returned number as a cloakroom ticket. It is only useful if you keep it. If you write

```js
setTimeout(doSomething, 5000);
```

and later change your mind, there is nothing you can do. The note is in the tray, you have no ticket, and it is going to happen.

> **Tip: `clearTimeout` on nonsense is harmless.** Calling it with an id that has already fired, or with a variable that is still `undefined`, does nothing at all and throws no error. That turns out to be genuinely useful, and we lean on it in Part 2.

<details>
<summary>Rabbit hole: why is there a stray number under my loop?</summary>

If you ran the countdown loop in the console rather than in a file, you saw something like this:

```
for (let i = 1; i <= 3; i++) { setTimeout(function () { console.log(i); }, 1000); }
4
1
2
3
```

Where did the `4` come from? Nothing in that code logs a `4`.

The console prints two different kinds of thing, and it does not label which is which. There is the output your code produces with `console.log`, and there is the **value of the statement you typed**, which the console echoes back automatically. That echo is the `4`.

Now that you know `setTimeout` returns a timer id, the rest follows. The value of a loop is the value of the last thing that happened inside it, which here is the third `setTimeout` call, which is the third note's id. Run the same line again and the number will have gone up, because ids are handed out in order and keep counting for the life of the page:

```
for (let i = 1; i <= 3; i++) { setTimeout(function () { console.log(i); }, 1000); }
7
1
2
3
```

Same three notes, ids five, six and seven this time.

You have almost certainly never noticed this before, because most things you type echo `undefined`, which is quiet enough to scroll past:

| What you type | What is echoed |
| --- | --- |
| `console.log('hello')` | `undefined` |
| `const x = 5;` | `undefined` |
| `if (true) { 41 + 1 }` | `42` |
| `setTimeout(fn, 1000)` | a timer id |

It shows up again in Part 4, where the demonstration is a `try...catch` wrapped round a `setTimeout`. That whole statement echoes a timer id too, so you will get a bare number immediately and the interesting part a second later. Do not let the number distract you; it is not the answer to anything.

If it bothers you, end the line with `undefined` and the console will echo that instead. Or put the code in a file, where nothing echoes at all.

One more Firefox-specific oddity, since it appears in the same output: each logged line is tagged `debugger eval code:3:13`. That is Firefox telling you which line the `console.log` was on. It says line 3 rather than line 1 because the console wraps whatever you type in a small function before running it, and the line number refers to that wrapper. Chrome shows the same information as `VM123:1`. In both cases it is bookkeeping about the console, not about your code.

</details>

### Exercise 1 - Three notes

**Goal:** to be sure about when notes are written and when they come due.

**Steps**

1. Predict the output order of this, before running it. Write your prediction down.

   ```js
   console.log('A');

   setTimeout(function () {
     console.log('B');
   }, 0);

   for (let i = 0; i < 3; i++) {
     console.log('C' + i);
   }

   console.log('D');
   ```

2. Run it. If your prediction was wrong, write one sentence explaining why `B` lands where it does even with a delay of `0`.
3. Now build a proper countdown: log `3`, `2`, `1`, `Go!` at one-second intervals, using only `setTimeout` inside a loop plus one extra call.
4. Give the countdown a cancel: store the note ids, and write a `cancel()` function that stops any of them that have not fired yet.

<details>
<summary>Solution</summary>

**Steps 1 and 2**

The order is `A`, `C0`, `C1`, `C2`, `D`, `B`.

`B` is last despite the `0`. A delay of `0` does not mean "now" - it means "as soon as you are free". The browser only reads notes from the tray once your code has run out of things to do, and everything else in this script is still queued up ahead of it. `0` moves the note to the front of the tray, not to the front of your program.

**Steps 3 and 4**

```js
const noteIds = [];

for (let i = 3; i >= 1; i--) {
  const secondsToWait = 3 - i;

  const id = setTimeout(function () {
    console.log(i);
  }, secondsToWait * 1000);

  noteIds.push(id);
}

const goId = setTimeout(function () {
  console.log('Go!');
}, 3000);

noteIds.push(goId);

function cancel() {
  noteIds.forEach(function (id) {
    clearTimeout(id);
  });

  console.log('Countdown cancelled.');
}
```

The counting is the fiddly part. The loop counts `i` down from 3, but the waits have to count **up** from zero, which is what `3 - i` gives us: `3` after no wait at all, `2` after one second, `1` after two.

Notice that `cancel` can be called at any point and does the right thing. Ids belonging to notes that have already fired are simply ignored.

</details>

---

## Part 2 - A note that keeps coming back

Something worth building: a small board that watches an exchange rate and refreshes itself.

We will use [Frankfurter](https://frankfurter.dev/), a free exchange rate API with no key and no sign-up. One address gives us one number:

```
https://api.frankfurter.dev/v2/rate/EUR/NOK
```

Open that in a browser tab. The reply is small enough to read in one go:

```json
{ "date": "2026-08-25", "base": "EUR", "quote": "NOK", "rate": 11.7412 }
```

Create two files.

**index.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Rate ticker</title>
  </head>
  <body>
    <h1>1 EUR in NOK</h1>

    <p id="rate">Not checked yet</p>
    <p id="status"></p>

    <button id="start">Start</button>
    <button id="stop">Stop</button>

    <script src="script.js"></script>
  </body>
</html>
```

**script.js**

```js
const rateOutput = document.querySelector('#rate');
const statusOutput = document.querySelector('#status');
const startButton = document.querySelector('#start');
const stopButton = document.querySelector('#stop');

const RATE_URL = 'https://api.frankfurter.dev/v2/rate/EUR/NOK';
const CHECK_DELAY = 10000;

let tickerId;

async function check() {
  const response = await fetch(RATE_URL);
  const data = await response.json();

  rateOutput.textContent = data.rate;
  statusOutput.textContent = 'Updated';
}

startButton.addEventListener('click', function () {
  check();
  tickerId = setInterval(check, CHECK_DELAY);
});

stopButton.addEventListener('click', function () {
  clearInterval(tickerId);
  statusOutput.textContent = 'Stopped';
});
```

Open it, click Start, and watch. A number appears, and every ten seconds it quietly refreshes itself.

`setInterval` is the same note, except the browser puts it straight back in the tray after every reading. One note, read over and over, until somebody tears it up. `clearInterval` is the tearing up, and it needs the ticket in exactly the same way `clearTimeout` does.

The `check()` call on its own line before the `setInterval` is not a mistake. `setInterval(check, 10000)` waits ten seconds before its first run, so without that extra call the board would sit there saying "Not checked yet" for ten seconds after you clicked Start.

> **Tip: ten seconds is a long time to wait while developing.** Drop `CHECK_DELAY` to `2000` while you are working on this, and put it back afterwards. Rates only move once a working day, so a real version of this would poll far less often than either number, but a fast tick makes the behaviour visible.

### Exercise 2 - The runaway ticker

Do this before reading on. Click Start. Then click Start again. Then again.

Watch the Network tab in your developer tools.

**Goal:** to see what an untracked timer costs you, and to stop it happening.

**Steps**

1. Click Start three or four times and describe, in the Network tab, what is now happening.
2. Click Stop once. What happens? Why does it not fix it?
3. Fix `startButton` so that clicking Start any number of times leaves exactly one ticker running.

<details>
<summary>Solution</summary>

**Steps 1 and 2**

Every click starts an entirely new interval. After four clicks there are four separate notes in the tray, all firing every ten seconds, so you are making four requests per cycle instead of one.

Clicking Stop kills exactly one of them - the last one, because `tickerId` was overwritten each time and only remembers the most recent ticket. The other three are unreachable. There is no way to stop them, short of reloading the page. This is a **leak**: something is running, it is consuming a real resource, and you have thrown away the only handle you had on it.

**Step 3**

```js
startButton.addEventListener('click', function () {
  clearInterval(tickerId);

  check();
  tickerId = setInterval(check, CHECK_DELAY);
});
```

One extra line. Before starting a ticker, stop whatever ticker might already be running.

On the very first click `tickerId` is still `undefined`, and `clearInterval(undefined)` does nothing and complains about nothing, so there is no need to guard it with an `if`. This is the harmless-nonsense property from Part 1 earning its keep.

The rule underneath: **anything you start, you need to be able to stop.** If you cannot name it, you cannot stop it.

</details>

---

## Part 3 - When the reply does not come, and when it says no

Our ticker works beautifully as long as nothing goes wrong. Let us make something go wrong.

In your developer tools, find the Network tab and tick the **Offline** checkbox. Then wait for the next tick.

The board freezes on its old number, and the console has an angry red line:

```
Uncaught (in promise) TypeError: Failed to fetch
```

The `await fetch(...)` threw. Everything after it in `check` was skipped, so nothing updated and nothing told the user why. To a person looking at the page, the number is simply wrong now, with no hint that it is stale.

This is what `try...catch` is for.

```js
async function check() {
  statusOutput.textContent = 'Checking...';

  try {
    const response = await fetch(RATE_URL);
    const data = await response.json();

    rateOutput.textContent = data.rate;
    statusOutput.textContent = 'Updated';
  } catch (error) {
    statusOutput.textContent = 'Could not update: ' + error.message;
  }
}
```

Turn Offline back on. The board now says `Could not update: Failed to fetch`, and keeps trying. Untick Offline and the next tick recovers on its own.

The thing arriving in `catch (error)` is an object with two properties you will use constantly:

```js
} catch (error) {
  console.log(error.name);    // TypeError
  console.log(error.message); // Failed to fetch
}
```

`name` is the category, `message` is the description. `error.message` is usually fine to log; it is usually *not* fine to show a user, because it is written for developers. `Failed to fetch` means nothing to somebody who just wanted to know what their euros are worth.

It is also not the same everywhere. Chrome and Edge say `Failed to fetch`; Firefox says something longer about a network error. The `name` is `TypeError` in both, which is one small reason to lean on `name` rather than on the exact wording of `message` when your code needs to make a decision.

### Now the sneaky failure

Untick Offline. Instead, break the address. Change `EUR` to `EURO`:

```js
const RATE_URL = 'https://api.frankfurter.dev/v2/rate/EURO/NOK';
```

Reload and click Start.

No red console error. No `Could not update`. The status line says **Updated**, confidently, and the rate line has gone blank.

Blank, rather than the word `undefined`, which is what you might have expected. `data.rate` genuinely is `undefined` - there is no `rate` in a 404 reply - but assigning `undefined` to `textContent` empties an element rather than writing the word into it. That makes this worse, not better: a blank line looks like something still loading, not like something broken.

Nothing was caught, because nothing was thrown, because as far as `fetch` is concerned nothing went wrong. Look in the Network tab: the request went out, the server received it, understood it, and replied `404 Not Found` with a body explaining that there is no such currency. That is a successful conversation. The letter was delivered and a reply came back. The reply just happens to say no.

> `fetch` only throws when the message never got through. A reply saying "no" is still a reply.

So checking the reply is our job. `response.ok` is `true` for status codes in the 200s and `false` for everything else, and when it is false, we throw the error that `fetch` declined to throw:

```js
try {
  const response = await fetch(RATE_URL);

  if (!response.ok) {
    throw new Error('The server said no. Status: ' + response.status);
  }

  const data = await response.json();

  rateOutput.textContent = data.rate;
  statusOutput.textContent = 'Updated';
} catch (error) {
  statusOutput.textContent = 'Could not update: ' + error.message;
}
```

Reload with the broken address still in place. Now it says `Could not update: The server said no. Status: 404`.

`throw` is how you say "stop, this is not acceptable" about something JavaScript is perfectly happy with. The `Error` object you throw lands in the same `catch` as a real crash would, with the same `name` and `message` properties, and everything downstream treats it identically. That is the point: your rules and JavaScript's rules use the same plumbing.

Put `EUR` back before carrying on.

### Exercise 3 - Three ways to fail

**Goal:** to produce each kind of failure deliberately, and to tell them apart.

**Steps**

1. Add a check that throws if the reply arrives but has no usable rate in it - if `data.rate` is not a number. Give it a clear message.
2. Cause all three failures in turn and note what `error.name` is for each:
   - no connection (the Offline checkbox)
   - a rejected address (`EURO` instead of `EUR`)
   - a reply with no rate (temporarily change `data.rate` to `data.price` in your code to fake it)
3. Change your `catch` so the status line shows a message a normal person could act on, rather than `error.message`, while still logging the technical detail to the console.

<details>
<summary>Solution</summary>

**Step 1**

```js
const data = await response.json();

if (typeof data.rate !== 'number') {
  throw new Error('The reply did not contain a rate.');
}

rateOutput.textContent = data.rate;
```

**Step 2**

Offline gives `TypeError` - JavaScript's own, thrown by `fetch` itself. The other two give `Error`, because that is what we constructed. That difference is a real clue when reading a log: `TypeError` here almost always means the request never completed, and a plain `Error` almost always means our own code rejected something.

**Step 3**

```js
} catch (error) {
  console.log('Check failed:', error.name, error.message);
  statusOutput.textContent = 'Could not reach the rate service. Trying again shortly.';
}
```

Two audiences, two messages, one `catch`. The developer gets the truth in the console; the user gets something that tells them what is happening and that it is being dealt with.

You could go further and vary the user's message by inspecting the error, and sometimes that is worth doing. Often it is not: from the user's point of view "the rate is not available right now" covers all three cases equally well, and three different phrasings of the same non-news is just noise.

</details>

---

## Part 4 - The net that catches nothing

Now the important one. Run this exactly as it is.

```js
try {
  setTimeout(function () {
    throw new Error('Too late');
  }, 1000);
} catch (error) {
  console.log('Caught it:', error.message);
}
```

`Caught it` never appears. A second later, `Uncaught Error: Too late` shows up in red instead.

There is a `throw`. There is a `catch`, wrapped right round it, visibly enclosing it on the screen. The error is not caught.

Read it with the two sentences from the start of this lesson.

The `try` block's job was to call `setTimeout`. It did. Writing the note **worked perfectly** - there was nothing to catch, so the block finished, and the net was folded up and put away. About a second later the browser takes the note out of the tray and runs the function on it. That function throws. But it is running in an empty room. The `try` block ended a second ago, and there has not been a net under this code for a very long time in computer terms.

> A `try...catch` catches what falls **inside the block**. The callback does not run inside the block. It runs later, somewhere else entirely, and the block is long gone by then.

The fix is to put the net where the falling actually happens - inside the callback:

```js
setTimeout(function () {
  try {
    throw new Error('Too late');
  } catch (error) {
    console.log('Caught it:', error.message);
  }
}, 1000);
```

`Caught it: Too late`.

Once you have seen this, you can spot the shape from across a room. Any time a `try` block contains a function that will be called later - a `setTimeout` callback, a `setInterval` callback, an event listener, an `addEventListener` handler - the `catch` will not cover what happens inside it. Wrapping the registration is not wrapping the work.

### So why does `await` work?

Fair question, because this is in our ticker and it does work:

```js
try {
  const response = await fetch(RATE_URL);
} catch (error) {
  // this really does catch a network failure
}
```

`fetch` takes time - sometimes seconds. So why is the net still there when it fails, if it was not there for `setTimeout`?

Because `await` does something a callback cannot: it pauses the function you are already in, and later resumes it **at the same spot, with everything still around it**. Your position inside the `try` block is part of what gets frozen and thawed. When the failure comes back, you are still standing in the block, and the net is still up.

A callback is not a resumption. It is a different function, called fresh from somewhere else, with nothing of yours around it.

> `await` keeps you inside the block. `setTimeout` sends you outside it.

That single distinction explains most confusing error handling in asynchronous JavaScript, and it is the reason `async`/`await` largely replaced callbacks for this kind of work.

### Exercise 4 - Move the net

Here is a small auto-dismissing notice. It works most of the time and dies loudly when the element has already been removed.

```js
function showNotice(text) {
  const notice = document.querySelector('#notice');
  notice.textContent = text;

  try {
    setTimeout(function () {
      notice.textContent = '';
      notice.parentElement.classList.remove('has-notice');
    }, 3000);
  } catch (error) {
    console.log('Could not clear the notice:', error.message);
  }
}
```

**Goal:** to see the net in the wrong place and put it in the right one.

**Steps**

1. Explain why the `catch` can never run, even though the code inside the timeout can definitely fail.
2. Move the net so it works. Prove it by calling `showNotice('Saved')` and then removing the notice element from the page with `document.querySelector('#notice').remove()` before three seconds are up.
3. There is a second, arguably better fix that does not involve `try...catch` at all. What is it?

<details>
<summary>Solution</summary>

**Step 1**

The only thing inside the `try` block is `setTimeout(...)`, which succeeds. Scheduling a note never fails. Three seconds later the callback runs, `notice.parentElement` is `null` because the element is no longer in the page, and reading `.classList` off `null` throws a `TypeError` - in an empty room, with the block long finished.

**Step 2**

```js
function showNotice(text) {
  const notice = document.querySelector('#notice');
  notice.textContent = text;

  setTimeout(function () {
    try {
      notice.textContent = '';
      notice.parentElement.classList.remove('has-notice');
    } catch (error) {
      console.log('Could not clear the notice:', error.message);
    }
  }, 3000);
}
```

The net moves inside the callback, which is where the falling happens.

**Step 3**

Keep the ticket and tear the note up when the notice goes away:

```js
let noticeId;

function showNotice(text) {
  const notice = document.querySelector('#notice');
  notice.textContent = text;

  clearTimeout(noticeId);

  noticeId = setTimeout(function () {
    notice.textContent = '';
  }, 3000);
}

function hideNotice() {
  clearTimeout(noticeId);
  document.querySelector('#notice').textContent = '';
}
```

This is better because it prevents the error rather than catching it. `try...catch` is for things you cannot prevent - the network, other people's servers, data you did not write. A note that should no longer fire is entirely within your control, and the right response to it is `clearTimeout`, not a net.

Both fixes are worth having in your head. The question to ask is always: could I have stopped this from happening at all?

</details>

<details>
<summary>Rabbit hole: the same trap, one layer up</summary>

An `async` function that you call without `await` has the same problem, for the same reason.

```js
try {
  check(); // async, no await
} catch (error) {
  console.log('never runs');
}
```

Calling `check()` starts it and hands you back a promise immediately. The `try` block finishes at once, net folded away, while `check` is still going. When it fails later, there is nobody there - the browser reports an "Uncaught (in promise)" error, which is its way of saying *this went wrong and nothing was listening*.

That message is a smell worth learning to recognise. It nearly always means somebody started an asynchronous job and walked away from it.

Two ways out: `await` the call, so you stay in the block; or handle failure inside the function itself, which is what our `check` does. The second is why `startButton` can call `check()` without `await` and still be safe - the net is inside.

</details>

---

## Part 5 - The block that always runs, and a better ticker

Our `check` sets the status to `Checking...` at the start. On the happy path something replaces it. On the sad path something replaces it. But there is a third path we have not thought about: what if we later add an early `return`, or throw from inside the `catch`? Then `Checking...` stays on screen forever and the user believes something is still happening.

`finally` is the answer to "this must happen either way".

```js
try {
  // might work, might not
} catch (error) {
  // only if it did not
} finally {
  // both ways, always
}
```

It runs after the `try` succeeds. It runs after the `catch` handles a failure. It even runs if you `return` out of the middle. It is the "put the chairs away" block.

### Making `finally` earn its place

We could use it for tidying up the status line, and that would be fine but a bit dull. There is something better to do with it, and it fixes a real problem in our ticker.

Think about what `setInterval(check, 10000)` actually promises: it fires **every ten seconds, no matter what**. It does not know or care whether the previous check finished. On a slow connection, a check that takes twelve seconds is still in flight when the next one starts, and now you have two overlapping requests, and then three, and their replies can arrive out of order and overwrite each other with older data.

What we actually want is *ten seconds after the last check finished*. And "after it finished, whether it worked or not" is precisely what `finally` means.

So we throw away `setInterval` entirely and let each check schedule the next one:

```js
const RATE_URL = 'https://api.frankfurter.dev/v2/rate/EUR/NOK';
const CHECK_DELAY = 10000;

let tickerId;
let running = false;

async function check() {
  statusOutput.textContent = 'Checking...';

  try {
    const response = await fetch(RATE_URL);

    if (!response.ok) {
      throw new Error('The server said no. Status: ' + response.status);
    }

    const data = await response.json();

    if (typeof data.rate !== 'number') {
      throw new Error('The reply did not contain a rate.');
    }

    rateOutput.textContent = data.rate;
    statusOutput.textContent = 'Updated';
  } catch (error) {
    console.log('Check failed:', error.name, error.message);
    statusOutput.textContent = 'Could not reach the rate service.';
  } finally {
    if (running) {
      tickerId = setTimeout(check, CHECK_DELAY);
    }
  }
}

startButton.addEventListener('click', function () {
  clearTimeout(tickerId);
  running = true;
  check();
});

stopButton.addEventListener('click', function () {
  clearTimeout(tickerId);
  running = false;
  statusOutput.textContent = 'Stopped';
});
```

There is no interval anywhere now. There is one note at a time, and each check writes the next one on its way out. A chain, not a metronome.

The `if (running)` matters more than it looks. Press Stop while a check is in flight and two things happen: `clearTimeout` tears up any pending note, and `running` becomes false. When the in-flight check finally lands, its `finally` block asks "are we still meant to be running?", gets no for an answer, and quietly declines to schedule another. Without that check, Stop would appear to work and then the ticker would start up again a few seconds later, which is a maddening bug to chase.

### Exercise 5 - Prove that `finally` always runs

**Goal:** to see all three paths for yourself rather than taking my word for it.

**Steps**

1. Add a `console.log` at the top of your `finally` block.
2. Trigger each path in turn and confirm the log appears every time: a normal successful check; a check with Offline ticked; a check with the address broken to `EURO`.
3. Now the surprising one. Put a `return` at the end of the `try` block, before the closing brace. Does `finally` still run?
4. Write a small function of your own that proves the answer to step 3 in isolation, without any of the ticker around it.

<details>
<summary>Solution</summary>

**Steps 1 and 2**

The log appears on all three paths. That is the entire promise of `finally`.

**Steps 3 and 4**

Yes, it still runs. `finally` runs before the function actually hands control back.

```js
function test() {
  try {
    console.log('in try');
    return 'returned from try';
  } finally {
    console.log('in finally');
  }
}

console.log(test());
```

Output:

```
in try
in finally
returned from try
```

The `return` value is worked out, then `finally` runs, then the function returns. This is why `finally` is the right home for cleanup: there is no way out of the `try` block that skips it. No happy path, no error, and no early `return` can dodge it.

Note that this example has no `catch` at all. `try...finally` without a `catch` is perfectly legal, and it means "I am not going to handle this error, but I am going to tidy up before it carries on upwards".

</details>

---

## Part 6 - Knowing when to stop

One more thing a real ticker needs. If the rate service has been down for an hour, hammering it every ten seconds helps nobody, and the user has long since stopped believing the status line.

Give it a budget.

```js
const MAX_FAILURES = 3;

let failures = 0;
```

In the `try` block, after everything has succeeded:

```js
rateOutput.textContent = data.rate;
statusOutput.textContent = 'Updated';
failures = 0;
```

And in the `catch`:

```js
} catch (error) {
  console.log('Check failed:', error.name, error.message);

  failures = failures + 1;

  if (failures >= MAX_FAILURES) {
    running = false;
    statusOutput.textContent =
      'Gave up after ' + failures + ' failed attempts. Press Start to try again.';
  } else {
    statusOutput.textContent =
      'Attempt ' + failures + ' failed. Trying again shortly.';
  }
}
```

`startButton` should reset the count too, since pressing Start is the user saying "try again from scratch":

```js
startButton.addEventListener('click', function () {
  clearTimeout(tickerId);
  running = true;
  failures = 0;
  check();
});
```

Tick Offline and watch it count up to three and stop. Untick it, press Start, and it recovers.

Look at how the three blocks cooperate here, because this is the shape you will write again and again:

- the **try** does the work and, at the very end, records that the work succeeded
- the **catch** decides what a failure means this time - is it a blip, or is it time to give up
- the **finally** carries out whatever decision was made, without needing to know which one it was

The `finally` block never asks whether the check worked. It only asks whether we are still running, and both of the other blocks are free to change that answer. Keeping the decision and the action separate like this is why the whole thing stays readable when the rules get more complicated.

<details>
<summary>Rabbit hole: backing off instead of giving up</summary>

Stopping dead after three tries is blunt. A gentler version keeps trying but waits longer each time, so a service that comes back after five minutes is picked up automatically without anyone pressing anything.

```js
function delayForAttempt(failureCount) {
  if (failureCount === 0) {
    return CHECK_DELAY;
  }

  return CHECK_DELAY * Math.pow(2, failureCount);
}
```

With a ten second base, that gives ten seconds while healthy, then twenty, forty, eighty, and so on. In `finally`:

```js
} finally {
  if (running) {
    tickerId = setTimeout(check, delayForAttempt(failures));
  }
}
```

Because `failures` resets to zero on any success, the delay snaps straight back to normal the moment the service recovers.

This is called **exponential backoff**, and it is close to universal in software that talks to other software. The reasoning is worth knowing: when a server is struggling, a thousand clients all retrying at a fixed interval is exactly the traffic pattern that keeps it down. Backing off is partly self-interest and partly good manners.

Real implementations usually add a cap, so the delay does not grow to hours, and a small random amount, so that clients that failed together do not all come back at the same instant.

</details>

<details>
<summary>Rabbit hole: giving your errors a surname</summary>

We are currently telling our two kinds of failure apart by reading `error.name`, which works but is coarse: everything we throw ourselves is just `Error`.

You can make your own kinds:

```js
class RateServiceError extends Error {
  constructor(message) {
    super(message);
    this.name = 'RateServiceError';
  }
}
```

`extends Error` means "the same as an Error, plus my changes". `super(message)` hands the message up to the built-in `Error` so that `error.message` works normally. Setting `this.name` is what makes it identifiable.

Then throw it, and test for it with `instanceof`:

```js
if (!response.ok) {
  throw new RateServiceError('The server said no. Status: ' + response.status);
}
```

```js
} catch (error) {
  if (error instanceof RateServiceError) {
    statusOutput.textContent = 'The rate service is having trouble.';
  } else {
    statusOutput.textContent = 'Could not reach the rate service.';
  }
}
```

Why bother, when you could check `error.message`? Because messages get rewritten. Somebody improves the wording six months from now and a `catch` block that was matching on text silently stops matching. A type survives rewording.

For a page with two failure modes this is more machinery than the problem deserves. For an application where one kind of error means "show a login screen" and another means "retry quietly", it pays for itself quickly.

</details>

---

## Part 7 - The loose end from last time

The previous extra lesson ended with a deliberate cliffhanger. In the lending shelf, `load` looked like this:

```js
function load() {
  const saved = localStorage.getItem('lendingShelf');

  if (saved) {
    shelf = JSON.parse(saved);
  }
}
```

And we showed that anything unreadable in storage - written by an older version of your own code, by another tab, by an extension, by you poking about in the console - would make `JSON.parse` throw and take the entire page down before it ever drew anything.

You now have the tool.

```js
function load() {
  const saved = localStorage.getItem('lendingShelf');

  if (!saved) {
    return;
  }

  try {
    shelf = JSON.parse(saved);
  } catch (error) {
    console.log('The stored shelf was unreadable:', error.message);
    shelf = [];
  }
}
```

Go back to that project and try it. Put rubbish in storage with `localStorage.setItem('lendingShelf', 'this is not JSON')`, reload, and instead of a dead page you get an empty shelf, a note in the console, and a working application.

Three things about that `catch` block are worth naming, because they generalise well beyond this example.

It **decides something**. `shelf = []` is a real choice: we are saying that unreadable data should be treated as no data. The alternative - showing the user an error and refusing to continue - is also defensible. What is not defensible is an empty `catch` block that swallows the problem and leaves the program in a state nobody chose.

It **leaves a trace**. The `console.log` is not decoration. Without it, the shelf silently empties itself one day and there is no way to find out why.

And it is **narrow**. Only the parse is inside it. If we had wrapped the whole of `load`, a bug in some later line would also be quietly caught and turned into an empty shelf, and we would be debugging a phantom. A `try` block should contain the thing you expect to fail and as little else as possible.

---

## Self study task: an auto-saving notepad

A different app, exercising the same three ideas: a note for later, a net in the right place, and a decision about when to stop trying.

**The idea.** A text box that saves itself two seconds after you stop typing. Saving sometimes fails, because saving always sometimes fails. The page has to tell you honestly what is going on without nagging.

**The two helpers.** Paste these in as they are. `wait` is a `setTimeout` wrapped in a promise so you can `await` it, and `fakeSave` is a pretend server that takes about half a second and fails roughly one time in three.

```js
function wait(milliseconds) {
  return new Promise(function (resolve) {
    setTimeout(resolve, milliseconds);
  });
}

async function fakeSave(text) {
  await wait(500);

  if (Math.random() < 0.34) {
    throw new Error('The server rejected the save.');
  }

  return { savedAt: Date.now(), length: text.length };
}
```

**Level 1**

1. A `<textarea>` and a status line.
2. Typing sets the status to `Unsaved changes`.
3. Two seconds after the **last** keystroke, save. Not two seconds after the first one, and not once per keystroke. Every keystroke should tear up the pending note and write a fresh one, so that the save only happens once the typing stops. This pattern is called **debouncing**, and it is `clearTimeout` plus `setTimeout` and nothing more.
4. Handle both outcomes: `Saved` on success, something honest on failure.

**Level 2**

5. Show the time of the last successful save, formatted with `Intl.DateTimeFormat` and `timeStyle`, using the `savedAt` timestamp that `fakeSave` returns. You built exactly this in the previous extra lesson.
6. Disable the textarea while a save is in flight, and re-enable it afterwards on both paths. Put the re-enabling where it cannot be skipped.
7. On failure, retry automatically after five seconds rather than waiting for the next keystroke.

**Level 3 (recommended)**

8. Give the retries a budget. After three consecutive failures, stop retrying and tell the user their work is not saved and they should copy it somewhere safe. Any new keystroke starts the whole cycle over.
9. Prevent a save firing while another is still in flight. `fakeSave` takes half a second, and a fast typist can easily get two going at once.
10. Add a `Save now` button that saves immediately and cancels any pending automatic save, so you do not get a second save two seconds later doing the same work again.

<details>
<summary>Hints, if you want them</summary>

**On step 3.** One variable holding one ticket, at the top of the file. Every `input` event does the same two things: `clearTimeout(saveNoteId)` and then `saveNoteId = setTimeout(save, 2000)`. Tear up the old note, write a new one. That is the entire pattern.

**On step 4.** `fakeSave` is `async` and throws, so the net goes inside your own `async` save function, around the `await fakeSave(...)`. Not around the `setTimeout` that schedules it - see Part 4 if that is not yet obvious.

**On step 6.** `textarea.disabled = true` and `false`. "Where it cannot be skipped" means `finally`. If you re-enable it at the end of the `try`, a failure leaves the user locked out of their own text, which is about the worst possible outcome for a notepad.

**On step 7.** The retry is another note, so it needs its own ticket if you want step 10 to be able to cancel it. It is fine for the retry and the debounce to share one variable, as long as you are honest with yourself about the fact that they do.

**On step 8.** Same shape as the ticker: a counter, incremented in `catch`, reset to zero on success and on any new keystroke.

**On step 9.** A boolean like `let saving = false`, set to `true` before the `await` and back to `false` in `finally`. Check it before starting. Think about what the right behaviour is when a save is blocked this way - dropping it entirely means the last thing you typed never gets saved, which is not acceptable in a notepad. Setting a flag to save again as soon as the current one lands is one honest answer.

**On step 10.** `clearTimeout` first, then save. If the button just calls `save()` without cancelling, the pending note fires two seconds later and saves the identical text a second time.

</details>

---

## What to take away

**A timer is a note, not a pause.** `setTimeout` schedules and returns immediately. Everything after it runs first. If you might ever want to cancel, keep the ticket.

**Anything you start, you need to be able to stop.** An interval whose id you did not keep is unreachable and will run until the page closes.

**`try...catch` covers a block, not a period of time.** A callback scheduled inside a `try` block runs long after the block has ended, and the `catch` will not see its errors. Put the net inside the callback. `await` is the exception, and only because it resumes you in the same place, still inside the block.

**`fetch` only throws when the reply never arrived.** A `404` is a successful conversation with a disappointing outcome. Check `response.ok` and throw your own error, and treat data that arrived but is not usable exactly the same way.

**`finally` is for what must happen either way.** Cleanup, re-enabling, rescheduling. Nothing that leaves the `try` block can skip it - not success, not failure, not an early `return`.

And the one underneath all of them: **decide what failure means before it happens.** A `catch` block that quietly swallows a problem is worse than no `catch` at all, because now the program carries on in a state that nobody chose and nothing anywhere records that it happened.
