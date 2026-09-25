# JavaScript 1 - Extra Lessons

Optional extra lessons for students taking JavaScript 1 at Noroff.

Each extra lesson covers the same ground as one or two of the regular Moodle lessons, but takes a different road there. It uses different examples, and it is usually built around two or three central ideas instead of a list of features. If the regular lessons made sense, an extra lesson should help them stick. If they didn't, the extra lesson might be the version that lands.

> [!NOTE]
> These lessons are extra material for your own benefit. They don't replace the regular lessons, and they are not part of what you are assessed on. Where an extra lesson and the course material seem to disagree, go with the course material, and let your teacher know.

## How to use these

- **Do the regular lessons first**, then use the matching extra lesson to go over the same ground again from another angle.
- **Type the code and run it.** Most lessons are built around one running example, and the understanding comes from running it and breaking it, not from reading.
- **Serve pages over HTTP.** From Module 4 onwards, open your pages with a local server (for example the Live Server extension in VS Code) rather than double-clicking the HTML file. Modules and `fetch` often won't work from `file://`.
- **The self study tasks are optional too.** Most lessons end with exercises and a small task. They are the best way to find out whether the idea has really landed.

## Module 1: Getting started with JavaScript

| Lesson | Covers | In short |
| --- | --- | --- |
| [One value at a time](Module-1/one-value-at-a-time.md) | Lessons 1.1 and 1.2 | Values, variables and types, all in the browser console, and why the plus sign sometimes adds and sometimes joins. |
| [Write the stop first](Module-1/write-the-stop-first.md) | Lessons 1.3 and 1.4 | Conditions and loops as one idea, using a hiking route with a fixed amount of daylight left. |

## Module 2: Arrays, functions and string properties

| Lesson | Covers | In short |
| --- | --- | --- |
| [The question decides the shape](Module-2/the-question-decides-the-shape.md) | Lessons 2.1 and 2.2 | Array or object? Choosing the shape of your data first, using the board game shelf at a village hall. |
| [Answer a question, or do a thing](Module-2/answer-a-question-or-do-a-thing.md) | Lessons 2.3 and 2.4 | Functions, parameters, return values and scope, continuing with the same board game shelf. |

## Module 3: Number and array methods, Object and ES6 modules

| Lesson | Covers | In short |
| --- | --- | --- |
| [Taking callbacks and array methods apart](Module-3/callbacks-and-array-methods.md) | Lessons 3.1 and 3.2 | Build `map`, `filter` and friends yourself with a `for` loop, then meet the real ones. |
| [Unpacking data, and getting out of one file](Module-3/unpacking-and-modules.md) | Lessons 3.3 and 3.4 | Destructuring, spread and rest, object methods and ES modules, built on three ideas instead of seven features. |

## Module 4: Create and update HTML, DOM events and managing web forms

| Lesson | Covers | In short |
| --- | --- | --- |
| [Look at the DOM, then draw it from your data](Module-4/the-dom.md) | Lessons 4.1 and 4.2 | Inspect the DOM before writing code against it, then draw the page from your data with one function. |
| [Events, or how to hand your code to the browser](Module-4/events.md) | Lessons 4.3 and 4.4 | Why the browser calls your functions, one listener instead of twenty, forms, and handlers that change data and redraw. |

## Module 5: API requests and async code

| Lesson | Covers | In short |
| --- | --- | --- |
| [Waiting without freezing](Module-5/async-api.md) | Lessons 5.1 and 5.2 | Callbacks, promises and `async`/`await`, by swapping the plumbing under one Studio Ghibli film board. |
| [Four verbs and one parcel](Module-5/http-methods.md) | Lessons 5.3 and 5.4 | GET, POST, PUT and DELETE as one request shape, status codes, and the Network tab at the moment you need it. |
| [The ticket, not the ID card](Module-5/tokens.md) | Extra, builds on the above | Logging in, the `Authorization` header, expired tokens, and causing a `401` on purpose before it happens by accident. |

## Module 6: Data storage, dates, destructuring objects and error handling

| Lesson | Covers | In short |
| --- | --- | --- |
| [Flat paper, and the date that came back wrong](Module-6/storage-and-dates.md) | Lessons 6.1 and 6.2 | `localStorage`, JSON and dates, and the bug where a saved date comes back as text. |
| [A note for later, and the net you left behind](Module-6/timers-and-errors.md) | Lessons 6.3 and 6.4 | Timers and error handling, and why a `try...catch` around a `setTimeout` catches nothing. |

## Module 7: Pagination, user experience, routing and search

| Lesson | Covers | In short |
| --- | --- | --- |
| [The question is the URL](Module-7/the-question-is-the-url.md) | Lessons 7.1 and 7.2 | Server-side search, filtering and pagination against the Art Institute of Chicago's open API. |
| [One belt, four stations](Module-7/one-belt-four-stations.md) | Lessons 7.3 and 7.4 | Filter, sort, paginate and render in the browser, in the right order, with the app's state kept in one place. |

## Questions and feedback

Questions about a lesson are welcome on Teams. If you spot something that's wrong, out of date or could be explained better, flag it there too, or open an issue in this repository.