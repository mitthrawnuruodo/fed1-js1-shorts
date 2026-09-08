# The ticket, not the ID card

**Estimated time:** about 1 hour, plus 30 minutes for the self study task.  
**Prerequisites:** Lessons 5.1 and 5.2, and the extra lesson "Four verbs and one parcel". You need `fetch` with an options object, `JSON.stringify`, status codes, and `response.ok`.

## Why this is separate

The extra lesson on HTTP methods used JSONPlaceholder, which lets anybody do anything. That was deliberate - nothing stood between you and the parcel idea. But it left out the one header that every real API expects, including the Noroff API you need for the Course Assignment.

This lesson is only about that header. It is short, and it has one job: to make a `401` something you have *caused on purpose* three or four times, rather than something that happens to you at eleven at night the week the assignment is due.

Two sentences hold it together.

**One: a token is a ticket, not an ID card.** The server does not check that you are you. It checks that you are holding the ticket. Everything awkward about tokens follows from that one fact.

**Two: logging in is a POST, and being logged in is a header.** There is no hidden "logged in" state in your browser. You send a username and password, you get a string back, and from then on you paste that string onto every request. That is genuinely all it is.

### The API we will use

**DummyJSON**, at `https://dummyjson.com`. It has a real login endpoint that hands back a real token, real protected endpoints that reject you without one, and - the part that makes it worth using here - **tokens that you can make expire in one minute**. That last feature turns the most confusing thing about authentication into something you can watch happen.

The login details are published on purpose. We will use:

```
username: emilys
password: emilyspass
```

Any user from `https://dummyjson.com/users` works, and the password is always the first name in lower case with `pass` on the end.

---

## Before you start

Two files.

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Tokens</title>
    <script src="app.js" defer></script>
  </head>
  <body>
    <h1>Tokens</h1>
    <p id="status">Not logged in.</p>
    <button id="loginButton" type="button">Log in</button>
    <button id="profileButton" type="button">Who am I?</button>
  </body>
</html>
```

`app.js`:

```js
const baseUrl = 'https://dummyjson.com';

const statusText = document.querySelector('#status');
const loginButton = document.querySelector('#loginButton');
const profileButton = document.querySelector('#profileButton');

let accessToken = null;
```

Open it with Live Server. Keep the console open throughout - this lesson is mostly about reading responses.

---

## Part 1 - Logging in is just a POST

Look closely at this and notice how ordinary it is.

```js
async function logIn() {
  const options = {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      username: 'emilys',
      password: 'emilyspass',
    }),
  };

  const response = await fetch(baseUrl + '/auth/login', options);
  const result = await response.json();

  console.log('Status:', response.status);
  console.log('What came back:', result);

  accessToken = result.accessToken;
  statusText.textContent = 'Logged in as ' + result.firstName + '.';
}

loginButton.addEventListener('click', logIn);
```

There is nothing new in it. A POST, a `Content-Type` header, a stringified body. Exactly the shape from the previous extra lesson. **Logging in is not a special kind of request.** It is a POST to an endpoint that happens to answer with a token.

Click the button and look at what came back. Along with the user's name and email there is `accessToken`, and it is a long ugly string in three parts separated by full stops.

That string is now the only thing standing between you and the protected endpoints. Not your password - the password did its job and is finished. From here on, the token is what matters.

Try it with the wrong password. Change `emilyspass` to `wrong`, click again, and read the status: `400`, and a body explaining that the credentials are invalid. Note that this is not a `401`. A failed *login* is a bad request; a `401` is what you get later, when you try to use a protected endpoint without a valid ticket.

<details>

<summary>Rabbit hole: what is actually inside that token?</summary>

Those three dot-separated chunks make it a **JWT** (JSON Web Token). The middle chunk is not encrypted. It is just JSON that has been encoded so it survives being put in a header, and **anyone who has the token can read it**. Paste one into `https://jwt.io` and you will see the user id, the username, and two timestamps: when it was issued and when it stops working.

Two things follow, and the second one catches people out.

Because it is readable, **nothing secret ever goes inside a token**. If you ever build a back end, do not put anything in there you would not print on a poster.

Because it is readable *but signed*, you cannot usefully change it. Edit one character and the signature no longer matches, and the server rejects it. So a token proves the server issued it - but only the server can check that. Reading `"role": "admin"` out of a token in your front-end code and deciding to show the admin buttons is not security, it is decoration. The real check has to happen on the server, every time. Front-end code decides what to *show*; the server decides what is *allowed*.

This is not in your syllabus. It is here because "the token is just readable JSON" removes most of the mystery.

</details>

---

## Part 2 - Being logged in is just a header

Now use it:

```js
async function showProfile() {
  const options = {
    method: 'GET',
    headers: {
      Authorization: 'Bearer ' + accessToken,
    },
  };

  const response = await fetch(baseUrl + '/auth/me', options);
  const result = await response.json();

  console.log('Status:', response.status);
  console.log('Profile:', result);
}

profileButton.addEventListener('click', showProfile);
```

Click **Log in**, then **Who am I?**. You get the profile.

The header value has a precise shape: the word `Bearer`, then **one space**, then the token. `Bearer` is the scheme name - it tells the server how to interpret what follows.

No `Content-Type` here, because a GET has no body to describe. Authentication and content are separate concerns; adding `Content-Type` to a GET is harmless but meaningless.

### Now break it three ways on purpose

This is the actual point of the lesson. Do all three.

**One: no token at all.** Reload the page and click **Who am I?** *without* logging in first. `accessToken` is still `null`, so the header reads `Bearer null`. Read the status.

**Two: the missing space.** Log in properly, then change the header to `'Bearer' + accessToken` - no space. Click again. Read the status.

**Three: a mangled token.** Log in, then add a stray character:

```js
Authorization: 'Bearer ' + accessToken + 'x',
```

Read the status.

All three give you a `401`, and none of them tells you which mistake you made. That is what makes the missing space such a good bug: the code looks right, and the server's answer is identical to the answer it gives when you send nothing at all.

**Which is why you look, rather than guess.** Open DevTools, **Network** tab, filter to **Fetch/XHR**, click the `me` request, and read **Request Headers**. The value of `Authorization` is right there. `Bearer null` and `BearereyJhbGci...` are both obvious the moment you actually look at them, and invisible if you only stare at your code.

<details>

<summary>Exercise 1: three ways to fail, one status code</summary>

**Goal:** produce each failure deliberately and confirm what the server does and does not tell you.

1. Write an async function `tryProfile(headerValue)` that takes a complete `Authorization` header value as a string, sends a GET to `/auth/me` with it, and logs the status and the response body.
2. Call it four times: with a correct header, with `'Bearer null'`, with a correct token but no space after `Bearer`, and with a correct token plus a stray character.
3. For each one, note the status and the message.
4. Then answer: could you tell these apart from the response alone?

</details>

<details>

<summary>Solution 1</summary>

```js
async function tryProfile(headerValue) {
  const options = {
    method: 'GET',
    headers: {
      Authorization: headerValue,
    },
  };

  const response = await fetch(baseUrl + '/auth/me', options);
  const result = await response.json();

  console.log(response.status, '|', headerValue.slice(0, 25), '|', result.message || 'ok');
}

async function runAll() {
  await logIn();

  await tryProfile('Bearer ' + accessToken);
  await tryProfile('Bearer null');
  await tryProfile('Bearer' + accessToken);
  await tryProfile('Bearer ' + accessToken + 'x');
}

runAll();
```

The first gives `200`. The other three all give `401`, and the messages are about the token being missing or invalid - nothing that distinguishes "you forgot a space" from "you never logged in".

So no, you cannot tell them apart from the response. **The information you need is in the request, not the response**, which is why the Network tab's Request Headers panel is the first place to look when a `401` surprises you.

`headerValue.slice(0, 25)` just keeps the log readable - a full token is about 200 characters and would swamp the console.

</details>

<details>

<summary>Rabbit hole: 401 and 403 are not the same disappointment</summary>

Both mean "no". They mean different noes, and mixing them up leads to genuinely bad user experience.

**`401 Unauthorized`** means *I do not know who you are.* No token, bad token, expired token. The fix is to log in. Sending the user to a login screen is the right response.

**`403 Forbidden`** means *I know exactly who you are, and no.* Your token is perfectly valid. You are trying to delete somebody else's post. Sending this user to a login screen is useless and infuriating - they are already logged in, and logging in again changes nothing. The right response is a message explaining they are not allowed.

The names are historically backwards, which does not help. `401` is about authentication (who are you), `403` is about authorisation (what may you do). If you handle both with "please log in", one of your two error paths is a dead end.

</details>

---

## Part 3 - Tokens go stale, and you can watch it happen

Here is the thing that makes authentication different from every other header you will ever send: **the same code, unchanged, stops working after a while.**

Your token has an expiry timestamp baked into it. DummyJSON defaults to 60 minutes, but it lets you ask for something shorter, which is how we can see this in under a minute.

Change the login body:

```js
body: JSON.stringify({
  username: 'emilys',
  password: 'emilyspass',
  expiresInMins: 1,
}),
```

Now:

1. Click **Log in**.
2. Click **Who am I?** straight away. `200`, profile, all fine.
3. Wait a bit over a minute. Make a cup of tea.
4. Click **Who am I?** again. Change nothing.

`401`.

Nothing in your code changed. The token in the variable is the same token. It simply stopped being valid, and the only way you find out is by making a request and being turned away.

That is the whole lesson in one experiment, and it is why authentication is not just "add a header once and forget it". Your app has to cope with a token going stale mid-session. In practice that means: whenever a request comes back `401`, treat it as *the user is no longer logged in*, throw the token away, and get them to log in again.

```js
async function authFetch(path) {
  const options = {
    method: 'GET',
    headers: {
      Authorization: 'Bearer ' + accessToken,
    },
  };

  const response = await fetch(baseUrl + path, options);

  if (response.status === 401) {
    accessToken = null;
    statusText.textContent = 'Your session has expired. Please log in again.';
    throw new Error('Not logged in');
  }

  if (!response.ok) {
    throw new Error('Request failed: ' + response.status);
  }

  return await response.json();
}
```

Two things worth noticing.

The `401` gets its **own branch, above the general `!response.ok` check**. Order matters - a `401` is not ok, so a single `!response.ok` check would swallow it and you would lose the chance to react specifically.

And it **clears `accessToken`**. If you leave the dead token sitting in the variable, every subsequent request keeps sending it and keeps failing, and your app sits there insisting it is logged in while the server disagrees.

Set `expiresInMins` back to something sensible when you have finished experimenting.

<details>

<summary>Exercise 2: expire on purpose, recover gracefully</summary>

**Goal:** make expiry visible on the page instead of only in the console.

1. Use `expiresInMins: 1` in your login.
2. Rewrite `showProfile` to use `authFetch('/auth/me')` inside a `try...catch`.
3. On success, put the user's first name and email in `statusText`.
4. On failure, leave whatever message `authFetch` set, and log the error.
5. Also disable the **Who am I?** button whenever `accessToken` is `null`, and enable it after a successful login. Write one small function that does this and call it from both places.
6. Test the whole cycle: log in, check, wait, check again, log in again.

</details>

<details>

<summary>Solution 2</summary>

```js
function updateButtons() {
  profileButton.disabled = accessToken === null;
}

async function logIn() {
  const options = {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      username: 'emilys',
      password: 'emilyspass',
      expiresInMins: 1,
    }),
  };

  const response = await fetch(baseUrl + '/auth/login', options);
  const result = await response.json();

  if (!response.ok) {
    statusText.textContent = 'Could not log in: ' + result.message;
    return;
  }

  accessToken = result.accessToken;
  statusText.textContent = 'Logged in as ' + result.firstName + '.';
  updateButtons();
}

async function showProfile() {
  try {
    const me = await authFetch('/auth/me');
    statusText.textContent = me.firstName + ' (' + me.email + ')';
  } catch (error) {
    console.error(error);
    updateButtons();
  }
}

loginButton.addEventListener('click', logIn);
profileButton.addEventListener('click', showProfile);

updateButtons();
```

The `updateButtons()` call at the very bottom matters: it runs once on page load, so the button starts disabled rather than looking available and then failing.

Note that `logIn` checks `response.ok` before touching `result.accessToken`. Without that check a failed login would set `accessToken` to `undefined`, and `undefined` is not `null`, so `updateButtons` would happily enable the button for a session that does not exist.

The `catch` calls `updateButtons()` because `authFetch` has just set `accessToken` to `null` - the UI needs to catch up with that.

</details>

<details>

<summary>Rabbit hole: refresh tokens, or why real apps do not log you out every hour</summary>

Short expiry is good for security and terrible for users. The usual answer is a **second** token.

Look again at what the login response contained: `accessToken` **and** `refreshToken`. The access token is the one you attach to requests, and it expires quickly. The refresh token lives much longer and does exactly one thing - it can be traded at `/auth/refresh` for a fresh access token, without asking the user for their password again.

So a real app, on catching a `401`, does not immediately throw the user out. It quietly tries the refresh endpoint first, and only sends them to the login screen if that fails too. That is why you can stay logged in to a site for weeks while its access tokens are expiring every hour behind the scenes.

Well outside your syllabus, and the Noroff API works differently. Worth recognising the pattern when you meet it.

</details>

---

## Part 4 - Where the token goes, and where it must not

A token is a ticket. Anyone holding it is you, for as long as it lasts. So:

**Never in a repository.** Not in your JavaScript, not in a config file you commit, not "temporarily" while you test. Public GitHub repositories are scanned automatically for exactly this, within minutes of the push. And deleting the line in a later commit does not help - it is still in the history.

**Never in a URL.** Query strings end up in browser history, server logs, and anything you paste into a chat. This is the same argument as never putting a password in a GET request, for the same reason.

**Never in a screenshot.** Including the screenshot of your console that you paste into the Teams channel when asking for help with a `401`. Blur it, or log `accessToken.slice(0, 20)` instead.

In this lesson the token lives in a plain variable, which means it disappears on reload and you log in again. That is fine for learning and hopeless for a real app - reloading the page should not log you out. Storing it somewhere that survives a reload is Module 6's territory, along with the trade-offs involved. For now, a variable.

One more thing, and this one is easy to get wrong because it looks like the opposite of a rule: **the token is not a secret from the user.** It is *their* session. They can open DevTools and read it whenever they like. The point of keeping it out of repositories and URLs is stopping it reaching *third parties*, not hiding it from the person it belongs to.

<details>

<summary>Rabbit hole: two traps in the DummyJSON docs</summary>

Worth seeing, because both are the kind of thing that will happen to you with real API documentation.

**The field got renamed.** Older tutorials - and there are many - read `result.token`. The current API returns `result.accessToken`. If you follow a two-year-old blog post you will get `undefined`, build a header saying `Bearer undefined`, and receive a `401` that has nothing to do with your token handling. **When a tutorial and the official docs disagree, the docs win**, and when both look right, log the actual response and see.

**Leave out `credentials: 'include'`.** The official examples include it. It asks the browser to send and store cookies too, and on this API it triggers a CORS error rather than working - you cannot combine credentialed requests with a wildcard `Access-Control-Allow-Origin`. Since we are passing the token in the header ourselves, we do not need cookies at all. Omit the line.

Neither of these is your mistake, and both look exactly like your mistake. This is normal. Documentation drifts, examples get copied from older versions, and part of the job is noticing when the map disagrees with the ground.

</details>

---

## Part 5 - What changes for the Noroff API

Everything you have just done transfers, with one addition. The Noroff API wants a second header:

```js
const options = {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: 'Bearer ' + accessToken,
    'X-Noroff-API-Key': apiKey,
  },
  body: JSON.stringify(data),
};
```

`Authorization` says **who the user is** - it comes from logging in, and it expires.

`X-Noroff-API-Key` says **which application is asking**. You generate it once and it does not expire. Headers starting with `X-` are the convention for headers a particular API invented for itself, so you will only ever learn about them by reading that API's documentation. Nothing generic will tell you this one exists.

Two different questions, two different headers, and a missing one of either gives you a `401`. When that happens, check both - it is easy to spend twenty minutes staring at a perfectly good token when the API key was the thing you forgot.

<details>

<summary>Exercise 3: read a failure without running it</summary>

Each of these produces a `401` against an API that wants both headers. Say why, and say what you would check first.

```js
// A
const headersA = {
  Authorization: 'Bearer ' + accessToken,
};

// B
const headersB = {
  Authorization: accessToken,
  'X-Noroff-API-Key': apiKey,
};

// C - accessToken was set from result.token
const headersC = {
  Authorization: 'Bearer ' + accessToken,
  'X-Noroff-API-Key': apiKey,
};

// D - this worked forty minutes ago and nothing has been edited since
const headersD = {
  Authorization: 'Bearer ' + accessToken,
  'X-Noroff-API-Key': apiKey,
};
```

</details>

<details>

<summary>Solution 3</summary>

**A** - no API key. The token may be perfect. Check the Network tab's Request Headers for `X-Noroff-API-Key`; if it is absent, that is your answer.

**B** - the scheme name is missing. The value is the bare token with no `Bearer ` in front, so the server cannot tell what kind of credential it is being handed. Same family of bug as the missing space.

**C** - the comment gives it away. The response field is `accessToken`, not `token`, so `accessToken` is `undefined` and the header reads `Bearer undefined`. Log the token straight after login, or set a breakpoint and read it in the Scope pane.

**D** - nothing is wrong with the code. The token expired. This is the one that wastes the most time, because the instinct when something breaks is to look for what you changed, and you changed nothing. **If it worked earlier and the code is untouched, suspect the token before you suspect yourself.**

</details>

---

## Self study task: a login form that behaves

**Estimated time:** about 30 minutes.

### Brief

1. Build a form with a username field, a password field, and a submit button.
2. On submit, POST to `/auth/login` with `expiresInMins: 1`.
3. On success, store the token in a variable, hide the form, and show the user's first name, email and a **Log out** button.
4. On failure, show the server's message next to the form and leave the form visible with the username still filled in.
5. Add a **Who am I?** button that calls `/auth/me` through your `authFetch`.
6. When a request comes back `401`, clear the token, show the form again, and say the session expired.
7. **Log out** clears the token and shows the form. No request needed - logging out is just forgetting the ticket.

### Requirements

- `event.preventDefault()` on the form.
- Disable the submit button while the login request is in flight.
- Never log the whole token. `accessToken.slice(0, 20)` is plenty.
- Test the full cycle including expiry: log in, check, wait a minute, check again.

### Things that will bite you

- A failed login returning `400` still has a readable JSON body. Parse it before deciding what to show.
- If you only check `response.ok` and never look at `401` specifically, expiry and a wrong password will produce the same message, and one of them is wrong.
- Hiding the form and showing the profile are the same state change in two places. Write one `render()` that reads the current state and sets everything, rather than poking elements from four different handlers.

<details>

<summary>Solution: HTML</summary>

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Log in</title>
    <script src="app.js" defer></script>
  </head>
  <body>
    <h1>Log in</h1>

    <form id="loginForm">
      <label for="usernameField">Username</label>
      <input id="usernameField" type="text" value="emilys" required />

      <label for="passwordField">Password</label>
      <input id="passwordField" type="password" value="emilyspass" required />

      <button id="submitButton" type="submit">Log in</button>
    </form>

    <div id="profile">
      <p id="profileText"></p>
      <button id="profileButton" type="button">Who am I?</button>
      <button id="logoutButton" type="button">Log out</button>
    </div>

    <p id="status"></p>
  </body>
</html>
```

</details>

<details>

<summary>Solution: JavaScript</summary>

```js
const baseUrl = 'https://dummyjson.com';

const loginForm = document.querySelector('#loginForm');
const usernameField = document.querySelector('#usernameField');
const passwordField = document.querySelector('#passwordField');
const submitButton = document.querySelector('#submitButton');
const profile = document.querySelector('#profile');
const profileText = document.querySelector('#profileText');
const profileButton = document.querySelector('#profileButton');
const logoutButton = document.querySelector('#logoutButton');
const statusText = document.querySelector('#status');

let accessToken = null;
let currentUser = null;

function render() {
  if (accessToken === null) {
    loginForm.style.display = 'block';
    profile.style.display = 'none';
    return;
  }

  loginForm.style.display = 'none';
  profile.style.display = 'block';

  if (currentUser) {
    profileText.textContent =
      currentUser.firstName + ' (' + currentUser.email + ')';
  }
}

async function authFetch(path) {
  const options = {
    method: 'GET',
    headers: {
      Authorization: 'Bearer ' + accessToken,
    },
  };

  const response = await fetch(baseUrl + path, options);

  if (response.status === 401) {
    accessToken = null;
    currentUser = null;
    statusText.textContent = 'Your session has expired. Please log in again.';
    render();
    throw new Error('Session expired');
  }

  if (!response.ok) {
    throw new Error('Request failed: ' + response.status);
  }

  return await response.json();
}

async function logIn(event) {
  event.preventDefault();

  submitButton.disabled = true;
  statusText.textContent = 'Logging in...';

  const options = {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      username: usernameField.value,
      password: passwordField.value,
      expiresInMins: 1,
    }),
  };

  try {
    const response = await fetch(baseUrl + '/auth/login', options);
    const result = await response.json();

    if (!response.ok) {
      statusText.textContent = 'Could not log in: ' + result.message;
      passwordField.value = '';
      submitButton.disabled = false;
      return;
    }

    accessToken = result.accessToken;
    currentUser = result;

    console.log('Token starts with:', accessToken.slice(0, 20));
    statusText.textContent = 'Logged in.';
  } catch (error) {
    console.error(error);
    statusText.textContent = 'Could not reach the server.';
  }

  submitButton.disabled = false;
  render();
}

async function showProfile() {
  statusText.textContent = 'Checking...';

  try {
    currentUser = await authFetch('/auth/me');
    statusText.textContent = 'Still logged in.';
    render();
  } catch (error) {
    console.error(error);
  }
}

function logOut() {
  accessToken = null;
  currentUser = null;
  statusText.textContent = 'Logged out.';
  render();
}

loginForm.addEventListener('submit', logIn);
profileButton.addEventListener('click', showProfile);
logoutButton.addEventListener('click', logOut);

render();
```

</details>

<details>

<summary>Notes on the solution</summary>

**One `render()`, called from everywhere.** Logging in, logging out and expiring all end with the same call. The alternative - each handler hiding and showing elements itself - means four places that must agree about what "logged in" looks like, and they will stop agreeing the moment you add a fifth.

**`render()` runs once at the bottom.** The page starts in the correct state instead of flashing the profile panel before the script hides it.

**The password field is cleared on a failed login, the username is not.** Small thing, and it is the difference between a form that feels considerate and one that makes you retype everything because of one typo.

**`authFetch` handles the `401` and re-renders before throwing.** By the time the `catch` in `showProfile` runs, the UI has already updated, so the catch only has to log. Putting the recovery in one place beats repeating it at every call site - and there will be more call sites.

**`logOut` sends no request.** With bearer tokens, logging out is forgetting the ticket. Some APIs offer a logout endpoint to invalidate the token server-side, which is stronger, but the client-side part is always just this.

</details>

---

## What to take away

**A token is a ticket.** Whoever holds it is you. That single fact explains all of it: why it must not reach a repository or a URL, why it expires, and why the server never asks who you are once you have one.

**The shape is exact.** `Bearer`, one space, the token. Every way of getting it wrong produces the same `401`, and the response never tells you which way. The answer is in the request - Network tab, Request Headers - so look there first rather than rereading your code.

**Working code stops working.** Tokens go stale, and nothing announces it. Give `401` its own branch, clear the token when you see one, and get the user back to a login screen. If something that worked forty minutes ago fails now and you have edited nothing, suspect the token first.
