# Extra lesson: The question is the URL

Lesson 7.1 teaches server-side pagination properly: query parameters, the `meta` object, Next and Previous, numbered buttons. Then it has to admit that the `/old-games` endpoint holds two records, so `page` and `limit` change nothing. You build the machinery and never get to see it work.

This lesson gives it something real to work on. The Art Institute of Chicago publishes its entire collection through a free API with no key and no sign-up, and it is large enough that you have no choice but to let the server do the slicing.

Open this in a browser tab and leave it open, because we will keep coming back to it:

```text
https://api.artic.edu/api/v1/artworks?limit=2
```

## The three sentences

1. **When the server holds the data, every control on your page adds a word to the request.**
2. **You only ever hold one page.**
3. **Whatever the server will not do for you, you cannot do at all.**

Four jobs have to happen to get twelve artworks onto your screen: search, filter, sort and paginate. None of them will happen in your browser. They all happen in Chicago, on a machine you will never see, and your job is not to run them but to describe what you want clearly enough that someone else can.

That is the whole difference. Later this week you will build all four of those jobs yourself, in the browser, over a dataset small enough to hold. Notice as you go which parts of today's code would survive that move and which parts exist only because the data is somewhere else.

The picture to hold on to is a request slip at a library desk. You do not walk into the stacks. You write down what you want on a slip, hand it over, and someone brings back one tray. If the slip has no box for "sorted by date", you do not get sorted by date.

## Part 1: Measure before you decide

The module gives you a rule of thumb for choosing client-side or server-side: "less than a megabyte". That is not something you can act on unless you know how big the thing is.

This API will tell you, almost for free. Ask for zero records and it hands back the count and an empty list:

```text
https://api.artic.edu/api/v1/artworks?limit=0
```

Try it. At the time of writing it reports `"total": 132681`.

Now do the arithmetic out loud. Even trimmed to five fields, each record is roughly a hundred bytes of JSON. 132,681 records is somewhere north of thirteen megabytes, and untrimmed it is far worse. Downloading that before showing the visitor anything is not a judgement call. It is simply not an option.

Compare that with the datasets you have worked with so far, which were small enough to sit in a `const` at the top of a file. Later this week you will meet one of those again and the arithmetic will come out the other way. Same question, same three seconds of work, opposite answer, and the thing that decided it was a single number rather than a rule of thumb.

This is the honest version of "less than a megabyte": ask, multiply, decide.

## Part 2: The request slip

Make a folder with `index.html` and `script.js`.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Collection browser</title>
  </head>
  <body>
    <h1>Art Institute of Chicago collection</h1>

    <label for="search">Search the collection:</label>
    <input type="search" id="search" placeholder="e.g. harbour, still life" />

    <label for="publicDomain">
      <input type="checkbox" id="publicDomain" /> Public domain only
    </label>

    <p id="count"></p>
    <div id="results"></div>
    <div id="pager"></div>

    <script src="script.js"></script>
  </body>
</html>
```

Three query parameters do most of the work.

- `page` is which tray you want, counting from 1.
- `limit` is how many records go in the tray. This API caps it at 100.
- `fields` is a comma-separated list of the fields you actually want back.

`fields` is not optional in any practical sense. A full artwork record from this API is enormous. Ask for five fields and you get five fields:

```text
https://api.artic.edu/api/v1/artworks?limit=2&fields=id,title,artist_title,date_display,image_id
```

The response has three parts we care about:

```json
{
  "pagination": {
    "total": 132681,
    "limit": 2,
    "offset": 0,
    "total_pages": 66341,
    "current_page": 1,
    "next_url": "..."
  },
  "data": [ ... ],
  "config": { "iiif_url": "https://www.artic.edu/iiif/2" }
}
```

`data` is your tray. `pagination` is the note that comes with it telling you where in the stacks you are. `config` we will use later for pictures.

Before writing any of it, one decision about where the app keeps what it knows.

The tempting approach is to scatter it: a `let currentPage` here, read the search term out of the input box there, check the checkbox's `checked` property somewhere else. It works, and it falls apart the moment two of those have to agree with each other.

Instead, put everything the app knows in one object:

```js
const API_BASE = 'https://api.artic.edu/api/v1/artworks';
const FIELDS = 'id,title,artist_title,date_display,image_id';

const state = {
  term: '',
  publicDomainOnly: false,
  page: 1,
  perPage: 12,
};
```

Read that object and you know exactly what the page is currently showing. More to the point here, read that object and you know exactly what the next URL will say, because those four fields are precisely the four things that go on the request slip. The state and the URL are the same information written two ways, and a function to convert between them is coming in Part 5.

## Part 3: The trap in the numbered pager

Before writing the pager, look at that number again: `total_pages` is 66,341 at two per page, and about 11,057 at twelve per page.

The module's `renderPaginationNumbers` does this:

```js
for (let i = 1; i <= meta.pageCount; i++) {
  // create a button
}
```

Point that at this API and you are asking the browser to build eleven thousand buttons every time anybody clicks anything. **Do it.** Write that loop, run it, and watch the tab go quiet for a second or two. It is worth seeing once, because it is the single most common way a pagination UI dies in the wild.

Real sites show a short run of pages around wherever you are. Five is plenty:

```js
/**
 * Picks a short run of page numbers to show around the current page.
 * @param {number} current The page we are on.
 * @param {number} totalPages How many pages we can reach.
 * @returns {Array<number>} At most five page numbers.
 */
function pageWindow(current, totalPages) {
  const pages = [];
  let first = current - 2;
  let last = current + 2;

  if (first < 1) {
    first = 1;
    last = 5;
  }

  if (last > totalPages) {
    last = totalPages;
    first = totalPages - 4;
  }

  if (first < 1) {
    first = 1;
  }

  for (let i = first; i <= last; i++) {
    pages.push(i);
  }

  return pages;
}
```

The two `if` blocks are the whole idea: slide the window when it falls off either end, and the last `if` catches the case where there are fewer than five pages in total.

There is a second limit. This API will not let you reach beyond ten thousand records through any combination of `page` and `limit`. So `total_pages` is what exists, and this is what you can actually get to:

```js
const MAX_RECORDS = 10000;

/**
 * Works out how many pages we are actually allowed to reach.
 * @param {number} totalPages The total_pages figure the museum reported.
 * @returns {number} The number of pages we can really ask for.
 */
function usablePages(totalPages) {
  const allowed = Math.floor(MAX_RECORDS / state.perPage);

  if (totalPages > allowed) {
    return allowed;
  }

  return totalPages;
}
```

At twelve per page that is 833 pages rather than 11,057. Building the pager from `total_pages` would offer visitors thousands of buttons that return an error when clicked.

<details>
<summary>Rabbit hole: why there is a wall at all</summary>

To hand you page 900, a search engine generally has to work out the first 900 pages and throw away 899 of them. The deeper the page, the more work per request, and the cost grows with the page number rather than staying flat. So most large search systems draw a line somewhere and refuse to go past it.

This is why "jump to the last page" quietly vanished from most big sites over the last decade, and why infinite scroll and "load more" became popular. They only ever ask for what comes next, which is cheap, instead of asking for a page deep in the middle, which is not.

If you ever need everything, the answer is not to paginate 11,000 times. This museum, like many, publishes nightly data dumps for exactly that purpose, and asks that you use those instead of hammering the API.

</details>

## Part 4: Search moves the endpoint

Here is the part that catches people out. Searching is not another parameter on the same URL. It is a different endpoint:

```text
https://api.artic.edu/api/v1/artworks              <- listing
https://api.artic.edu/api/v1/artworks/search?q=    <- search
```

Both are paginated the same way and both return the same `pagination` block, so everything you have written still works. But your URL builder has to choose between two addresses.

And notice what has just happened to your search box. If the whole collection were sitting in the browser, typing a letter would run `filter` over an array in memory, which costs nothing at all. Here, typing a letter sends a request across the Atlantic. Type "harbour" and you have sent seven requests, six of which you did not want. That is the real argument for debouncing, and it is waiting for you in lesson 7.4.

## Part 5: The trap in the URL

Build the URL the obvious way and it works until someone searches for something with a space in it.

Try this in the console before reading on. What do you think the server receives?

```js
const term = 'still life & flowers';
const url = `https://api.artic.edu/api/v1/artworks/search?q=${term}&limit=12`;
```

<details>
<summary>What the server actually receives</summary>

The `&` in the search term ends the `q` parameter early. The server sees three parameters:

- `q` is `"still life "`
- `" flowers"` is a parameter in its own right, with an empty value, which the API ignores
- `limit` is `12`

So the visitor searched for "still life & flowers" and the museum was asked about "still life ". No error, no warning, just a quietly wrong answer. Those are the worst kind.

A `#` in the term would be worse still: everything after it never leaves the browser at all.

</details>

The fix is a tool built for the job. `URLSearchParams` holds a set of key and value pairs and turns them into a correctly encoded query string:

```js
/**
 * Builds the request URL from the current state.
 * @returns {string} A full URL, ready to hand to fetch.
 */
function buildUrl() {
  const params = new URLSearchParams();
  params.set('fields', FIELDS);
  params.set('limit', state.perPage);
  params.set('page', state.page);

  const searching = state.term !== '';

  if (searching) {
    params.set('q', state.term);
  }

  if (searching) {
    return `${API_BASE}/search?${params}`;
  }

  return `${API_BASE}?${params}`;
}
```

You never touch `?` or `&` yourself, and you never think about encoding. Putting `params` inside a template literal calls its `toString` for you, which is why there is no `.toString()` in that last line.

Run the trap again through `URLSearchParams` and the space becomes `+` and the ampersand becomes `%26`, so the whole phrase arrives intact.

## Part 6: A filter on top of the search

Now the useful part. Most of the collection is still in copyright, which means you can read the metadata but you cannot show the picture. About half of it is public domain, released under CC0, and those you can display freely.

The search endpoint accepts a filter alongside the search term:

```text
https://api.artic.edu/api/v1/artworks/search?q=harbour&query[term][is_public_domain]=true
```

Combine it with the counting trick from Part 1 and you can see the shape of the collection without downloading any of it:

```text
https://api.artic.edu/api/v1/artworks?limit=0
https://api.artic.edu/api/v1/artworks/search?query[term][is_public_domain]=true&limit=0
```

At the time of writing: 132,681 works in total, 62,046 of them public domain.

Adding it to the builder is two changes. A checkbox in the state, and one more line on the slip:

```js
function buildUrl() {
  const params = new URLSearchParams();
  params.set('fields', FIELDS);
  params.set('limit', state.perPage);
  params.set('page', state.page);

  const searching = state.term !== '';

  if (searching) {
    params.set('q', state.term);
  }

  if (state.publicDomainOnly) {
    params.set('query[term][is_public_domain]', 'true');
  }

  if (searching || state.publicDomainOnly) {
    return `${API_BASE}/search?${params}`;
  }

  return `${API_BASE}?${params}`;
}
```

The last `if` is the fiddly bit and worth reading twice. The filter only exists on the search endpoint, so ticking the box sends you to `/search` even when the search box is empty. That is fine: `/artworks/search` with no `q` and a filter returns the whole filtered collection.

Those square brackets look alarming but they are just part of the parameter name as far as your code is concerned. `URLSearchParams` encodes them to `%5B` and `%5D` and the server unpacks them at the other end. You do not need to understand the query language behind them to use this one documented filter.

## Part 7: The rest of the app

One function runs everything, and every listener does exactly two things: change the state, then call that function.

That pattern is worth naming, because it is what stops this app turning into a tangle. `update()` is the only place that knows how to turn the state into a screen. Nothing else fetches, nothing else draws, and no listener ever tries to work out what should change. It sets a field and asks for the whole picture again.

```js
const searchInput = document.querySelector('#search');
const publicDomainBox = document.querySelector('#publicDomain');
const results = document.querySelector('#results');
const pager = document.querySelector('#pager');
const count = document.querySelector('#count');

function drawArtworks(list) {
  results.innerHTML = '';

  if (list.length === 0) {
    results.textContent = 'Nothing in the collection matches that.';
    return;
  }

  list.forEach((artwork) => {
    const row = document.createElement('p');
    const artist = artwork.artist_title;
    const shownArtist = artist === null ? 'Artist unknown' : artist;
    row.textContent = `${artwork.title} - ${shownArtist}, ${artwork.date_display}`;
    results.appendChild(row);
  });
}

function drawPager(pagination) {
  const totalPages = usablePages(pagination.total_pages);
  pager.innerHTML = '';

  pageWindow(state.page, totalPages).forEach((number) => {
    const button = document.createElement('button');
    button.textContent = number;
    button.dataset.page = number;

    if (number === state.page) {
      button.classList.add('active');
    }

    pager.appendChild(button);
  });
}

function drawCount(pagination) {
  if (pagination.total === 0) {
    count.textContent = 'No matches.';
    return;
  }

  const reachable = usablePages(pagination.total_pages);
  count.textContent = `${pagination.total} works found, page ${state.page} of ${reachable}`;
}

async function update() {
  results.textContent = 'Asking the museum...';
  pager.innerHTML = '';

  try {
    const response = await fetch(buildUrl());

    if (!response.ok) {
      throw new Error(`The museum said no. Status: ${response.status}`);
    }

    const result = await response.json();
    drawArtworks(result.data);
    drawPager(result.pagination);
    drawCount(result.pagination);
  } catch (error) {
    results.textContent = `Could not load the collection. ${error.message}`;
    count.textContent = '';
  }
}

searchInput.addEventListener('input', () => {
  state.term = searchInput.value.trim();
  state.page = 1;
  update();
});

publicDomainBox.addEventListener('change', () => {
  state.publicDomainOnly = publicDomainBox.checked;
  state.page = 1;
  update();
});

pager.addEventListener('click', (event) => {
  if (event.target.tagName === 'BUTTON') {
    state.page = Number(event.target.dataset.page);
    update();
  }
});

update();
```

`artist_title` is often `null`, because a lot of the collection is by makers nobody recorded. Checking for it is not defensive programming for its own sake, it is what the data is actually like.

Note the `state.page = 1` in both listeners. Without it, a visitor on page 40 who then searches for something with two results stays on page 40, and page 40 of two results is nothing at all. They would get "Nothing in the collection matches that" for a search that clearly matched.

Resetting the page is not a special case to remember. It is part of what "a new question" means. And here it costs more than a blank screen: it is a wasted round trip to Chicago to be told about records that do not exist.

## Part 8: The catch

You now have pagination, search and a filter, all running on the server. You do not have sorting, and you are not going to get it.

This API has no `sort` parameter you can use from a beginner's toolkit. And here is the thing worth sitting with: **you cannot fix that from the browser.** You could call `sort` on `result.data`, and it would run without complaint, and the twelve works on screen would come out in a tidy order. But you would have sorted a page, not a collection. Page 2 would start over from wherever it happened to start. The result is a list that looks ordered and is not, which is worse than an obviously unordered list.

That is sentence three, and it is the real trade-off that lesson 7.1 never spells out:

- **Client-side**: you hold everything, so you can search, filter and sort instantly, in any way you can think of. Only possible if everything fits.
- **Server-side**: scales to any size, but every feature you offer has to be one the API already supports. If it will not sort, you do not sort.

This is why so many large catalogue sites have a sort dropdown with exactly four options, and why those four never change. Someone chose them at the back end, and the front end is not allowed to invent a fifth.

Later this week you will take the other road, with a dataset small enough to hold in a variable, and you will get all four jobs back: search, filter, sort and paginate, all yours to run. Keep the sorting problem above in mind when you do, because it does not disappear just because the data is local. It only stops being the server's fault.

## Exercises

### Exercise 1: Read the meter

Use `limit=0` and nothing else. No code, just URLs in the address bar.

1. How many works are in the collection in total?
2. How many are public domain?
3. How many match a search for something you are interested in? Try a few words.
4. For your search in step 3, work out roughly how many megabytes it would be to download all the matches at about a hundred bytes each. Would client-side handling be reasonable? At what number of results would you change your mind?

<details>
<summary>Solution</summary>

```text
https://api.artic.edu/api/v1/artworks?limit=0
https://api.artic.edu/api/v1/artworks/search?query[term][is_public_domain]=true&limit=0
https://api.artic.edu/api/v1/artworks/search?q=harbour&limit=0
```

At the time of writing the first two give 132,681 and 62,046.

For step 4 there is no single right answer, which is the point. A rough guide: a few hundred results is comfortably client-side, a few thousand is a judgement call that depends on how many fields you ask for, and tens of thousands is not.

The thing to take away is that you now have a way of asking rather than guessing, and it costs one request with almost no payload.

</details>

### Exercise 2: Let the visitor choose the page size

Add a dropdown offering 12, 24 and 48 works per page.

```html
    <label for="perPage">Per page:</label>
    <select id="perPage">
      <option value="12">12</option>
      <option value="24">24</option>
      <option value="48">48</option>
    </select>
```

Two things to get right. Changing the page size must reset to page 1, for the same reason a search does. And watch what happens to the number of reachable pages when you switch from 12 to 48.

<details>
<summary>Solution</summary>

```js
const perPageSelect = document.querySelector('#perPage');

perPageSelect.addEventListener('change', () => {
  state.perPage = Number(perPageSelect.value);
  state.page = 1;
  update();
});
```

`Number()` matters. The value from a `<select>` is always a string, and `usablePages` divides by it. `10000 / '48'` happens to work because JavaScript coerces it, but `(page - 1) * perPage` with a string would give you string concatenation somewhere else down the line. Convert at the edge, once, and then trust it.

On the reachable pages: at 12 per page you can reach 833 pages, and at 48 you can reach 208. The wall is at ten thousand records, not ten thousand pages, so a bigger page size gets you the same distance into the collection in fewer clicks. That is a genuinely good reason to offer the control.

</details>

### Exercise 3: Walk into the wall

Temporarily change your pager to build its buttons from `pagination.total_pages` rather than `usablePages(...)`. Then navigate to a page beyond 833 and see what the museum says.

Predict first. Will you get an empty list, an error status, or a page of results?

<details>
<summary>What happens, and what to do about it</summary>

You get an error response rather than an empty list, so `response.ok` is `false` and your `catch` block shows the message. If you had not written that check, `response.json()` would have handed you something with no `data` array and the failure would have surfaced later, somewhere less obvious, as a confusing error about reading a property of `undefined`.

This is the value of `if (!response.ok) throw` that lesson 7.1 slips in without much comment. It converts a wrong answer into a loud one, at the moment it happens.

Put `usablePages` back afterwards. The correct fix is not to handle the error more gracefully, it is to never offer the visitor a button that cannot work.

</details>

## Self study task

### Level 1: Show the pictures

Public domain works can actually be displayed. Each record has an `image_id`, and the response has an `iiif_url` in its `config` block. Put them together like this:

```text
{iiif_url}/{image_id}/full/200,/0/default.jpg
```

Add the image to each result. Two things to handle: `image_id` is `null` for plenty of records, and you should only be displaying images for works that are public domain.

<details>
<summary>Solution</summary>

```js
function drawArtworks(list, iiifUrl) {
  results.innerHTML = '';

  if (list.length === 0) {
    results.textContent = 'Nothing in the collection matches that.';
    return;
  }

  list.forEach((artwork) => {
    const row = document.createElement('p');
    const artist = artwork.artist_title;
    const shownArtist = artist === null ? 'Artist unknown' : artist;
    row.textContent = `${artwork.title} - ${shownArtist}, ${artwork.date_display}`;
    results.appendChild(row);

    if (state.publicDomainOnly && artwork.image_id !== null) {
      const picture = document.createElement('img');
      picture.src = `${iiifUrl}/${artwork.image_id}/full/200,/0/default.jpg`;
      picture.alt = artwork.title;
      results.appendChild(picture);
    }
  });
}
```

And in `update()`, pass the value through rather than hardcoding it:

```js
drawArtworks(result.data, result.config.iiif_url);
```

The museum asks that you read `iiif_url` from `config` rather than hardcoding it, so that they can move their image server without breaking your page. Reading it costs nothing, since it arrives in every response anyway.

The `200,` in the URL is the width in pixels. The museum recommends sticking to 200, 400, 600 and 843, because those sizes are already cached on their servers and anything else has to be generated on demand.

</details>

### Level 2: Loading, empty and broken

When the data is already in the browser, these three barely matter, because everything is instant and nothing can fail halfway. Here every click is a trip across the Atlantic and any of it can go wrong.

Handle all three properly: a loading state while a request is in flight, a clear empty state when there are no matches, and a real error state when the request fails.

To test the last one honestly, open DevTools, go to the Network tab, and set the throttling dropdown to Offline. Then click a page button.

<details>
<summary>Notes on the solution</summary>

The code in Part 7 already covers the basic shape: set a loading message before the `await`, catch failures, and check `list.length === 0` separately. Two refinements worth making.

First, clear the pager while loading as well as the results. Otherwise the old page buttons sit there looking clickable, and an impatient visitor will click one and queue a second request on top of the first.

Second, distinguish the two kinds of failure in your message. `response.ok` being false means the museum answered and said no, so a status code is useful. A thrown network error means the request never arrived, so the honest message is about the connection, not the museum.

```js
    const response = await fetch(buildUrl());

    if (!response.ok) {
      throw new Error(`The museum refused the request. Status: ${response.status}`);
    }
```

Offline mode throws before it ever reaches that check, so the `catch` receives a `TypeError` about a failed fetch, and your message should not blame Chicago for the visitor's wifi.

</details>

### Level 3: Now debouncing is worth it

Count the requests. Open the Network tab, clear it, and type "harbour" into your search box. Seven letters, seven requests, six of them for search terms nobody wanted.

Add a debounce so that the request only goes out once the visitor stops typing. Lesson 7.4 gives you the function. Use a 300ms delay, then watch the Network tab again.

<details>
<summary>Solution, and why this is the right place for it</summary>

```js
function debounce(func, wait) {
  let timeoutId;

  return function (...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      func(...args);
    }, wait);
  };
}

function handleSearch() {
  state.term = searchInput.value.trim();
  state.page = 1;
  update();
}

searchInput.addEventListener('input', debounce(handleSearch, 300));
```

Typing "harbour" now sends one request instead of seven.

Lesson 7.4 introduces debouncing on a project where all the data is already in the browser, and then concedes that the performance impact there is minimal. It is right to concede that: adding a 300ms delay to something that was already instant makes the page feel slower, not faster.

Here it is not minimal. You are cutting six network requests, six waits, and six lots of load on a museum that is giving you this for free and asks for no more than sixty requests a minute. Debouncing does not become worthwhile because the operation is expensive to compute. It becomes worthwhile when the operation leaves the machine.

Note that this version drops the `this` handling from the module's debounce. Look at what calls it: an arrow-free named function used only as an event handler, which reads `searchInput` directly rather than from `event.target`. Nothing needs `this`, so nothing needs `apply`.

</details>

<details>
<summary>Rabbit hole: being a good guest</summary>

This API is free, needs no key, and is run by a museum rather than a company selling you something. That is worth not abusing.

Their published guidelines, in short:

- Sixty requests per minute for anonymous users. Debouncing your search box is most of the way to respecting that.
- Always use `fields` to ask only for what you need.
- Cache responses where you can, rather than re-requesting the same page.
- Consider adding an `AIC-User-Agent` header naming your project and a contact email. Not required, but it means they can get in touch rather than just blocking you if something you wrote goes wrong.

Public APIs disappear mainly because they get expensive to run. Being cheap to serve is how they stay.

</details>

<details>
<summary>Rabbit hole: why search is a separate endpoint</summary>

`/artworks` and `/artworks/search` almost certainly talk to two different systems. Listing pages come from a database, which is good at "give me rows 24 to 36 in a stable order". Search comes from a search engine, which is good at "score every record for relevance to the word harbour, then give me the best twelve".

You can see the seam in the response. Search results carry a `_score` field that listing results do not have, because relevance is a thing the search engine invented and the database has no opinion about.

This also explains the sorting situation from Part 8. Sorting by relevance is what the search engine does by default; sorting by anything else means asking it to ignore relevance entirely, which is a different request built a different way.

</details>
