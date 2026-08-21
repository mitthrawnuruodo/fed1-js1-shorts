# Extra lesson: Four verbs and one parcel

**Estimated time:** about 2 hours for the core path, plus 45 to 60 minutes for the self study task.  
**Prerequisites:** Modules 1 to 4, plus Lessons 5.1 and 5.2. You need `fetch`, `async`/`await`, `response.json()`, and enough DOM to select an element, create one, and listen for a click.  

## How this lesson is different

The regular lessons take one method at a time - GET, then POST, then PUT, then DELETE - and each one is a separate recipe with its own block of code. Lesson 5.4 then arrives afterwards and teaches you the debugging tools.

This lesson does two things differently.

First, we treat all four methods as **one shape with different labels on it**. By Part 5 we write a single function that can do all four, because by then you will see that they were never really four different things.

Second, the debugging tools from Lesson 5.4 turn up **at the moment you would actually reach for them**. There is no separate debugging section at the end. When we send our first POST, that is when the Network tab appears, because that is when you would want it.

Two sentences hold the whole lesson together.

**One: every request is a parcel.** The address on the front is the URL, and it says *which thing*. The label is the method and the headers, and it says *what to do with that thing* and *how to read what is inside*. The box is the body, and it holds the new contents. GET and DELETE send an empty box.

**Two: the answer is a number before it is data.** The server always replies with a status code first. `response.ok` only tells you that number was somewhere in the 200s. It does not tell you that you got what you wanted, and it does not tell you there is anything in the box coming back.

### The API we will use

We will use **JSONPlaceholder** at `https://jsonplaceholder.typicode.com`. No key, no sign-up, no token, and it allows requests from any page, so nothing gets in the way of the JavaScript.

It gives you six collections of fake data:

| Path | What is in it |
| --- | --- |
| `/posts` | 100 posts |
| `/comments` | 500 comments |
| `/albums` | 100 albums |
| `/photos` | 5000 photos |
| `/todos` | 200 to-do items |
| `/users` | 10 users |

We will mostly use `/todos` and `/users`.

There is one thing you must know before we start, and it matters more than it sounds.

> **Writes are faked.** When you POST, PUT or DELETE against JSONPlaceholder, the server answers exactly as a real server would - correct status code, correct response body - but it changes nothing. Send the same POST twice and you get the same new id both times. Ask for the thing you just created and you get a 404.

That sounds like a limitation. It is actually rather useful for learning, because it means you can send a DELETE two hundred times and break nothing, and because it forces a habit that will serve you well: **the response is what the server says happened, not proof of what happened.** In Part 4 we will come back to that.

---

## Before you start

Make a folder with three files.

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Chore board</title>
    <link rel="stylesheet" href="style.css" />
    <script src="app.js" defer></script>
  </head>
  <body>
    <h1>Chore board</h1>
    <p id="status">Nothing loaded yet.</p>
    <button id="loadButton" type="button">Load chores</button>
    <ul id="choreList"></ul>
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

#choreList {
  list-style: none;
  padding: 0;
}

#choreList li {
  border: 1px solid #ccc;
  border-radius: 6px;
  padding: 0.5rem 1rem;
  margin-bottom: 0.5rem;
}

.done {
  color: #777;
  text-decoration: line-through;
}
```

`app.js`:

```js
const baseUrl = 'https://jsonplaceholder.typicode.com';

const statusText = document.querySelector('#status');
const loadButton = document.querySelector('#loadButton');
const choreList = document.querySelector('#choreList');

console.log('app.js is running');
```

**Open the page with Live Server, not by double-clicking the file.** In VS Code, install the Live Server extension, right-click `index.html`, choose "Open with Live Server". The address bar should say `http://127.0.0.1:5500/...` and not `file:///C:/...`.

<details>

<summary>A note on backticks</summary>

The regular lessons build URLs with backticks, like this:

```js
const url = `https://v2.api.noroff.dev/social/profiles/${username}`;
```

This lesson uses plain string joining instead:

```js
const url = 'https://jsonplaceholder.typicode.com/users/' + userId;
```

These do exactly the same thing. Backticks are tidier once the string gets long, and you will see them everywhere in real code and in the regular lessons, so do learn them. Joining with `+` is used here only because it is one less thing to think about while you are thinking about HTTP.

</details>

---

## Part 1 - GET, and actually looking at what arrived

You already know this one from Lesson 5.2. A GET request asks for something. It has no body. Anything extra you want to say goes in the URL.

Add this to `app.js`:

```js
async function loadChores() {
  statusText.textContent = 'Loading...';

  const response = await fetch(baseUrl + '/todos?userId=3');
  const chores = await response.json();

  console.log('Number of chores:', chores.length);
  console.log('First chore:', chores[0]);

  statusText.textContent = 'Loaded ' + chores.length + ' chores.';
  return chores;
}

loadButton.addEventListener('click', loadChores);
```

Click the button. You should see 20 chores logged.

The `?userId=3` part is a **query string**. It starts with a `?`, and it is made of `key=value` pairs joined by `&`. It is not a separate feature of `fetch` - it is just part of the URL. Which is exactly why it is the wrong place for a password: the URL ends up in your browser history, in the server's log files, and in anything you paste to a friend.

Now change it to `?userId=3&completed=false` and click again. Fewer chores, because you asked the server a narrower question. The chores it left out never travelled across the network at all.

### Tip: stop reading arrays of objects with `console.log`

When you log an array of 20 objects, the console gives you 20 collapsed triangles to click. `console.table` gives you a grid instead, with one row per item and one column per property.

Change the two log lines to:

```js
console.table(chores);
```

Now you can see all 20 chores, all four properties, at a glance. You can click a column heading to sort by it. This is easily the highest-value trick in Lesson 5.4, and it costs you five keystrokes more than `console.log`.

It works on any array of objects, which includes almost everything an API hands you.

<details>

<summary>Exercise 1: two ways to narrow a list</summary>

**Goal:** see the difference between asking the server for less and asking for everything and throwing most of it away.

1. Write an async function `countByServer()` that fetches `/todos?userId=3&completed=true` and logs how many items came back.
2. Write an async function `countByBrowser()` that fetches `/todos` (all 200 of them), then uses `filter` to keep only the ones where `userId` is 3 and `completed` is `true`, and logs how many are left.
3. Call both. Confirm they give the same number.
4. Open the Network tab in DevTools, reload, and click the button. Compare the **Size** column for the two requests.
5. Answer for yourself: which one would you rather run on a phone on a train?

</details>

<details>

<summary>Solution 1</summary>

```js
async function countByServer() {
  const response = await fetch(baseUrl + '/todos?userId=3&completed=true');
  const chores = await response.json();
  console.log('Filtered by the server:', chores.length);
}

async function countByBrowser() {
  const response = await fetch(baseUrl + '/todos');
  const allChores = await response.json();

  const mine = allChores.filter(function (chore) {
    return chore.userId === 3 && chore.completed === true;
  });

  console.log('Filtered in the browser:', mine.length);
}

countByServer();
countByBrowser();
```

Both log the same number.

The second request downloads all 200 items in order to keep a handful. On a fast laptop you will not notice. On a slow connection, with a list of 20,000 items instead of 200, you very much would. The rule of thumb: **if the server can filter it, let it.** You cannot always - not every API supports it, and the ones that do all spell it differently - so you check the API's documentation first.

</details>

---

## Part 2 - POST, and putting something in the box

To send data, `fetch` takes a **second argument**: an options object. That object is the label and the box.

```js
async function addChore(title) {
  const newChore = {
    userId: 3,
    title: title,
    completed: false,
  };

  const options = {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(newChore),
  };

  const response = await fetch(baseUrl + '/todos', options);
  const created = await response.json();

  console.log('Status was:', response.status);
  console.log('Server gave the new chore this id:', created.id);
  return created;
}

addChore('Descale the kettle');
```

Three things in that options object, and each one answers a different question.

**`method: 'POST'`** answers *what should the server do with this?* Create something new. `fetch` assumes `'GET'` when you leave this out, which is why Part 1 needed no options object at all.

**`body: JSON.stringify(newChore)`** answers *with what?* Note the `JSON.stringify`. An HTTP request can only carry text. Your JavaScript object is not text, it is a thing in your browser's memory. `JSON.stringify` turns it into a string that says the same thing. Leave it out and you will send the string `[object Object]` to a confused server.

**`headers`** answers *how should the server read the box?* Setting `Content-Type` to `application/json` tells the server the body is JSON and should be parsed as such. Without it the server may treat your careful JSON as a meaningless blob of text.

Notice the URL: it is `/todos`, the whole collection, not a particular item. That is deliberate. You are saying "add one to this pile". You do not name the new thing, because it does not exist yet - **the server chooses the id and tells you what it picked**. That is why POST responses matter: the reply contains information you did not have before.

Notice also the status: `201`, not `200`. `201 Created` is the specific "I made a new thing" success.

### Tip: the Network tab tells you what you actually sent

Right now you are trusting that your options object turned into the request you intended. Do not. Check.

1. Open DevTools and go to the **Network** tab.
2. Click the **Fetch/XHR** filter so you see your API calls and not fonts and images.
3. Reload and trigger the request.
4. Click the `todos` row that appears.

You now get several panels:

- **Headers** - under *General*, the full request URL, the request method and the status code. Under *Request Headers*, whether your `Content-Type` really got sent.
- **Payload** - the body you sent, parsed back into something readable. This is where you find out you stringified the wrong variable.
- **Response** - the raw body the server sent back, before your `.json()` touched it. When a request fails with a `400`, the explanation is almost always sitting here.

When an API call misbehaves, this tab answers the first question you should always ask: **is this my code being wrong, or my request being wrong, or the server being wrong?** Those are three very different problems and the Network tab tells them apart in about ten seconds.

<details>

<summary>Rabbit hole: why did the server give me id 201, and why is it still 201?</summary>

JSONPlaceholder has 200 todos, so a new one gets 201. Run `addChore` five times and you get 201 five times. On a real server you would get 201, 202, 203, and so on.

That is the faking showing through. Nothing is stored, so the server's idea of "the highest id so far" never moves.

Two useful lessons hide in this.

The first is that **you should never guess an id yourself**. It is tempting to write `const newId = chores.length + 1` and send that. Do not. The server is the only thing that knows what ids exist, especially when other people are using the same API at the same time. Send the data, let the server name it, use the name it gives back.

The second is that this is exactly the sort of surprise that makes a "working" tutorial break when you point it at a real API. When you swap JSONPlaceholder for a real backend, expect the ids to behave differently, and expect the thing you created to still be there when you ask for it again.

</details>

<details>

<summary>Exercise 2: send a comment, and check it in the Network tab</summary>

**Goal:** build an options object from scratch and verify it with DevTools rather than by hoping.

1. Write an async function `addComment()`.
2. Inside it, build an object with `postId` set to `5`, `name` set to anything, `email` set to anything, and `body` set to a sentence.
3. Build an options object that POSTs it to `https://jsonplaceholder.typicode.com/comments` with the right `Content-Type`.
4. Await the fetch, await `response.json()`, and log both `response.status` and the returned object.
5. Open the Network tab, filter to Fetch/XHR, run it, click the `comments` row, and check the **Payload** panel. Does it show the four properties you meant to send?
6. Now break it deliberately: remove `JSON.stringify` and send the raw object as the body. Run it again and look at Payload. What does the server receive?

</details>

<details>

<summary>Solution 2</summary>

```js
async function addComment() {
  const newComment = {
    postId: 5,
    name: 'Kettle update',
    email: 'lasse@example.com',
    body: 'Descaled at last. It only took three weeks of looking at it.',
  };

  const options = {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(newComment),
  };

  const response = await fetch(baseUrl + '/comments', options);
  const created = await response.json();

  console.log('Status:', response.status);
  console.log('Created:', created);
}

addComment();
```

Status is `201`, and the returned object is your four properties plus `id: 501`.

On step 6, with `body: newComment` instead of `body: JSON.stringify(newComment)`, the Payload panel shows `[object Object]`. That is JavaScript converting your object to a string the only way it knows how when something demands a string and you handed it an object. The server receives five useless characters. No error is thrown in your code, which is what makes this one nasty - it fails quietly, and only the Network tab tells you why.

</details>

---

## Part 3 - PUT and DELETE, which are the same parcel with a different label

Here is the whole of PUT:

```js
async function replaceChore(id, chore) {
  const options = {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(chore),
  };

  const response = await fetch(baseUrl + '/todos/' + id, options);
  const updated = await response.json();

  console.log('Status:', response.status);
  return updated;
}

replaceChore(5, {
  userId: 3,
  id: 5,
  title: 'Descale the kettle',
  completed: true,
});
```

Compare it with `addChore`. Two differences, and that is all.

**The method says `'PUT'`.** And **the URL names a specific item**: `/todos/5`, not `/todos`. That follows from what PUT means. You are not adding to a pile, you are replacing a particular thing, so you have to say which thing.

The word "replacing" is doing real work in that sentence. PUT sends the whole item, not the bit you changed. If you send only `{ completed: true }`, a strict server will take you at your word and the title will be gone. This is why the object above includes `userId`, `id` and `title` even though only `completed` is different.

DELETE is shorter still, because there is nothing to put in the box:

```js
async function removeChore(id) {
  const options = {
    method: 'DELETE',
  };

  const response = await fetch(baseUrl + '/todos/' + id, options);
  console.log('Status:', response.status);
}

removeChore(5);
```

No `body`, so no `JSON.stringify`, so no `Content-Type` - there is no content to describe. The URL says everything the server needs.

### Which of these can you safely do twice?

This is the idea the regular lesson calls **idempotency**, and it is worth having in plain words because it decides real things about your app.

Think of name tags at an event.

- **PUT** is writing "Alex" on a tag with a marker. Do it ten times and you have one tag saying Alex.
- **DELETE** is binning a tag. Do it ten times and the tag is gone, still just as gone.
- **POST** is taking a fresh blank tag off the stack. Do it ten times and you have ten tags.

So PUT and DELETE are safe to repeat, and POST is not. That is not trivia. It is why your browser warns you before re-submitting a form, and it is why a double-click on a "Place order" button can produce two orders while a double-click on "Save profile" cannot produce two profiles. When you build the checkout in your Course Assignment, this is the reason you disable the button while the request is in flight.

### Tip: `console.group` when one action makes several requests

Once an action fires two or three requests, your console turns into a wall of unlabelled lines. `console.group(label)` opens a collapsible, indented block, and `console.groupEnd()` closes it.

```js
async function replaceAndRemove(id) {
  console.group('Chore ' + id);

  console.log('Step 1: replacing');
  await replaceChore(id, {
    userId: 3,
    id: id,
    title: 'Temporary',
    completed: false,
  });

  console.log('Step 2: removing');
  await removeChore(id);

  console.groupEnd();
}

replaceAndRemove(7);
```

One tidy foldable block per chore instead of six loose lines. Put the `console.groupEnd()` somewhere it will always run - the regular lesson uses a `finally` block for exactly this reason, and you will meet `finally` properly in Lesson 5.5.

<details>

<summary>Rabbit hole: PUT sends everything, PATCH sends the difference</summary>

Sending the entire object just to flip one boolean is a bit silly, and HTTP has an answer: `PATCH`. It means "here are the bits that changed, leave everything else alone".

```js
const options = {
  method: 'PATCH',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ completed: true }),
};
```

JSONPlaceholder supports it, and so do many real APIs. It is not in your syllabus, and the Noroff API you will use for the Course Assignment expects PUT, so use PUT there. It is worth knowing the word exists so that you recognise it when you meet it, and so you understand why some APIs are relaxed about receiving a partial object under PUT: they are quietly treating your PUT as a PATCH.

</details>

<details>

<summary>Exercise 3: tick a chore off</summary>

**Goal:** fetch one item, change one property, send the whole thing back.

1. Write an async function `toggleChore(id)`.
2. GET the single chore from `/todos/{id}`. Note that this returns one object, not an array.
3. Log the chore's `completed` value before you change anything.
4. Flip it: `chore.completed = !chore.completed;`
5. PUT the whole modified object back to `/todos/{id}`.
6. Log the status code and the object the server sends back. Confirm the flipped value came back flipped.
7. Run it twice with the same id. Does the second run flip it back?

</details>

<details>

<summary>Solution 3</summary>

```js
async function toggleChore(id) {
  const getResponse = await fetch(baseUrl + '/todos/' + id);
  const chore = await getResponse.json();

  console.log('Before:', chore.completed);

  chore.completed = !chore.completed;

  const options = {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(chore),
  };

  const putResponse = await fetch(baseUrl + '/todos/' + id, options);
  const updated = await putResponse.json();

  console.log('Status:', putResponse.status);
  console.log('After:', updated.completed);
}

toggleChore(1);
```

On step 7: no, it does not flip back. Chore 1 starts as `completed: false`, so every single run logs `Before: false` and `After: true`.

That is the faked-writes behaviour again, and it makes a genuinely important point. Your PUT succeeded - status 200, correct object returned - and yet the next GET shows the old value. **A successful response tells you the server accepted your request. Only a fresh read tells you the data changed.** Against a real API those two usually agree. When they do not, you get a bug where the screen says one thing and the database says another, and it is a horrible one to track down.

Note also that this took two round trips: one to read, one to write. That is normal for this pattern. If you already have the chore in a variable from an earlier fetch, you can skip the first one.

</details>

---

## Part 4 - The number before the data

Every response starts with a three-digit number, and the first digit is the whole story.

| Range | Meaning | Whose problem |
| --- | --- | --- |
| `1xx` | Hold on, still going | Nobody's. You will rarely see these. |
| `2xx` | Worked | Nobody's. |
| `3xx` | It moved, look over there | Usually handled for you. |
| `4xx` | Your request was wrong | Yours. Fix the request. |
| `5xx` | The server fell over | Theirs. Try later, tell the user. |

The ones you will actually meet:

- **`200 OK`** - the standard success. Used by GET, and by PUT on most APIs.
- **`201 Created`** - a successful POST. There is a new thing, and its details are in the response.
- **`204 No Content`** - it worked and there is nothing to send back. Common for DELETE.
- **`400 Bad Request`** - the server understood you and does not like what you sent. Missing field, wrong type, password too short. The response body normally says which.
- **`401 Unauthorized`** - the server does not know who you are. No token, wrong token, expired token.
- **`403 Forbidden`** - the server knows exactly who you are and you are still not allowed. Deleting somebody else's post.
- **`404 Not Found`** - no such thing at that address. Wrong URL, or wrong id.
- **`500 Internal Server Error`** - something broke on their end.

Now the part that catches everyone. From Lesson 5.2, in the envelope form: **an envelope saying "no such person" still arrived safely.**

```js
async function showStatus() {
  const response = await fetch(baseUrl + '/todos/99999');
  console.log('Did fetch throw?', 'No, we got here.');
  console.log('response.ok:', response.ok);
  console.log('response.status:', response.status);
}

showStatus();
```

There is no chore 99999. `fetch` does not throw. Your `catch` block never runs. `response.ok` is `false` and `response.status` is `404`, and unless you check one of those yourself, your code sails happily on and tries to read `.title` off an error object.

`fetch` only rejects when the request never made it - no connection, DNS failure, blocked by CORS. **A rejected promise means the letter never arrived. A 404 means it arrived and said no.**

So the check you have to write yourself:

```js
if (!response.ok) {
  throw new Error('Request failed with status ' + response.status);
}
```

And one more trap, this one specific to DELETE. If the status is `204`, there is no body, and `await response.json()` on an empty body throws. So check the status **before** you parse:

```js
async function removeChoreProperly(id) {
  const response = await fetch(baseUrl + '/todos/' + id, { method: 'DELETE' });

  if (response.status === 204) {
    console.log('Deleted, and nothing to read.');
    return;
  }

  if (!response.ok) {
    throw new Error('Delete failed with status ' + response.status);
  }

  const result = await response.json();
  console.log('Deleted, server said:', result);
}

removeChoreProperly(5);
```

Run that. JSONPlaceholder answers `200` with an empty object `{}`, not `204`. Which is the point: **different APIs answer differently, and you check rather than assume.** Both branches are there because both are plausible.

### Tip: use the right console level and let the colours work for you

`console.log`, `console.warn` and `console.error` all print. The difference is that warnings are yellow, errors are red with a stack trace showing where it came from, and the console's filter bar lets you hide everything except errors.

That last part is the real benefit. In a busy app you can switch to errors only and see the three lines that matter instead of the four hundred that do not. This only works if you have been honest about which is which - if everything is a `console.log`, the filter has nothing to work with.

```js
async function loadWithLevels(url) {
  const response = await fetch(url);

  if (response.status === 404) {
    console.warn('Nothing at ' + url + ' - showing an empty list.');
    return [];
  }

  if (!response.ok) {
    console.error('Request to ' + url + ' failed with ' + response.status);
    return [];
  }

  const data = await response.json();
  console.log('Loaded from ' + url);
  return data;
}

loadWithLevels(baseUrl + '/todos?userId=3');
loadWithLevels(baseUrl + '/todos/99999');
```

Rough guide: `error` for something that broke, `warn` for something odd that you handled, `log` for progress.

<details>

<summary>Exercise 4: a function that reads the number</summary>

**Goal:** turn status codes into something a user could read.

1. Write a function `describeStatus(status)` that takes a number and returns a short sentence.
2. Handle `200`, `201`, `204`, `400`, `401`, `403` and `404` individually.
3. For anything `500` or above, return a single message about the server having trouble.
4. For anything else, return something honest that includes the number.
5. Write an async function `checkUrl(url)` that fetches a URL and logs `describeStatus(response.status)`.
6. Test it against `/todos/1` (exists) and `/todos/99999` (does not).

</details>

<details>

<summary>Solution 4</summary>

```js
function describeStatus(status) {
  if (status === 200) {
    return 'Worked. Here is your data.';
  }
  if (status === 201) {
    return 'Created. The new item is in the response.';
  }
  if (status === 204) {
    return 'Worked. Nothing to send back.';
  }
  if (status === 400) {
    return 'Something was wrong with what we sent.';
  }
  if (status === 401) {
    return 'You need to log in first.';
  }
  if (status === 403) {
    return 'You are logged in, but not allowed to do that.';
  }
  if (status === 404) {
    return 'We could not find that.';
  }
  if (status >= 500) {
    return 'The server is having trouble. Please try again later.';
  }
  return 'Unexpected response (' + status + ').';
}

async function checkUrl(url) {
  const response = await fetch(url);
  console.log(url, '->', response.status, describeStatus(response.status));
}

checkUrl(baseUrl + '/todos/1');
checkUrl(baseUrl + '/todos/99999');
```

A `switch` would work just as well here, and is arguably tidier for a long list of exact values. The `if (status >= 500)` line is the reason this version uses `if` - a range is awkward in a `switch`.

Notice that this returns strings rather than logging them. That makes it usable from anywhere: log it while developing, put it in `statusText.textContent` for the user. A function that both decides something and prints it is harder to reuse than one that just decides.

</details>

---

## Part 5 - One helper, four methods

Look back at `loadChores`, `addChore`, `replaceChore` and `removeChore`. Nearly identical. The differences are: which method, whether there is a body, and which URL.

So make that the function's parameters.

```js
async function sendRequest(path, method, data) {
  const options = {
    method: method,
    headers: {},
  };

  if (data) {
    options.headers['Content-Type'] = 'application/json';
    options.body = JSON.stringify(data);
  }

  const response = await fetch(baseUrl + path, options);

  if (!response.ok) {
    throw new Error(method + ' ' + path + ' failed: ' + response.status);
  }

  if (response.status === 204) {
    return null;
  }

  return await response.json();
}
```

That is the whole of HTTP for our purposes. Now all four verbs are one line each:

```js
async function demo() {
  const chores = await sendRequest('/todos?userId=3', 'GET');
  console.log('Got', chores.length, 'chores');

  const created = await sendRequest('/todos', 'POST', {
    userId: 3,
    title: 'Sort the recycling',
    completed: false,
  });
  console.log('Created id', created.id);

  const updated = await sendRequest('/todos/1', 'PUT', {
    userId: 1,
    id: 1,
    title: 'Sort the recycling',
    completed: true,
  });
  console.log('Updated:', updated.completed);

  await sendRequest('/todos/1', 'DELETE');
  console.log('Deleted');
}

demo();
```

Two details worth pausing on.

`if (data)` is doing the work of deciding whether this is a request with a box. GET and DELETE call the function without a third argument, so `data` is `undefined`, which is falsy, so no body and no `Content-Type` get added. POST and PUT pass an object, which is truthy.

And `sendRequest` **throws** rather than returning something odd, because that lets the caller wrap the whole sequence in one `try...catch` instead of checking after every line. That is the shape you will want in the Course Assignment, and Lesson 5.5 goes into it properly.

### Tip: a breakpoint beats five `console.log`s when you need to see what you built

`sendRequest` assembles that options object out of three arguments, which means there is now a gap between what you asked for and what actually goes out over the wire. A breakpoint closes it. It shows you the finished object, plus every other variable in scope, without you adding a single line of code.

1. DevTools, **Sources** tab.
2. Find `app.js` in the file list on the left.
3. Click the line number of the `await fetch(...)` line inside `sendRequest`. A blue marker appears.
4. Trigger a request. Everything freezes just **before** that line runs.
5. Look at the **Scope** pane on the right. Expand `options`. Expand `headers`. Read the actual values.

You will see immediately whether `data` arrived as you expected, whether `Content-Type` got added or skipped, and what the joined-up URL really says. All of which you could get from `console.log`, one variable at a time, with a reload between each.

Then use **Step over** (F10) to run one line at a time. Over an `await`, the debugger goes quiet while the request is in flight and re-pauses on the following line once the response arrives - which is a rather nice way of watching the "puts the pen down and picks it up again" idea from Lesson 5.1 happen in front of you.

### Tip: throttle the network so you can see your own loading states

Your API calls come back in 80 milliseconds on a good connection, which means your "Loading..." message flashes past too fast to check. Meanwhile the person using your site on a train sees it for four seconds.

In the **Network** tab there is a throttling dropdown, normally set to "No throttling". Set it to **Slow 3G** and use your page.

Three things you will now notice that you could not before:

1. Whether your loading message appears at all, and whether it goes away afterwards.
2. What happens if you click the button five times while the first request is in flight. Five requests, five responses, arriving in whatever order they like. This is why you disable the button.
3. Whether the page looks broken or merely busy while it waits.

There is also an **Offline** checkbox in the same row. Tick it and click your button: `fetch` rejects, your `catch` runs, and you find out whether you wrote a useful message or let the page sit on "Loading..." forever.

Ten minutes with these two controls will improve your Course Assignment more than an hour of styling.

<details>

<summary>Exercise 5: put the board on the page</summary>

**Goal:** wire `sendRequest` to the DOM, with a loading state and a disabled button.

1. Write a `renderChores(chores)` function that empties `choreList` with `choreList.innerHTML = '';` then builds one `li` per chore with `createElement` and `appendChild`.
2. Each `li` should show the chore's title. If `completed` is `true`, add the `done` class with `setAttribute('class', 'done')`.
3. Write an async function `showChores()` that:
   - sets `statusText.textContent` to `'Loading...'` and sets `loadButton.disabled = true`
   - calls `sendRequest('/todos?userId=3', 'GET')` inside a `try`
   - calls `renderChores` with the result and reports how many arrived
   - in the `catch`, puts a readable message in `statusText`
   - re-enables the button afterwards, on both paths
4. Attach it to the button's click.
5. Test it on Slow 3G, then with Offline ticked.

</details>

<details>

<summary>Solution 5</summary>

```js
function renderChores(chores) {
  choreList.innerHTML = '';

  for (const chore of chores) {
    const item = document.createElement('li');
    item.textContent = chore.title;

    if (chore.completed) {
      item.setAttribute('class', 'done');
    }

    choreList.appendChild(item);
  }
}

async function showChores() {
  statusText.textContent = 'Loading...';
  loadButton.disabled = true;

  try {
    const chores = await sendRequest('/todos?userId=3', 'GET');
    renderChores(chores);
    statusText.textContent = 'Showing ' + chores.length + ' chores.';
  } catch (error) {
    console.error(error);
    statusText.textContent = 'Could not load the chores. Please try again.';
    choreList.innerHTML = '';
  }

  loadButton.disabled = false;
}

loadButton.addEventListener('click', showChores);
```

The `loadButton.disabled = false;` sits after the whole `try...catch`, not inside the `try`. If it were inside, a failed request would leave the button disabled forever and the user stuck. This is the job that `finally` does more neatly, which you will see in Lesson 5.5.

`choreList.innerHTML = '';` in the catch block matters too. Without it, a failed reload leaves the previous list sitting there under an error message, which reads as though those chores are current when they are not.

With Offline ticked you should see the error message, and `console.error` should show a `TypeError: Failed to fetch`. That is the "letter never arrived" case from Part 4.

</details>

<details>

<summary>Exercise 6: spot the bugs, no code required</summary>

Each of these has at least one bug. Name it, and say what status code or symptom you would expect. Assume a real API rather than JSONPlaceholder, so that writes actually stick.

```js
// A
const optionsA = {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: { title: 'Hello' },
};

// B
const optionsB = {
  method: 'POST',
  body: JSON.stringify({ title: 'Hello' }),
};

// C
const optionsC = {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ completed: true }),
};
// used with: fetch(baseUrl + '/todos', optionsC)

// D
const optionsD = {
  method: 'DELETE',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ id: 12 }),
};
// used with: fetch(baseUrl + '/todos', optionsD)
```

</details>

<details>

<summary>Solution 6</summary>

**A** - no `JSON.stringify`. The body becomes the string `[object Object]`. Expect `400 Bad Request`, and a server complaining that required fields are missing. Nothing throws in your own code, so the Network tab's Payload panel is how you find it.

**B** - a body with no `Content-Type` header. The server has a JSON string and no reason to believe it is JSON, so it may refuse to parse it and report the fields as missing. Expect `400`, sometimes `415 Unsupported Media Type`. Some servers guess correctly and it works anyway, which is worse, because then it breaks the day you change servers.

**C** - two problems. The URL points at the collection rather than a specific item, so there is nothing to update; expect `404`, or a `400` on a server that objects to PUT on a collection. And the body sends only `completed`, so even against the right URL a strict server would wipe the title. PUT replaces.

**D** - the id is in the body instead of the URL. It should be `fetch(baseUrl + '/todos/12', optionsD)` with no body at all. Many servers ignore a body on DELETE entirely, so this may appear to work while deleting nothing - and "appears to work but does nothing" is the worst outcome of the four, because you will not go looking for it.

</details>

---

## Self study task: a post browser with a working create button

**Estimated time:** 45 to 60 minutes.

Build a small page against `/users` and `/posts` that reads and writes.

### Brief

1. On page load, GET `/users` and fill a `<select>` with the ten users. Each option's value should be the user's `id` and its text the user's `name`.
2. When the selection changes, GET `/posts?userId={id}` and render the posts as a list, each showing its title.
3. Give each rendered post a **Delete** button. Clicking it should DELETE `/posts/{postId}` and, on success, remove that item from the page.
4. Add a form with a title field and a body field. Submitting it should POST a new post for the currently selected user, and add it to the top of the list using the id the server returns.
5. Show a status line throughout: loading, how many posts, or a readable error.
6. Disable the submit button while the POST is in flight.
7. Use `sendRequest` from Part 5 for all four calls.

### Requirements

- Everything goes through `sendRequest`. If you find yourself writing a second options object, stop and pass an argument instead.
- Every request path is wrapped so that a failure puts a message on the page, not just in the console.
- No page reloads. `event.preventDefault()` on the form.
- Test it with Slow 3G on, and with Offline on.

### Things that will bite you

- The form's submit event fires a page reload unless you prevent it.
- Rebuilding the user dropdown whenever the list re-renders will silently reset the user's choice. Build it once.
- After a POST, the server hands back id 101 every time. If you use the id as a key for the delete button, deleting one new post will look like it deletes all of them. Think about what you would do differently against a real API.

<details>

<summary>Solution: HTML</summary>

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Post browser</title>
    <link rel="stylesheet" href="style.css" />
    <script src="app.js" defer></script>
  </head>
  <body>
    <h1>Post browser</h1>

    <label for="userSelect">Author</label>
    <select id="userSelect">
      <option value="">Loading users...</option>
    </select>

    <p id="status">Starting up.</p>

    <form id="newPostForm">
      <label for="titleField">Title</label>
      <input id="titleField" type="text" required />

      <label for="bodyField">Body</label>
      <textarea id="bodyField" required></textarea>

      <button id="submitButton" type="submit">Add post</button>
    </form>

    <ul id="postList"></ul>
  </body>
</html>
```

</details>

<details>

<summary>Solution: JavaScript</summary>

```js
const baseUrl = 'https://jsonplaceholder.typicode.com';

const userSelect = document.querySelector('#userSelect');
const statusText = document.querySelector('#status');
const postList = document.querySelector('#postList');
const newPostForm = document.querySelector('#newPostForm');
const titleField = document.querySelector('#titleField');
const bodyField = document.querySelector('#bodyField');
const submitButton = document.querySelector('#submitButton');

let posts = [];

async function sendRequest(path, method, data) {
  const options = {
    method: method,
    headers: {},
  };

  if (data) {
    options.headers['Content-Type'] = 'application/json';
    options.body = JSON.stringify(data);
  }

  const response = await fetch(baseUrl + path, options);

  if (!response.ok) {
    throw new Error(method + ' ' + path + ' failed: ' + response.status);
  }

  if (response.status === 204) {
    return null;
  }

  return await response.json();
}

function renderPosts() {
  postList.innerHTML = '';

  for (const post of posts) {
    const item = document.createElement('li');

    const heading = document.createElement('h2');
    heading.textContent = post.title;
    item.appendChild(heading);

    const deleteButton = document.createElement('button');
    deleteButton.textContent = 'Delete';
    deleteButton.setAttribute('type', 'button');

    deleteButton.addEventListener('click', function () {
      deletePost(post, deleteButton);
    });

    item.appendChild(deleteButton);
    postList.appendChild(item);
  }
}

async function deletePost(post, button) {
  button.disabled = true;
  statusText.textContent = 'Deleting...';

  try {
    await sendRequest('/posts/' + post.id, 'DELETE');

    posts = posts.filter(function (item) {
      return item !== post;
    });

    renderPosts();
    statusText.textContent = 'Deleted. ' + posts.length + ' posts left.';
  } catch (error) {
    console.error(error);
    statusText.textContent = 'Could not delete that post.';
    button.disabled = false;
  }
}

async function loadUsers() {
  try {
    const users = await sendRequest('/users', 'GET');

    userSelect.innerHTML = '';

    for (const user of users) {
      const option = document.createElement('option');
      option.value = user.id;
      option.textContent = user.name;
      userSelect.appendChild(option);
    }

    statusText.textContent = 'Pick an author.';
    await loadPosts();
  } catch (error) {
    console.error(error);
    statusText.textContent = 'Could not load the list of authors.';
  }
}

async function loadPosts() {
  const userId = userSelect.value;

  if (!userId) {
    return;
  }

  statusText.textContent = 'Loading posts...';
  postList.innerHTML = '';

  try {
    posts = await sendRequest('/posts?userId=' + userId, 'GET');
    renderPosts();
    statusText.textContent = 'Showing ' + posts.length + ' posts.';
  } catch (error) {
    console.error(error);
    posts = [];
    postList.innerHTML = '';
    statusText.textContent = 'Could not load posts for that author.';
  }
}

async function addPost(event) {
  event.preventDefault();

  const userId = userSelect.value;

  if (!userId) {
    statusText.textContent = 'Pick an author first.';
    return;
  }

  submitButton.disabled = true;
  statusText.textContent = 'Saving...';

  try {
    const created = await sendRequest('/posts', 'POST', {
      userId: Number(userId),
      title: titleField.value,
      body: bodyField.value,
    });

    posts.unshift(created);
    renderPosts();

    titleField.value = '';
    bodyField.value = '';
    statusText.textContent = 'Saved with id ' + created.id + '.';
  } catch (error) {
    console.error(error);
    statusText.textContent = 'Could not save that post.';
  }

  submitButton.disabled = false;
}

userSelect.addEventListener('change', loadPosts);
newPostForm.addEventListener('submit', addPost);

loadUsers();
```

</details>

<details>

<summary>Notes on the solution</summary>

**The dropdown is built once.** `loadUsers` runs at startup and never again. `loadPosts` only touches `postList`. If the dropdown were rebuilt on every render, the user's choice would be silently reset every time a post loaded - a bug that is maddening to find because nothing errors.

**Delete buttons close over the post object, not the id.** Each `deleteButton` handler captures its own `post` variable, and the filter removes by identity rather than by id:

```js
posts = posts.filter(function (item) {
  return item !== post;
});
```

This is the fix for the "delete one new post, they all vanish" trap. Because the fake server hands out id 101 to every new post, filtering on `item.id !== post.id` would remove every newly created post at once. Against a real API the ids would be unique and either version would work - but writing the version that does not depend on that is free, so write it.

**`Number(userId)` on the way out.** A `<select>` value is always a string, even when you set it from a number. The API expects a number. Converting at the boundary, where the data leaves your code, keeps the rest of the app honest.

**`unshift` puts the new post on top.** It is the counterpart to `push`, adding to the front of an array instead of the back.

**Every path re-enables its button.** `submitButton.disabled = false;` sits after the `try...catch`, and `deletePost` re-enables its button in the `catch` (but not on success, since that button is about to be removed from the page anyway).

</details>

---

## What to take away

Three things, and the third is the one that will save you the most time.

**The four methods are one shape.** URL says which, method says what to do, headers say how to read the box, body is the box. GET and DELETE send no box. POST creates and lets the server name the result. PUT replaces and needs you to name the target. Repeating a PUT or DELETE is safe; repeating a POST is not.

**Check the number before you trust the data.** `fetch` only rejects when the request never arrived. A 404 arrives perfectly well. `if (!response.ok)` is not boilerplate, it is the whole difference between an app that fails clearly and one that fails weirdly three functions later.

**Use the tools before you start guessing.** `console.table` for any array of objects. The Network tab's Payload panel when you suspect you sent the wrong thing. Throttling to see your own loading states. A breakpoint and the Scope pane when you need to know what a variable actually contains. Each of these replaces about twenty minutes of adding `console.log` lines and reloading.

Lesson 5.5 picks up the loose thread: what to do about all these thrown errors, and how `try`, `catch` and `finally` fit together properly.
