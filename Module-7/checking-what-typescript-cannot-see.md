# Check it, and TypeScript believes you

**Estimated time:** about 2 hours for the core path, plus 30 to 45 minutes for the self study task.  
**Prerequisites:** Modules 1 to 6, plus lessons 7.1 to 7.4. You need to know how to make a Vite project, from lesson 7.1 or from the extra lesson [Two jobs, two tools](typescript-with-vite.md).  

This is an optional extra lesson covering the same ground as lesson 7.5, type guards and generics, by a different road.

The regular lesson starts with a type guard written in one go. This one builds it up a single check at a time, and lets TypeScript's red lines tell you which check to write next. Two sentences carry it:

**1. TypeScript cannot see your HTML, the answer from a server, or what was thrown in a `catch`, so it starts out not trusting any of them.**

**2. A type guard is a check that runs, and TypeScript believes a value is what you say once it has watched you check.**

The running example is a small page that shows a random photo of a dog breed you type in. It uses the free Dog CEO API, which needs no key and no sign-up.

---

## Part 1 - A fresh project

Make a new Vite TypeScript project called `dog-picker`, exactly as in lesson 7.1 or the previous extra lesson. In short: open an empty folder in VS Code, open a terminal, run `npm create vite@latest`, and choose **Vanilla** and then **TypeScript**. Open the new `dog-picker` folder in VS Code, and run `npm install` and then `npm run dev`. Clear out the starter page the same way as before, and remember the line `"strict": true,` in `tsconfig.json`.

Then replace everything in `index.html` with this:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Dog picker</title>
  </head>
  <body>
    <h1>Dog picker</h1>
    <form id="breed-form">
      <label for="breed">Breed</label>
      <input id="breed" name="breed" type="text" placeholder="beagle" />
      <button type="submit">Show me one</button>
    </form>
    <p id="status"></p>
    <img id="photo" alt="" width="400" hidden />
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

And everything in `src/main.ts` with this:

```ts
import './style.css';
```

There is nothing new in the HTML. The `hidden` attribute on the image keeps it out of sight until there is a photo to show.

---

## Part 2 - The page might not have it

The first thing the code needs is the elements on the page. Add this to `main.ts`:

```ts
const statusText = document.querySelector('#status');
statusText.textContent = 'Ready.';
```

A red line appears under `statusText` on the second line. Hover over it: `'statusText' is possibly 'null'`.

You know from Module 3 that `querySelector` gives back `null` when nothing on the page matches. TypeScript knows that too. And TypeScript only ever reads your `.ts` files. It has never seen `index.html`, so it has no idea whether a `#status` is in there. As far as it can tell, the answer might be `null`.

That is the first sentence of this lesson. TypeScript cannot see the page, so it does not trust what comes from it.

### The check

So check. Change the code to this:

```ts
const statusText = document.querySelector('#status');

if (!statusText) {
  throw new Error('The page has no #status element.');
}

statusText.textContent = 'Ready.';
```

The red line is gone. Save, and the page says `Ready.`

Here is what happened. TypeScript read your `if`. It saw that if `statusText` were `null`, the code would throw an error and never get to the next line. So below the `if`, it knows `statusText` cannot be `null`. Hover over `statusText` on the last line to see: before the `if` it is `Element | null`, and after it, it is just `Element`. The `|` means "or", so `Element | null` reads "an element, or nothing".

You did not have to promise TypeScript anything. You checked, with code that actually runs, and TypeScript watched you do it. **That is a type guard**, and it is the second sentence of this lesson. Everything else in this lesson is the same idea in new places.

Throwing an error is a sensible answer here. If the page is missing an element that your code needs, something is badly wrong, and stopping straight away with a clear message is better than carrying on and failing in a confusing way later.

### Saying which kind of element

Now try the image:

```ts
const photo = document.querySelector('#photo');

if (!photo) {
  throw new Error('The page has no #photo element.');
}

photo.src = 'test.jpg';
```

The `null` problem is handled, but there is a new red line under `src`: `Property 'src' does not exist on type 'Element'`.

`querySelector('#photo')` can find any kind of element at all, so TypeScript only knows it found some element. Only images have a `src`. A paragraph does not, and neither does a `div`. TypeScript cannot tell which one `#photo` is, because, once again, it cannot see your HTML.

So you tell it, in angle brackets after `querySelector`:

```ts
const photo = document.querySelector<HTMLImageElement>('#photo');
```

The red line under `src` disappears. Every kind of element has a name like this: `HTMLImageElement`, `HTMLInputElement`, `HTMLFormElement`, `HTMLParagraphElement` and so on. If you are not sure of the name, type `HTML` inside the angle brackets and let VS Code suggest the rest.

Be honest about the difference between the two fixes, though. The `if` is a **check**: it runs every time, and it would catch a missing element. The angle brackets are a **claim**: TypeScript simply believes you, and nothing checks it. If you later changed the image in `index.html` into a `div`, there would be no red line anywhere. That is fine here, because you wrote both files and you can see that `#photo` is an image. Just know which of the two you are doing.

Delete the `photo.src = 'test.jpg';` line again before you go on.

<details>
<summary>Rabbit hole: the exclamation mark in Vite's starter code</summary>

The `main.ts` that Vite gave you had lines like this one:

```text
document.querySelector<HTMLDivElement>('#app')!.innerHTML = ...
```

The `!` after the brackets means "trust me, this is not `null`". It makes the red line go away, but it checks nothing at all. It is a claim with no evidence, and if the element is ever missing, you get the same confusing crash you would get in plain JavaScript.

You will see it a lot in code written by other people. Prefer the `if`. It costs two lines, and it fails with a message you wrote yourself.

</details>

### Exercise 1 - All four elements

The page has four elements that the code needs: the form, the input, the status paragraph and the image. Get all four, each with the right element name in angle brackets. Then replace the separate `if` checks with a single `if` that throws one error if any of the four is missing. Keep the `statusText.textContent = 'Ready.';` line at the end, to see that it all works.

<details>
<summary>Solution</summary>

```ts
import './style.css';

const form = document.querySelector<HTMLFormElement>('#breed-form');
const input = document.querySelector<HTMLInputElement>('#breed');
const statusText = document.querySelector<HTMLParagraphElement>('#status');
const photo = document.querySelector<HTMLImageElement>('#photo');

if (!form || !input || !statusText || !photo) {
  throw new Error('The page is missing an element that main.ts needs.');
}

statusText.textContent = 'Ready.';
```

`||` reads as "or": if the form is missing, or the input is missing, and so on. Below the `if`, TypeScript knows that none of the four can be `null`.

The variable is called `statusText` rather than `status`, because the browser already has an old built-in variable called `status`, and using the same name for your own is asking for confusion.

</details>

---

## Part 3 - Listening to the form

Before fetching any dogs, make the form do something simple. Remove the `'Ready.'` line, and add this at the bottom of `main.ts`:

```ts
form.addEventListener('submit', (event) => {
  event.preventDefault();
  const breed = input.value.trim().toLowerCase();
  statusText.textContent = 'You asked for: ' + breed;
});
```

Everything here is from Modules 2 and 4: `preventDefault` stops the page from reloading, `trim` removes spaces at the ends, and `toLowerCase` makes `Beagle` into `beagle`, which is what the API will want.

Notice that there are no red lines inside the arrow function, even though it uses `input` and `statusText`. TypeScript knows this function can only ever run after your check further up, so it knows none of them is `null` in here either.

Type `Beagle` into the box and press Enter. The page says `You asked for: beagle`.

---

## Part 4 - The answer you cannot see

Now the API. Open this address in a browser tab:

```text
https://dog.ceo/api/breed/beagle/images/random
```

The answer is a small object. The address of a beagle photo is in `message`:

```json
{
  "message": "https://images.dog.ceo/breeds/beagle/n02088364_11136.jpg",
  "status": "success"
}
```

Now try a breed that does not exist:

```text
https://dog.ceo/api/breed/notadog/images/random
```

```json
{
  "status": "error",
  "message": "Breed not found (main breed does not exist)",
  "code": 404
}
```

Look closely, because this API has a quirk. Both answers have a `message` and a `status`, but `message` means two completely different things. When `status` is `"success"`, `message` is a photo address. When `status` is `"error"`, `message` is a sentence about what went wrong. Your code has to look at `status` before it can know what `message` is.

Describe what both answers have in common, with an interface. Put it near the top of `main.ts`, just under the import:

```ts
interface DogApiAnswer {
  message: string;
  status: string;
}
```

### unknown: the honest type

When you fetch, `response.json()` gives you a value of type `any`. As the first TypeScript extra lesson showed, `any` switches the checking off completely: you can do anything with it, typos included, and TypeScript will not say a word.

There is a more honest type for a value you have not looked at yet, and it is called `unknown`. It is the opposite of `any`: TypeScript will not let you do anything at all with an `unknown` value until you have checked what it is. That sounds annoying, but it is exactly what you want for data from another computer.

### Exercise 2 - Your first guard, one check at a time

You are going to write a function that checks whether a value looks like a `DogApiAnswer`. Instead of writing the whole thing in one go, you will let TypeScript tell you what to check next.

Start with this, under the interface:

```ts
/**
 * Checks whether a value has the shape of an answer from the Dog API.
 * @param value Anything at all, usually data from a server.
 * @returns True if the value has a text message and a text status.
 */
function isDogApiAnswer(value: unknown): value is DogApiAnswer {
  return typeof value.message === 'string';
}
```

Ignore the odd return type `value is DogApiAnswer` for a moment, it gets explained right after this exercise. VS Code may also say that `isDogApiAnswer` is declared but never read. That only means nothing uses it yet, and it goes away in Exercise 3.

There is a red line under `value`. Hover over it, and do what each message asks, one at a time:

**Step 1.** The message is `'value' is of type 'unknown'`. TypeScript will not let you look inside something when it does not even know it is an object. So check that first. Put `typeof value === 'object' &&` in front of what is there. Then hover again.

**Step 2.** Now there are two messages. Take the first one first: `'value' is possibly 'null'`. That is because of one of JavaScript's oldest mistakes: `typeof null` is `'object'`. So the first check lets `null` through. Add `value !== null &&` straight after the first check.

**Step 3.** One message left: `Property 'message' does not exist on type 'object'`. TypeScript now knows `value` is an object, but not that this object has a `message`. The `in` operator from lesson 7.5 asks exactly that: `'message' in value` is `true` if the object has a property called `message`. Add `'message' in value &&` before the last check.

The red line is gone. Now add the same two checks for `status` on the end, so the function checks that `status` exists and is text, too.

<details>
<summary>Solution</summary>

```ts
/**
 * Checks whether a value has the shape of an answer from the Dog API.
 * @param value Anything at all, usually data from a server.
 * @returns True if the value has a text message and a text status.
 */
function isDogApiAnswer(value: unknown): value is DogApiAnswer {
  return (
    typeof value === 'object' &&
    value !== null &&
    'message' in value &&
    typeof value.message === 'string' &&
    'status' in value &&
    typeof value.status === 'string'
  );
}
```

The brackets around the whole thing only let you spread it over several lines, so each check gets a line of its own.

Read it from the top. Each check is only reached if all the ones before it were `true`, because `&&` stops at the first `false`. So by the time it gets to `value.message`, it already knows `value` is an object, not `null`, and has a `message`.

You did not have to know all six checks in advance. TypeScript told you what was missing, one message at a time. That is a very good way of working with it in general.

</details>

### What "value is DogApiAnswer" means

The function returns `true` or `false`, like any check. The return type `value is DogApiAnswer` adds a promise on top: "if this returns `true`, you may treat `value` as a `DogApiAnswer`". That is what lets you use it in an `if`, in exactly the same way as `if (!statusText)` in Part 2.

One thing worth knowing: TypeScript trusts that your guard function is correct. If you wrote `return true;` inside it, TypeScript would happily believe everything is a `DogApiAnswer`. So the guard moves the promise from "I am sure the server sends this" to "I am sure this function checks properly". That is a much better promise to have to make, because the function is yours, and you can read it and test it.

### Exercise 3 - What gets through?

For each line, decide whether `isDogApiAnswer` returns `true` or `false`, and which check stops it if it is `false`. Then add the lines to the bottom of `main.ts` and check your answers in the browser console.

```ts
console.log(isDogApiAnswer({ message: 'hello', status: 'success' }));
console.log(isDogApiAnswer({ message: 'hello' }));
console.log(isDogApiAnswer(null));
console.log(isDogApiAnswer('success'));
console.log(isDogApiAnswer({ message: 404, status: 'error' }));
console.log(isDogApiAnswer({ message: 'hi', status: 'error', code: 404 }));
```

<details>
<summary>Solution</summary>

`true`, `false`, `false`, `false`, `false`, `true`.

The first has both properties, both as text. The second has no `status`, so `'status' in value` stops it. `null` is stopped by `value !== null`, and only because that line is there. The text `'success'` is not an object, so the very first check stops it. In the fifth, `message` is a number, so `typeof value.message === 'string'` stops it.

The last one is the interesting one. It has an extra property, `code`, and it still gets through. The guard only checks that the properties you need are there and are the right type, and extra ones do no harm. That is what lets the real error answer from the API, which has a `code`, through the guard, so that your code can read its `message`.

Delete the six lines when you are done.

</details>

---

## Part 5 - Fetching, and the catch

Now a function that fetches a photo address and uses the guard. Add it under `isDogApiAnswer`:

```ts
/**
 * Asks the Dog API for one random photo of a breed.
 * @param breed A breed name in lower case, like "beagle".
 * @returns The address of the photo.
 */
async function fetchDogPhoto(breed: string): Promise<string> {
  const response = await fetch('https://dog.ceo/api/breed/' + breed + '/images/random');
  const data: unknown = await response.json();

  if (!isDogApiAnswer(data)) {
    throw new Error('That answer did not look like it came from the Dog API.');
  }
  if (data.status !== 'success') {
    throw new Error(data.message);
  }
  return data.message;
}
```

Going through it a line at a time:

The return type is `Promise<string>`. An `async` function always gives back a promise, as you saw in Module 4, and the type in the angle brackets says what the promise gives you once it is done waiting: a string.

`const data: unknown` is where you choose honesty. Writing `unknown` here, instead of letting it be `any`, is what makes TypeScript insist on a check.

The first `if` uses your guard. Hover over `data` below it, and it is a `DogApiAnswer`. If the answer was not even the right shape, you throw an error with a message you wrote yourself.

The second `if` handles the quirk. If `status` is anything other than `"success"`, then `message` is the API explaining what went wrong, so you throw that as an error.

If you get past both, `message` is a photo address, and you hand it back.

Notice that this function does not check `response.ok`, which you did in Module 4. That is only because this particular API sends a useful sentence when something goes wrong, so it makes sense to read the answer either way and let `status` decide. For an API that sends nothing useful when it fails, checking `response.ok` first is still the right habit.

### Using it in the form

Now replace the `submit` listener from Part 3 with this one:

```ts
form.addEventListener('submit', async (event) => {
  event.preventDefault();
  const breed = input.value.trim().toLowerCase();

  if (breed === '') {
    statusText.textContent = 'Type a breed first, for example beagle.';
    return;
  }

  statusText.textContent = 'Fetching ' + breed + '...';
  photo.hidden = true;

  try {
    photo.src = await fetchDogPhoto(breed);
    photo.alt = 'A random photo of a ' + breed;
    photo.hidden = false;
    statusText.textContent = '';
  } catch (error) {
    statusText.textContent = 'No luck: ' + error.message;
  }
});
```

The arrow function now has `async` in front of it, so that it can use `await`. Apart from that, it is a `try...catch` like the ones in Module 5.

There is one red line, under `error` in the `catch`. Hover over it: `'error' is of type 'unknown'`.

### Exercise 4 - Why unknown?

You might expect the thing in a `catch` always to be an `Error`. Think back to lesson 5.5, where you used `throw`. What is it possible to throw in JavaScript? Why can TypeScript not promise you that `error` has a `message`?

<details>
<summary>Solution</summary>

JavaScript lets you throw anything at all. `throw new Error('oops')` is the usual way, but `throw 'oops'` and `throw 42` are allowed too, and code you did not write might do something odd. A string has no `message`. If TypeScript let you write `error.message` without a check, the page could end up saying `No luck: undefined`.

So `catch` gives you `unknown`, and you check it, the same way you checked the server's answer. It is the first sentence of the lesson again: TypeScript cannot see what was thrown.

</details>

This time you do not need to write a guard yourself. In lesson 5.5, you used `instanceof` to ask whether an error was a particular kind of error. TypeScript understands `instanceof` as a type guard too. Change the `catch` to this:

```ts
  } catch (error) {
    if (error instanceof Error) {
      statusText.textContent = 'No luck: ' + error.message;
    } else {
      statusText.textContent = 'Something went wrong.';
    }
  }
```

Inside the `if`, `error` is an `Error`, so it has a `message`. Both of your own `throw new Error(...)` lines from `fetchDogPhoto` end up here, and so does the error that `fetch` throws if the network is down.

Save, and try it: `beagle`, `husky`, `pug`, `samoyed`. Then try `notadog`, and you get the API's own explanation: `No luck: Breed not found (main breed does not exist)`.

---

## Part 6 - Generics, which you have been using all along

Look back at two things you have written in this lesson:

```text
document.querySelector<HTMLImageElement>('#photo')
Promise<string>
```

Both have a type in angle brackets. That is what a **generic** is: something that can work with many types, where the angle brackets say which type this time. `querySelector` can find any kind of element, and you said which kind. A promise can give back any kind of value, and you said which kind.

You can write a generic function of your own. This one picks a random item from any array. It uses `Math.random` and `Math.floor` from Module 2. Add it under `fetchDogPhoto`:

```ts
/**
 * Picks one item at random from a list.
 * @param items The list to pick from. It must not be empty.
 * @returns One of the items.
 */
function pickOne<T>(items: T[]): T {
  const index = Math.floor(Math.random() * items.length);
  return items[index];
}
```

Read `<T>` as "this function works with some type, and we will call it T for now". The parameter is "an array of T", and the return value is "a T". So whatever kind of array goes in, the same kind of item comes out.

Now use it, at the bottom of `main.ts`, to give the input a different suggestion every time the page loads:

```ts
const easyBreeds = ['beagle', 'boxer', 'corgi', 'husky', 'pug', 'shiba', 'samoyed'];
input.placeholder = 'Try ' + pickOne(easyBreeds);
```

Reload the page a few times and watch the grey suggestion text in the box change.

### Exercise 5 - What comes out?

Add these two lines, and before you hover over them, decide what type TypeScript will say each one is:

```ts
const someBreed = pickOne(easyBreeds);
const someNumber = pickOne([3, 7, 12]);
```

<details>
<summary>Solution</summary>

`someBreed` is a `string`, and `someNumber` is a `number`.

You never wrote `<string>` or `<number>` anywhere. TypeScript looked at the array you passed in and worked out what `T` had to be, the same way it works out the type of `let count = 3`.

Without generics you would have two bad choices: write one `pickOne` for strings and another for numbers, or use `any` and lose the checking. With `<T>`, one function works for every kind of array, and TypeScript still keeps track of what you get back.

Delete the two lines when you are done, or VS Code will point out that they are never used.

</details>

<details>
<summary>Rabbit hole: the empty list</summary>

The comment on `pickOne` says the list must not be empty, and that matters. With an empty array, `items.length` is `0`, so the index is `0`, and `items[0]` is `undefined`. But the return type says `T`, so TypeScript believes a string came back when nothing did.

That happens because, by default, TypeScript does not check whether a position in an array actually has something in it. Otherwise you would need a check every single time you wrote `items[i]`. There is a stricter setting, called `noUncheckedIndexedAccess`, that makes you check. For now, the comment is the honest fix.

</details>

---

## The whole of main.ts

If anything has gone astray, this is what `main.ts` should look like now:

```ts
import './style.css';

interface DogApiAnswer {
  message: string;
  status: string;
}

/**
 * Checks whether a value has the shape of an answer from the Dog API.
 * @param value Anything at all, usually data from a server.
 * @returns True if the value has a text message and a text status.
 */
function isDogApiAnswer(value: unknown): value is DogApiAnswer {
  return (
    typeof value === 'object' &&
    value !== null &&
    'message' in value &&
    typeof value.message === 'string' &&
    'status' in value &&
    typeof value.status === 'string'
  );
}

/**
 * Asks the Dog API for one random photo of a breed.
 * @param breed A breed name in lower case, like "beagle".
 * @returns The address of the photo.
 */
async function fetchDogPhoto(breed: string): Promise<string> {
  const response = await fetch('https://dog.ceo/api/breed/' + breed + '/images/random');
  const data: unknown = await response.json();

  if (!isDogApiAnswer(data)) {
    throw new Error('That answer did not look like it came from the Dog API.');
  }
  if (data.status !== 'success') {
    throw new Error(data.message);
  }
  return data.message;
}

/**
 * Picks one item at random from a list.
 * @param items The list to pick from. It must not be empty.
 * @returns One of the items.
 */
function pickOne<T>(items: T[]): T {
  const index = Math.floor(Math.random() * items.length);
  return items[index];
}

const form = document.querySelector<HTMLFormElement>('#breed-form');
const input = document.querySelector<HTMLInputElement>('#breed');
const statusText = document.querySelector<HTMLParagraphElement>('#status');
const photo = document.querySelector<HTMLImageElement>('#photo');

if (!form || !input || !statusText || !photo) {
  throw new Error('The page is missing an element that main.ts needs.');
}

form.addEventListener('submit', async (event) => {
  event.preventDefault();
  const breed = input.value.trim().toLowerCase();

  if (breed === '') {
    statusText.textContent = 'Type a breed first, for example beagle.';
    return;
  }

  statusText.textContent = 'Fetching ' + breed + '...';
  photo.hidden = true;

  try {
    photo.src = await fetchDogPhoto(breed);
    photo.alt = 'A random photo of a ' + breed;
    photo.hidden = false;
    statusText.textContent = '';
  } catch (error) {
    if (error instanceof Error) {
      statusText.textContent = 'No luck: ' + error.message;
    } else {
      statusText.textContent = 'Something went wrong.';
    }
  }
});

const easyBreeds = ['beagle', 'boxer', 'corgi', 'husky', 'pug', 'shiba', 'samoyed'];
input.placeholder = 'Try ' + pickOne(easyBreeds);
```

Finish the way the previous lesson taught you: run `npx tsc` in the terminal. If it prints nothing, everything checks out.

---

## Self study task: two more buttons' worth

### Level 1: surprise me

Add a button to the form in `index.html`:

```html
<button type="button" id="surprise">Surprise me</button>
```

`type="button"` stops it from submitting the form. When it is clicked, it should put a random breed from `easyBreeds` into the input, and show a photo of that breed.

You do not want to copy the whole `try...catch` from the form listener into a second listener. Instead, move the code that fetches and shows a breed into a function of its own, that both listeners can call.

<details>
<summary>Hints</summary>

**The new element.** Get it with `querySelector<HTMLButtonElement>`, and add it to the big `if` check.

**The shared function.** Call it `showBreed`, and give it one parameter, `breed: string`. Move everything from the form listener into it, except `event.preventDefault()`. Then the form listener becomes three lines: prevent the default, and call `showBreed(input.value)`.

**Where, and how, to write it.** Write it as an arrow function stored in a `const`, like `const showBreed = async (breed: string): Promise<void> => { ... };`, and put it below the big `if` check. If you write it as an ordinary `function` instead, you get red lines saying `statusText` and `photo` are possibly `null`. The rabbit hole after the solution explains why.

</details>

<details>
<summary>Solution</summary>

Next to the other four elements, and in the check:

```ts
const surpriseButton = document.querySelector<HTMLButtonElement>('#surprise');

if (!form || !input || !statusText || !photo || !surpriseButton) {
  throw new Error('The page is missing an element that main.ts needs.');
}
```

Then, below the check, instead of the old form listener:

```ts
/**
 * Fetches a photo of a breed and shows it, or shows what went wrong.
 * @param breed The breed to show, as the user typed it.
 */
const showBreed = async (breed: string): Promise<void> => {
  const cleanBreed = breed.trim().toLowerCase();

  if (cleanBreed === '') {
    statusText.textContent = 'Type a breed first, for example beagle.';
    return;
  }

  statusText.textContent = 'Fetching ' + cleanBreed + '...';
  photo.hidden = true;

  try {
    photo.src = await fetchDogPhoto(cleanBreed);
    photo.alt = 'A random photo of a ' + cleanBreed;
    photo.hidden = false;
    statusText.textContent = '';
  } catch (error) {
    if (error instanceof Error) {
      statusText.textContent = 'No luck: ' + error.message;
    } else {
      statusText.textContent = 'Something went wrong.';
    }
  }
};

form.addEventListener('submit', (event) => {
  event.preventDefault();
  showBreed(input.value);
});

surpriseButton.addEventListener('click', () => {
  const breed = pickOne(easyBreeds);
  input.value = breed;
  showBreed(breed);
});
```

It works wherever `easyBreeds` is defined in the file, because the click listener only runs later, when someone clicks. It reads more naturally if you move the `const easyBreeds` line up, next to the elements. Putting the breed into the input as well tells the user what they are looking at.

</details>

<details>
<summary>Rabbit hole: why an ordinary function gets red lines here</summary>

Back in Module 2, the lesson on arrow functions mentioned that function declarations are "hoisted": JavaScript lets you call them before the line where they are written. That is the whole reason.

If `showBreed` were an ordinary `function`, it could in principle be called from a line above the `if` check, before anything has been checked. TypeScript cannot rule that out, so inside the function it goes back to "possibly null". An arrow function stored in a `const` cannot be used before its line, so TypeScript knows the check has always happened first.

`fetchDogPhoto`, `isDogApiAnswer` and `pickOne` do not have this problem, because they never touch the page. They get everything they need through their parameters.

</details>

### Level 2: remember the last breed

Make the page remember the last breed that was shown successfully, using `localStorage` from Module 5. When the page loads, put it in the input.

Before you write it, hover over `localStorage.getItem('lastBreed')`. What type does it give back, and what does that mean you have to do before you can put it into `input.value`?

<details>
<summary>Solution</summary>

`getItem` gives back `string | null`, because the very first time, nothing has been saved yet. So it needs one more type guard before you can use it.

Save the breed inside the `try`, after the photo is shown, so only breeds that actually worked get remembered:

```ts
    photo.hidden = false;
    statusText.textContent = '';
    localStorage.setItem('lastBreed', cleanBreed);
```

And at the very bottom of `main.ts`:

```ts
const savedBreed = localStorage.getItem('lastBreed');
if (savedBreed) {
  input.value = savedBreed;
}
```

Inside the `if`, `savedBreed` is a `string`. Try writing `input.value = localStorage.getItem('lastBreed');` without the check, and you get a red line, because `input.value` has to be a string and `null` is not one.

</details>

<details>
<summary>Extra, if you have programmed before: the last five breeds</summary>

Keep a list of the last five breeds that worked, newest first and without repeats, save it in `localStorage`, and show it as a row of buttons under the form. Clicking one shows that breed again.

Saving a list means `JSON.stringify`, and reading it back means `JSON.parse`. `JSON.parse` gives you `any`, just like `response.json()` did, so put the result in an `unknown` and write a guard `isListOfStrings(value: unknown): value is string[]` before you trust it. You need one built-in you have not met in this course: `Array.isArray(value)` is `true` if `value` is an array, and TypeScript understands it as a guard. Then check each item with a `for...of` loop.

Why a guard for data you saved yourself? Because `localStorage` belongs to the browser, not to your code. The user, a browser extension, or an older version of your own page could have put something else under the same name.

```ts
/**
 * Checks whether a value is an array where every item is a string.
 * @param value Anything at all, usually parsed JSON.
 * @returns True if the value is a list of strings.
 */
function isListOfStrings(value: unknown): value is string[] {
  if (!Array.isArray(value)) {
    return false;
  }
  for (const item of value) {
    if (typeof item !== 'string') {
      return false;
    }
  }
  return true;
}

/**
 * Reads the list of recent breeds from localStorage.
 * @returns The saved list, or an empty list if nothing usable was saved.
 */
function loadRecent(): string[] {
  const saved = localStorage.getItem('recentBreeds');
  if (!saved) {
    return [];
  }
  try {
    const data: unknown = JSON.parse(saved);
    if (isListOfStrings(data)) {
      return data;
    }
  } catch (error) {
    console.warn('recentBreeds was not valid JSON, starting again.', error);
  }
  return [];
}

/**
 * Adds a breed to the front of the list, keeping at most five.
 * @param list The current list.
 * @param breed The breed that just worked.
 * @returns A new list.
 */
function addRecent(list: string[], breed: string): string[] {
  const others = list.filter((item) => item !== breed);
  return [breed, ...others].slice(0, 5);
}
```

Add `<div id="recent"></div>` under the form, get it as `recentBox` with `querySelector<HTMLDivElement>`, and add it to the check. Then, below the check:

```ts
let recentBreeds = loadRecent();

/**
 * Draws one button per recent breed.
 */
const renderRecent = (): void => {
  recentBox.innerHTML = '';
  for (const breed of recentBreeds) {
    const button = document.createElement('button');
    button.type = 'button';
    button.textContent = breed;
    button.addEventListener('click', () => {
      input.value = breed;
      showBreed(breed);
    });
    recentBox.appendChild(button);
  }
};

renderRecent();
```

And in `showBreed`, after `localStorage.setItem('lastBreed', cleanBreed);`:

```ts
    recentBreeds = addRecent(recentBreeds, cleanBreed);
    localStorage.setItem('recentBreeds', JSON.stringify(recentBreeds));
    renderRecent();
```

To test the guard, run `localStorage.setItem('recentBreeds', '42')` in the browser console and reload. The page should start with an empty list instead of crashing.

</details>

---

## Where this leaves you

TypeScript cannot see your HTML, the answer from a server, or what was thrown in a `catch`. So `querySelector` gives you "an element, or `null`", `catch` gives you `unknown`, and data from a server is only `any` until you choose to call it `unknown`.

You have two ways forward from there. You can make a **claim**, like `querySelector<HTMLImageElement>`, and TypeScript simply believes you. Or you can write a **check** that runs, like `if (!photo)`, `error instanceof Error`, or a guard function of your own with `value is ...` as its return type, and TypeScript believes you because it watched you check. Use the check wherever the value comes from outside your own code. And when you write a guard, let the red lines tell you what to check next.

Generics are what the angle brackets are: a type that gets filled in later, by whoever uses the function, so one function can work with many types without losing track of which one it is holding.
