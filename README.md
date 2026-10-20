# JavaScript 1 - Extra Lessons

Optional extra lessons for students taking JavaScript 1 at Noroff.

Each extra lesson covers the same ground as one or two of the regular Moodle lessons, but takes a different road there. It uses different examples, and it is usually built around two or three central ideas instead of a list of features. If the regular lessons made sense, an extra lesson should help them stick. If they didn't, the extra lesson might be the version that lands.

> [!NOTE]
> These lessons are extra material for your own benefit. They don't replace the regular lessons, and they are not part of what you are assessed on. Where an extra lesson and the course material seem to disagree, go with the course material, and let your teacher know.

## How to use these

- **Do the regular lessons first**, then use the matching extra lesson to go over the same ground again from another angle.
- **Type the code and run it.** Most lessons are built around one running example, and the understanding comes from running it and breaking it, not from reading.
- **Serve pages over HTTP.** From Module 3 onwards, open your pages with a local server (for example the Live Server extension in VS Code) rather than double-clicking the HTML file. Modules and `fetch` often won't work from `file://`.
- **On Windows?** If a terminal command such as `npm` stops with an error about running scripts being disabled, see [the Windows section](Module-7/typescript-with-vite.md#on-windows-the-error-about-running-scripts) of the terminal and Vite lesson. It also explains how to use Git Bash in VS Code, if you want a terminal that works like the one on a Mac.
- **The self study tasks are optional too.** Most lessons end with exercises and a small task. They are the best way to find out whether the idea has really landed.

## Module 1: Values, conditions, loops and arrays

| Lesson | Covers | In short |
| --- | --- | --- |
| [One value at a time](Module-1/one-value-at-a-time.md) | Lessons 1.1 and 1.2 | Values, variables and types, all in the browser console, and why the plus sign sometimes adds and sometimes joins. |
| [Write the stop first](Module-1/write-the-stop-first.md) | Lessons 1.3 and 1.4 | Conditions and loops as one idea, using a hiking route with a fixed amount of daylight left. |

## Module 2: Objects, functions and array methods

| Lesson | Covers | In short |
| --- | --- | --- |
| [The question decides the shape](Module-2/the-question-decides-the-shape.md) | Lessons 1.5 and 2.1 | Array or object? Choosing the shape of your data first, using the board game shelf at a village hall. |
| [Answer a question, or do a thing](Module-2/answer-a-question-or-do-a-thing.md) | Lessons 2.2 and 2.3 | Functions, parameters, return values and scope, continuing with the same board game shelf. |
| [Taking callbacks and array methods apart](Module-2/callbacks-and-array-methods.md) | Lessons 2.4 and 2.5 | Build `map`, `filter` and friends yourself with a `for` loop, then meet the real ones. |

## Module 3: Destructuring, modules and the DOM

| Lesson | Covers | In short |
| --- | --- | --- |
| [Unpacking data, and getting out of one file](Module-3/unpacking-and-modules.md) | Lessons 3.1 and 3.2 | Destructuring, spread and rest, object methods and ES modules, built on three ideas instead of seven features. |
| [Look at the DOM, then draw it from your data](Module-3/the-dom.md) | Lessons 3.3 and 3.4 | Inspect the DOM before writing code against it, then draw the page from your data with one function. |

## Module 4: Events, async code and APIs

| Lesson | Covers | In short |
| --- | --- | --- |
| [Events, or how to hand your code to the browser](Module-4/events.md) | Lessons 4.1 and 4.2 | Why the browser calls your functions, one listener instead of twenty, forms, and handlers that change data and redraw. |
| [Waiting without freezing](Module-4/async-api.md) | Lessons 4.3 and 4.4 | Callbacks, promises and `async`/`await`, by swapping the plumbing under one Studio Ghibli film board. |
| [Four verbs and one parcel](Module-4/http-methods.md) | Lesson 4.5, and the Network tab from 5.1 | GET, POST, PUT and DELETE as one request shape, status codes, and the Network tab at the moment you need it. |
| [The ticket, not the ID card](Module-4/tokens.md) | Extra, builds on the above | Logging in, the `Authorization` header, expired tokens, and causing a `401` on purpose before it happens by accident. |

## Module 5: Debugging, storage, dates and error handling

| Lesson | Covers | In short |
| --- | --- | --- |
| [Flat paper, and the date that came back wrong](Module-5/storage-and-dates.md) | Lessons 5.2 and 5.3 | `localStorage`, JSON and dates, and the bug where a saved date comes back as text. |
| [A note for later, and the net you left behind](Module-5/timers-and-errors.md) | Lessons 5.4 and 5.5 | Timers and error handling, and why a `try...catch` around a `setTimeout` catches nothing. |

## Module 6: Pagination, filtering, search and sorting

| Lesson | Covers | In short |
| --- | --- | --- |
| [The question is the URL](Module-6/the-question-is-the-url.md) | Lessons 6.1 and 6.2 | Server-side search, filtering and pagination against the Art Institute of Chicago's open API. |
| [One belt, four stations](Module-6/one-belt-four-stations.md) | Lessons 6.3 and 6.4 | Filter, sort, paginate and render in the browser, in the right order, with the app's state kept in one place. |

## Module 7: TypeScript

| Lesson | Covers | In short |
| --- | --- | --- |
| [Claims, checked before running](Module-7/typescript-in-the-playground.md) | Lessons 7.1 to 7.4 | Types, inference, interfaces and typed functions in the TypeScript Playground, with nothing to install. |
| [Two jobs, two tools](Module-7/typescript-with-vite.md) | The setup in lesson 7.1 | The terminal and a Vite project one step at a time, and why a type error does not stop your page from running. |
| [Check it, and TypeScript believes you](Module-7/checking-what-typescript-cannot-see.md) | Lesson 7.5 | Type guards and generics, in a small page that fetches dog photos. |

> [!NOTE]
> The course was reorganised in autumn 2026, and some of these lessons moved to a different module. If you have an old link, the page it points to tells you where the lesson lives now.
>
> The lesson numbers in the tables and inside the lessons follow the new structure. If you are still on the old one, most lessons now come a little earlier than before: for example, the async lesson that went with 5.1 and 5.2 now goes with 4.3 and 4.4. If the numbers do not match your Moodle, go by the topic.

## Questions and feedback

Questions about a lesson are welcome on Teams. If you spot something that's wrong, out of date or could be explained better, flag it there too, or open an issue in this repository.