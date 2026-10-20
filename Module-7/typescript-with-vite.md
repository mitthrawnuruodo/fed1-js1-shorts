# Two jobs, two tools

**Estimated time:** about 1.5 hours for the core path, plus 30 minutes for the self study task.  
**Prerequisites:** Modules 1 to 6, plus lesson 7.1. The first TypeScript extra lesson, [Claims, checked before running](typescript-in-the-playground.md), helps but is not required.  

This is an optional extra lesson covering the project setup from lesson 7.1, which the regular lesson does in a single line: `npm create vite@latest`.

This lesson slows that line down. You will open a terminal, make a project, look at what you got, and find out what each part is for. There is very little new TypeScript in it. The point is to get comfortable with the tools, so that the next lesson can be about TypeScript again. Two sentences carry it:

**1. Vite's job is to get your code into the browser. Checking your types is TypeScript's job, and Vite will not do it for you.**

**2. So while you work, the red lines in VS Code are your checker, and before you hand anything in, you run the check yourself.**

If you have done the first TypeScript extra lesson, you will recognise the running example: the departure board for a small, made-up ferry company.

---

## Part 1 - The terminal

Until now, you have done everything by clicking: opening files, starting Live Server, opening the browser. Some things are done by typing commands instead, and the place where you type them is called the **terminal**.

A terminal is a window where you type a command, press Enter, and the computer answers in text. That is all it is. You cannot break your computer by opening one, and every command in this lesson is safe to run.

VS Code has a terminal built in. In VS Code, choose **Terminal** in the menu bar, then **New Terminal**. A panel opens at the bottom of the window, with a blinking cursor.

Type this and press Enter:

```text
node -v
```

You should see a version number, something like `v22.12.0`. Then try this:

```text
npm -v
```

Another version number. **Node** is a program that runs JavaScript outside the browser. **npm** comes with it, and it downloads and runs tools that other people have written.

If either command says something like "command not found" or "is not recognized", Node is not installed yet. The simplest way to install it is to go to `nodejs.org`, download the version marked **LTS**, and run the installer. LTS stands for "long-term support", the steady version that most projects use. Lesson 7.1 also mentions a tool called NVM, which can switch between several versions of Node, but you do not need that yet. (If you are on a Mac and already use Homebrew, `brew install node` works too.) Then close VS Code completely, open it again, open a new terminal, and try `node -v` once more. A terminal only notices newly installed programs after a restart.

### On Windows: the error about running scripts

On Windows, the terminal in VS Code is a program called PowerShell. Every command in this lesson works the same there as on a Mac, so you do not need to install anything extra. But on many Windows computers, the first `npm` command stops with a red error that says something like this:

```text
npm.ps1 cannot be loaded because running scripts is disabled on this system.
```

Nothing is broken, and you have not done anything wrong. It is a safety setting in Windows that blocks small script files, and npm happens to use one. You only need to change it once. Copy this line into the terminal and press Enter:

```text
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

If it asks you to confirm, type `Y` and press Enter. Then close the terminal (the bin icon at the top of the terminal panel), open a new one, and try `npm -v` again. You do not need to be an administrator to do this. The change only applies to your own user account, and it still blocks scripts downloaded from the internet that have not been signed.

If you would rather not change the setting, there is another way: use a different terminal, where the problem does not happen. Click the small arrow next to the **+** at the top of the terminal panel and choose **Select Default Profile**. Then pick **Command Prompt**, or **Git Bash** if you have it installed (see the rabbit hole just below). Open a new terminal, and from now on VS Code uses that one every time.

<details>
<summary>Rabbit hole for Windows: Git Bash, a terminal that works like the one on a Mac</summary>

You do not need this for any of the JavaScript 1 lessons. Everything here works in PowerShell and Command Prompt. But it is worth knowing about, and many Windows developers use it every day.

**What it is.** Mac and Linux use one kind of terminal, and Windows has its own kinds. Most tutorials, videos and answers online are written for the Mac and Linux kind. Commands like `ls` (list the files in a folder) or `mkdir new-folder && cd new-folder` (make a folder, then move into it) work there, but not always in PowerShell or Command Prompt.

**Git Bash** is a terminal for Windows that works like the Mac and Linux one. It comes free with **Git for Windows**, which you may already have if you have used Git or GitHub on this computer. It also has no problem with the "running scripts is disabled" error, because it is not PowerShell.

### Step 1: Do you have it already?

In VS Code, click the small arrow next to the **+** at the top of the terminal panel. If **Git Bash** is in the list, you already have it, and you can skip straight to Step 3.

### Step 2: Install Git for Windows

Go to `git-scm.com`, find the download for Windows, and run the installer when it has downloaded.

The installer asks a lot of questions, and you can keep the suggested choice for almost all of them by clicking **Next**. There is one screen worth changing. When it asks which **default editor** Git should use, the suggestion is a program called Vim, which is famously hard for beginners to get out of. Choose **Visual Studio Code** from the list instead.

The screens may look a little different from version to version. If you are unsure about one, the suggested choice is fine.

When it has finished, close VS Code completely and open it again, so that it notices the new program.

### Step 3: Make Git Bash the terminal VS Code opens

Click the small arrow next to the **+** again, choose **Select Default Profile**, and pick **Git Bash**. Then open a new terminal with the **+**.

It looks a little different from PowerShell. The line before the cursor shows your user name, the name of your computer, and the folder you are in, and it ends with a `$`. Check that Node is reachable from here too:

```text
node -v
npm -v
```

Two version numbers, and you are done. From now on, every new terminal in VS Code is Git Bash.

### Two things to know

**Use it inside VS Code.** Installing Git for Windows also adds a separate Git Bash app to the Start menu. Some programs that ask questions, like `npm create vite@latest`, can have trouble in that separate app: the arrow keys might not work, or nothing seems to happen. If that happens, run the command in VS Code's terminal instead, where it works normally.

**Folders are written differently.** Git Bash writes the folder `C:\Users\Kari\Documents` as `/c/Users/Kari/Documents`, with forward slashes. It is the same folder, just written the Mac and Linux way. Since you open folders through VS Code anyway, you will rarely need to type one.

</details>


Two more things worth knowing before you go on:

**The terminal always works inside one folder.** In VS Code, that is the folder you have open. So the simplest way to control where a command runs is to open the right folder in VS Code first, and then open the terminal.

**To stop a command that keeps running,** click in the terminal and press **Ctrl+C**. That is Ctrl, also on a Mac, not Cmd. On Windows, it may ask `Terminate batch job (Y/N)?`. Type `Y` and press Enter.

---

## Part 2 - Making the project

If you do not already have one, make a folder called `Repos` or `Projects` to keep all your coding projects in. Put it somewhere easy to reach but outside any folder that iCloud (on a Mac) or OneDrive (on Windows) syncs, since those often include your Desktop and Documents. Your user folder is a good place: on a Mac, that is the folder with the little house icon in Finder, and on Windows, it is `C:\Users\` followed by your user name. The reason is that every project like this one will soon contain thousands of small files, and a sync service trying to upload them all can make `npm install` slow, or make it fail in strange ways.

Inside that folder, make a new, empty folder for this lesson, for example `js1-typescript`. In VS Code, choose **File**, then **Open Folder**, and open it. Then open a new terminal, as in Part 1.

Now type the command from lesson 7.1 and press Enter:

```text
npm create vite@latest
```

The first time, npm may ask whether it is OK to install a package called `create-vite`. That is the little tool that builds the project for you. Type `y` and press Enter.

Then it asks you some questions. You answer with the arrow keys and Enter, or by typing:

**Project name:** type `ferry-board` and press Enter. This becomes the name of a new folder.

**Select a framework:** use the arrow keys to get to **Vanilla**, and press Enter. Vanilla means plain JavaScript or TypeScript, without React or any of the others in the list.

**Select a variant:** choose **TypeScript**.

**Install with npm and start now?** choose **No**. You will do those two steps yourself, so you can see what each one does.

It answers that it is done, and suggests three commands. Because of the folder rule from Part 1, you can skip the first one, which only moves the terminal into the new folder. Instead, choose **File**, then **Open Folder**, and open the `ferry-board` folder that has just appeared inside `js1-typescript`.

VS Code reloads. Open a new terminal, which now works inside `ferry-board`, and run:

```text
npm install
```

This downloads Vite and TypeScript into your project. It can take a minute, and a few yellow warnings are normal. When the blinking cursor comes back, run:

```text
npm run dev
```

The terminal shows an address, usually `http://localhost:5173/`. Hold Ctrl (Cmd on a Mac) and click it, or copy it into your browser. You will see Vite's welcome page.

This command keeps running for as long as you want to look at your page, because it is serving it, the same job Live Server did for you before. Leave it running. When you need to type another command, click the **+** at the top of the terminal panel to open a second terminal beside it.

<details>
<summary>Rabbit hole: what if the questions do not look exactly like this?</summary>

`create-vite` is updated now and then, and the exact wording or the list of frameworks can change a little. The idea stays the same: a name for the folder, a framework (Vanilla), a variant (TypeScript), and whether to install now. If something unfamiliar comes up, the choice it suggests is almost always fine. And if it goes wrong, you can delete the folder and start again, since nothing outside it has been changed.

</details>

---

## Part 3 - What you got

Look at the file list on the left. It looks like a lot, but you only need to know about a few of these.

**`index.html`** is your page, the same kind of file you have written since the very start. Near the bottom is the one line worth a close look:

```html
<script type="module" src="/src/main.ts"></script>
```

`type="module"` is the same thing you met with ES modules in lesson 3.2. The odd part is the `.ts` at the end, because a browser cannot run TypeScript. What happens is that when the browser asks for `main.ts`, Vite steps in, removes the types, and hands the browser plain JavaScript instead. That is Vite's main job: it sits between your files and the browser, and translates them as they are asked for.

**The `src` folder** is where your own code goes. `src/main.ts` is where it starts.

**`package.json`** describes the project. It lists the tools the project needs, and it has a part called `scripts`:

```json
"scripts": {
  "dev": "vite",
  "build": "tsc && vite build",
  "preview": "vite preview"
}
```

When you typed `npm run dev`, npm looked up `"dev"` here and ran what it says, which is `vite`. Notice that `"build"` runs two things, `tsc` and then `vite build`. Part 5 is about why.

**`tsconfig.json`** holds the settings for TypeScript's checker. You will change one line in it in a moment, and otherwise leave it alone.

**`node_modules`** is where `npm install` put Vite, TypeScript and everything they need. It is big, it is not yours, and you never edit anything in it. It is also listed in a file called `.gitignore`, which means Git leaves it out, so it never ends up on GitHub. Anyone who downloads your project runs `npm install` and gets their own copy.

**`public`** is for files that should be served exactly as they are, such as a favicon. You can ignore it for now.

---

## Part 4 - Clearing out the starter page

Vite's welcome page is not much use to you, so clear it out. Do these steps in this order, because the order matters.

**First,** replace everything in `index.html` with this:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Ferry board</title>
  </head>
  <body>
    <h1>Departures</h1>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

**Second,** replace everything in `src/main.ts` with this:

```ts
import './style.css';

console.log('main.ts is running');
```

The first line brings in the stylesheet. That only works because Vite handles it for you. You could not do that in a plain HTML project without Vite.

**Third,** replace everything in `src/style.css` with this, or with a few lines of your own:

```css
body {
  font-family: system-ui, sans-serif;
  max-width: 40rem;
  margin: 2rem auto;
  padding: 0 1rem;
}
```

**Last,** delete the file `src/counter.ts` and the folder `src/assets`. If you had deleted them first, the old `main.ts` would still have been trying to use them, and Vite would have covered the page with an error message.

Save everything and look at the browser. You did not reload it, but it already shows your new page. Vite noticed that you saved, and updated the page for you. Open the browser console (as you did in Module 1), and you should see `main.ts is running`.

### One line in tsconfig.json

Open `tsconfig.json`. Just after the line `"compilerOptions": {`, add this line:

```json
    "strict": true,
```

This switches on all of TypeScript's checks. New versions of TypeScript do that anyway, but VS Code comes with its own copy of TypeScript, which might be an older version. This line makes sure VS Code and your project check exactly the same things, so the red lines you see in VS Code always match what the checker says.

### The board

Now put something on the page. Replace everything in `main.ts` with this:

```ts
import './style.css';

interface Departure {
  destination: string;
  time: string;
  minutesLate: number;
}

const departures: Departure[] = [
  { destination: 'Holmen', time: '08:15', minutesLate: 0 },
  { destination: 'Sandvik', time: '09:40', minutesLate: 5 },
  { destination: 'Rong', time: '11:05', minutesLate: 0 },
];

/**
 * Builds one line of text for the departure board.
 * @param departure The departure to describe.
 * @returns A line like "09:40 Sandvik (5 min late)".
 */
function describe(departure: Departure): string {
  if (departure.minutesLate > 0) {
    return departure.time + ' ' + departure.destination + ' (' + departure.minutesLate + ' min late)';
  }
  return departure.time + ' ' + departure.destination;
}

const list = document.createElement('ul');

for (const departure of departures) {
  const item = document.createElement('li');
  item.textContent = describe(departure);
  list.appendChild(item);
}

document.body.appendChild(list);
```

Everything below the function is DOM code from Module 3, with nothing new in it. Save, and the three departures appear on the page.

You might wonder why this code never uses `querySelector`. That is on purpose. `document.body` is always there, so TypeScript is happy with it as it is. Getting hold of elements with `querySelector` takes one extra step in TypeScript, and that step is what the next lesson is about.

<details>
<summary>Rabbit hole: createElement and querySelector in TypeScript</summary>

If you want a sneak peek at that extra step, you can try it in this project right now. It takes about ten minutes, and you can skip it without missing anything, because the next lesson starts from the beginning.

**createElement already knows what it made.** Hover over `item` inside the loop. TypeScript says it is an `HTMLLIElement`, a list item, and `list` is an `HTMLUListElement`. You never wrote either of those. `createElement('li')` makes a brand new element from a tag name, so TypeScript knows exactly what kind of element came out, and it is never `null`, because you just made it.

**querySelector does not.** It looks for something that might be in your HTML, and TypeScript has never read your HTML. So it cannot know whether anything was found, or what kind of element it is.

To see that, add an empty list to `index.html`, just under the `h1`:

```html
<ul id="board"></ul>
```

Then, in `main.ts`, replace everything from `const list = document.createElement('ul');` down to the end with this:

```ts
const boardList = document.querySelector('#board');

for (const departure of departures) {
  const item = document.createElement('li');
  item.textContent = describe(departure);
  boardList.appendChild(item);
}
```

There is a red line under `boardList`, saying `'boardList' is possibly 'null'`. TypeScript is pointing out that if there were no `#board` in the HTML, `boardList` would be `null`, and `null.appendChild` would crash.

The fix is to check, and stop with a clear message if it is missing:

```ts
const boardList = document.querySelector<HTMLUListElement>('#board');

if (!boardList) {
  throw new Error('The page has no #board list.');
}

for (const departure of departures) {
  const item = document.createElement('li');
  item.textContent = describe(departure);
  boardList.appendChild(item);
}
```

Two things changed.

The `if` is a check that runs. TypeScript reads it, sees that the code can only get past it if `boardList` is not `null`, and the red line disappears. Hover over `boardList` above the `if` and below it, and compare.

`<HTMLUListElement>` in angle brackets tells TypeScript what kind of element to expect. Without it, TypeScript only knows it is "some element", which is enough for `appendChild` but not for anything that only some elements have, like the `src` of an image or the `value` of an input.

Save, and the board looks exactly the same as before, now drawn into the list from the HTML. Both versions are fine. Use `createElement` when you are making something new, and `querySelector` with a check when you need something that is already in the HTML. The next lesson is about that check: why TypeScript insists on it, and what to do about the other things it cannot see.

</details>

---

## Part 5 - The check nobody ran

Now make a mistake on purpose. In the array, change Sandvik's delay from the number `5` to the text `'5'`, and save:

```ts
  { destination: 'Sandvik', time: '09:40', minutesLate: '5' },
```

### Exercise 1 - Predict, then look

The interface says `minutesLate` is a number, and now it is text. Before you look anywhere, predict three things:

1. What does VS Code show?
2. What does the page in the browser show?
3. Does the terminal running `npm run dev` say anything?

<details>
<summary>Solution</summary>

1. VS Code shows a red line under `minutesLate`, saying `Type 'string' is not assignable to type 'number'`.
2. The page shows the board as if nothing were wrong, with `09:40 Sandvik (5 min late)`. The bug just happens not to show here, because `'5' > 0` is still true in JavaScript, and joining `'5'` into a sentence looks the same as joining `5`. It would show up the moment anything did sums with it.
3. The terminal says nothing at all.

That is the first sentence of this lesson. Vite's job is to get your code into the browser, and it does that by removing the types without reading them. A type error does not stop it, and it does not even mention it. The red line in VS Code is the only warning you get.

</details>

So who actually checks? The checker is called `tsc`, short for "TypeScript compiler", and it was installed by `npm install`. In your second terminal, run it:

```text
npx tsc
```

`npx` means "run a tool that is installed in this project". After a moment, `tsc` prints the same error that VS Code showed, with the file name and the line number. That is all it does here: it reads your code, checks it, and reports.

Now try a build, which is how you make the finished version of a site:

```text
npm run build
```

It stops with the same error, and nothing gets built. Remember what `"build"` said in `package.json`: `tsc && vite build`. The `&&` means "only do the second one if the first one worked", much like `&&` in an `if`. So a type error can slip past the dev server, but it can never get into a finished build.

Change `'5'` back to `5`, save, and run `npx tsc` again. This time it prints nothing at all, and nothing means no errors.

Run `npm run build` again too. It works now, and a new folder called `dist` appears. That is the finished site, and if you open the `.js` file in `dist/assets`, you will find it squashed onto a few long lines, with not a single type left in it. The browser only ever gets plain JavaScript. You can delete `dist` whenever you like, since a build makes a fresh one.

<details>
<summary>A message you will meet: "declared but its value is never read"</summary>

At some point, `tsc` or VS Code will tell you something like `'total' is declared but its value is never read`. That is not a type error. It is an extra rule that Vite switches on in new projects: a variable you never use is usually a leftover or a mistake, so the project treats it as an error.

It is harmless while you are halfway through writing something. But the build will not run until you either use the variable or remove it.

</details>

---

## Self study task: two files, and a second list

### Level 1: move the data out

`main.ts` is doing two jobs: it holds the data, and it draws the page. Split them up, the way you learned with ES modules in lesson 3.2.

Make a new file in `src` called `departures.ts`. Move the interface, the array and the `describe` function into it, and `export` the array and the function. Then `import` them in `main.ts`, so that `main.ts` only has the import lines and the drawing code.

One detail: in a Vite project, you write the file ending in the import, like this: `from './departures.ts'`.

<details>
<summary>Solution</summary>

`src/departures.ts`:

```ts
interface Departure {
  destination: string;
  time: string;
  minutesLate: number;
}

export const departures: Departure[] = [
  { destination: 'Holmen', time: '08:15', minutesLate: 0 },
  { destination: 'Sandvik', time: '09:40', minutesLate: 5 },
  { destination: 'Rong', time: '11:05', minutesLate: 0 },
];

/**
 * Builds one line of text for the departure board.
 * @param departure The departure to describe.
 * @returns A line like "09:40 Sandvik (5 min late)".
 */
export function describe(departure: Departure): string {
  if (departure.minutesLate > 0) {
    return departure.time + ' ' + departure.destination + ' (' + departure.minutesLate + ' min late)';
  }
  return departure.time + ' ' + departure.destination;
}
```

`src/main.ts`:

```ts
import './style.css';
import { departures, describe } from './departures.ts';

const list = document.createElement('ul');

for (const departure of departures) {
  const item = document.createElement('li');
  item.textContent = describe(departure);
  list.appendChild(item);
}

document.body.appendChild(list);
```

Notice that `main.ts` never mentions `Departure` at all, and it is still fully checked. TypeScript follows the types through the import: it knows `departures` is a `Departure[]`, so it knows what `departure` is inside the loop. Hover over it and see.

</details>

<details>
<summary>Rabbit hole: what if main.ts needs the interface too?</summary>

If you `export` the interface and then try to import it the ordinary way, the project complains that `'Departure' is a type and must be imported using a type-only import`. In a Vite project, imports of types are written a little differently:

```ts
import type { Departure } from './departures.ts';
```

`import type` means "I only need this for checking, not for running". Since types disappear before the code runs, there is nothing to actually import, and this tells Vite it can skip it entirely. You will not need it in this task.

</details>

### Level 2: the late list

Under the first list, add a heading that says `Running late`, and a second list that only shows the departures that are more than 0 minutes late. Use `filter` from Module 2 to make the shorter array, and the same `describe` function for the text.

Check your work the way this lesson taught you: watch for red lines in VS Code while you write, and finish by running `npx tsc`.

<details>
<summary>Solution</summary>

Add this to the bottom of `main.ts`:

```ts
const lateHeading = document.createElement('h2');
lateHeading.textContent = 'Running late';
document.body.appendChild(lateHeading);

const lateDepartures = departures.filter((departure) => departure.minutesLate > 0);

const lateList = document.createElement('ul');

for (const departure of lateDepartures) {
  const item = document.createElement('li');
  item.textContent = describe(departure);
  lateList.appendChild(item);
}

document.body.appendChild(lateList);
```

The page shows Sandvik under the new heading.

If you noticed that the two loops are almost the same, that is a good eye. A function `drawList(list: Departure[]): void` that builds and adds one list would remove the repetition. To write that one in `main.ts`, you would need the type, so it is the place to try the `import type` line from the rabbit hole above.

</details>

---

## Where this leaves you

A Vite project has two tools doing two different jobs. Vite gets your code into the browser, and on the way it removes the types without reading them. `tsc` is the one that checks. So while you work, the red lines in VS Code are your checker, because the dev server will not warn you about anything. `npm run build` runs `tsc` first, so a type error can never get into a finished build.

The terminal is just a window for typing commands, and it always works inside one folder. You now know the four commands you need: `npm install` once, `npm run dev` while you work, `npx tsc` to check, and `npm run build` to make the finished site.

The next TypeScript extra lesson uses a project just like this one, and goes back to TypeScript itself: what to do about the things TypeScript cannot see, like your HTML, an answer from a server, and errors.
