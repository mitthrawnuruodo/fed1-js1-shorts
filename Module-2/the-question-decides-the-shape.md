# Extra lesson: The question decides the shape

This is an optional extra lesson covering the same ground as lessons 2.1 and 2.2, by a different road.

The regular lessons show you arrays, then objects, and compare them once you have met both. This one starts with the choice. You have some real information in front of you and you have to decide which shape to put it in, and the syntax follows from that decision rather than coming first.

Two sentences carry the lesson:

**1. An array is for things you count through. An object is for one thing you look things up on.**

**2. An index is not which one it is. It is how far along it is.**

The running example is the board game shelf at a village hall. People bring games, leave them on the shelf, and borrow them for game night.

---

## Part 1: Two questions

Here is what is actually on the shelf.

```
Fjord Traders          3 players or more    90 minutes    all pieces present
Hare and Hound         2 players or more    20 minutes    all pieces present
The Lighthouse Keeper  2 players or more    45 minutes    a piece is missing
Snow Line              4 players or more    60 minutes    all pieces present
```

Now, two questions someone might ask about that shelf.

**"What is the third game along?"**

To answer that, you count. First, second, third. The thing that identifies the game is where it sits, and if somebody reshuffles the shelf, the answer changes. This is a job for an **array**.

**"How long does Fjord Traders take?"**

To answer that, you do not count anything. You find the right game and read off one particular fact about it. The thing that identifies the fact is its name, and it does not matter what order the facts are written in. This is a job for an **object**.

That is the whole distinction, and it is worth settling before you write any brackets:

- Positions, in an order, all the same sort of thing: an **array**.
- Named facts about one single thing: an **object**.

Most real data ends up needing both, and we will get there at the end of this lesson. But you cannot pick the combination sensibly until you can pick each one on its own.

---

## Part 2: Arrays are positions

You make an array with square brackets, with commas between the items.

```js
const gamesOnShelf = ['Fjord Traders', 'Hare and Hound', 'The Lighthouse Keeper', 'Snow Line'];

console.log(gamesOnShelf);
```

The console prints the whole list, and in the browser you can click the little triangle to expand it and see each item with its position.

The items can be any type you have met.

```js
const playingTimes = [90, 20, 45, 60];
const allPiecesPresent = [true, true, false, true];
```

They can even be mixed, though a list of unrelated things is usually a sign that you wanted an object.

### Getting one item out

You ask for an item by its position, in square brackets.

```js
const gamesOnShelf = ['Fjord Traders', 'Hare and Hound', 'The Lighthouse Keeper', 'Snow Line'];

console.log(gamesOnShelf[0]);
console.log(gamesOnShelf[2]);
```

```
Fjord Traders
The Lighthouse Keeper
```

That number is called the index, and it starts at 0. Every beginner trips over this, and being told to memorise it does not help much. So here is a way of thinking about it that means you do not have to.

**An index is not which one it is. It is how far along it is.**

Picture the shelf with a mark at the left-hand end. `Fjord Traders` is right at the mark, so it is zero places along. `Hare and Hound` is one place along. `The Lighthouse Keeper` is two places along.

Once you read the number as a distance rather than a count, everything else about indexes stops needing to be memorised. The first item is at 0 because it has not moved. And in a shelf of four games, the last one is three places along, which is one less than four. That fact comes up constantly, and we will use it in a moment.

### Asking for something that is not there

```js
const gamesOnShelf = ['Fjord Traders', 'Hare and Hound', 'The Lighthouse Keeper', 'Snow Line'];

console.log(gamesOnShelf[9]);
```

```
undefined
```

No error. JavaScript shrugs and hands you `undefined`, the same value you met in the previous lesson meaning "there is nothing here".

This is worth being wary of. An error would stop your program at the exact line where you made the mistake, which is annoying but honest. `undefined` lets the mistake travel. It gets stored in a variable, joined into a message, passed further along, and by the time anything looks wrong you are three steps away from the line that caused it. If you ever see the word `undefined` where a game title should be, an index that reached past the end of an array is one of the first things to suspect.

### Changing an item

The same square brackets, with an assignment.

```js
const gamesOnShelf = ['Fjord Traders', 'Hare and Hound', 'The Lighthouse Keeper', 'Snow Line'];

gamesOnShelf[1] = 'Nine Men\'s Morris';

console.log(gamesOnShelf);
```

```
[ 'Fjord Traders', "Nine Men's Morris", 'The Lighthouse Keeper', 'Snow Line' ]
```

Somebody swapped the second game for a different one. Note the backslash before the apostrophe in `Nine Men\'s Morris`, for the reason covered in the first extra lesson.

<details>

<summary>Rabbit hole: typeof says an array is an object</summary>

Try this and you get an answer that looks wrong:

```js
const gamesOnShelf = ['Fjord Traders', 'Hare and Hound'];

console.log(typeof gamesOnShelf);
```

```
object
```

Not `array`. This is not a mistake in your code, and it is not really a mistake in JavaScript either, though plenty of people would call it one.

Under the bonnet, an array **is** a kind of object. It is an object that has been given a set of extra abilities to do with order and position, and a special way of printing itself. You will understand exactly what that means later in the course. For now the practical consequence is simple: `typeof` cannot tell an array from an object, so do not reach for it when that is the question you are asking.

There is a proper tool for the job, and since the course does not stop to teach it anywhere, here it is:

```js
console.log(Array.isArray(gamesOnShelf));
```

```
true
```

`Array.isArray` takes one thing and answers a yes or no question about it: is this an array? It is the only reliable way to tell, and unlike `typeof` it is not an operator but a function you call with brackets.

You will rarely need it while everything in your program is data you wrote yourself, because you already know what shape it is. It starts to matter when data arrives from somewhere else and you are not certain what you have been handed.

The main reason it is here is so that when you see `typeof` report `object` for an array and think something has gone wrong, you know it has not.

</details>

---

## Part 3: The shelf is a stack

Two things you will want to know about any array: how many items are in it, and how to add or remove one.

### How many

`.length` tells you how many items an array is holding.

```js
const gamesOnShelf = ['Fjord Traders', 'Hare and Hound', 'The Lighthouse Keeper', 'Snow Line'];

console.log(gamesOnShelf.length);
```

```
4
```

No brackets after `length`. It is a fact about the array rather than something the array does, and the difference in punctuation matters. `gamesOnShelf.length()` is an error.

Now, that distance idea from Part 2 pays off. Four games on the shelf, and the last one is three places along:

```js
const lastIndex = gamesOnShelf.length - 1;

console.log(gamesOnShelf[lastIndex]);
```

```
Snow Line
```

Or in one line, which is how you will usually see it written:

```js
console.log(gamesOnShelf[gamesOnShelf.length - 1]);
```

That looks knotted the first time you see it. Read it from the inside out: work out `gamesOnShelf.length - 1`, which is 3, and then ask the shelf for the game three places along. The advantage over just typing `gamesOnShelf[3]` is that it keeps working when the shelf gets longer or shorter.

### Adding and taking away

After game night, games get put back on the shelf. They go on top of the pile, and the next person takes the top one off. That is exactly how `push` and `pop` work.

```js
const gamesOnShelf = ['Fjord Traders', 'Hare and Hound'];

gamesOnShelf.push('Cabin Fever');

console.log(gamesOnShelf);
console.log(gamesOnShelf.length);
```

```
[ 'Fjord Traders', 'Hare and Hound', 'Cabin Fever' ]
3
```

`push` puts an item on the end. `pop` takes the last item off, and hands it to you:

```js
const gamesOnShelf = ['Fjord Traders', 'Hare and Hound', 'Cabin Fever'];

const borrowedGame = gamesOnShelf.pop();

console.log('Someone borrowed: ' + borrowedGame);
console.log(gamesOnShelf);
```

```
Someone borrowed: Cabin Fever
[ 'Fjord Traders', 'Hare and Hound' ]
```

Two things to notice.

`pop` changed the shelf itself. `gamesOnShelf` is one shorter than it was, permanently. It did not make a copy and leave the original alone.

And `pop` gave something back, which is why it could be stored in `borrowedGame`. `push` gives something back too, though less usefully: it hands you the new length.

<details>

<summary>Rabbit hole: why the end and not the front?</summary>

Because taking from the end is easy and taking from the front is not.

Every item in an array knows how far along it is. Take the last one off and nothing else has to move: item zero is still zero places along, item one is still one place along. Take the **first** one off and every remaining item has to shuffle down by one. On a shelf of four games nobody would notice. On an array of fifty thousand, the difference is real.

There are methods for working at the front of an array, and you will meet them. They just cost more, which is why `push` and `pop` are the pair you are given first.

If you want the picture: `push` and `pop` treat an array like a stack of plates. You add to the top, you take from the top, and the plate at the bottom stays where it is until everything above it has gone.

</details>

---

## Part 4: Objects are names

An array was the right shape for the shelf. It is the wrong shape for one game.

You could try it:

```js
const game = ['Fjord Traders', 3, 90, true];
```

That works, in the sense that nothing breaks. But now you have to remember that the playing time is two places along, and so does everybody else who reads your code, forever. There is nothing in `game[2]` to tell you it means minutes.

An object gives every value a name. You make one with curly braces.

```js
const game = {
  title: 'Fjord Traders',
  minPlayers: 3,
  minutes: 90,
  isComplete: true,
};
```

```js
console.log(game);
```

```
{ title: 'Fjord Traders', minPlayers: 3, minutes: 90, isComplete: true }
```

The pieces:

- The curly braces `{}` hold the whole thing.
- `title` is a **key**, the name of one fact.
- The colon separates a key from its value.
- `'Fjord Traders'` is the **value**.
- A comma goes after each pair.

A key and its value together are called a property. Values can be any type, which is why one object can hold a string, two numbers and a boolean without complaint.

Note the comma after the last property, before the closing brace. JavaScript allows it, and it is worth getting into the habit, because it means adding another property later is a one-line change rather than a two-line one.

There is no order here in any meaningful sense. Writing `minutes` before `title` would give you exactly the same object. That is the real difference from an array, and it follows from how you get things out.

---

## Part 5: Dot or brackets

There are two ways to read a value out of an object, and the choice between them confuses people because both work most of the time.

**Dot notation** is the everyday one.

```js
const game = {
  title: 'Fjord Traders',
  minPlayers: 3,
  minutes: 90,
  isComplete: true,
};

console.log(game.title);
console.log(game.minutes);
```

```
Fjord Traders
90
```

**Bracket notation** does the same thing with the key written as a string.

```js
console.log(game['title']);
console.log(game['minutes']);
```

```
Fjord Traders
90
```

Which raises the obvious question: why would anyone type the longer one?

**Dot notation is for a name you know while you are writing the code. Bracket notation is for a name you will not know until the code runs.**

That is the distinction. Everything else follows from it.

When you type `game.title`, the word `title` is part of your program. It cannot change. When you type `game[something]`, JavaScript works out what `something` is first, and then looks up whatever key that turns out to name.

Here is the difference doing something useful. Say the village hall has a little display that shows one fact about a game, and which fact it shows is set elsewhere:

```js
const game = {
  title: 'Fjord Traders',
  minPlayers: 3,
  minutes: 90,
  isComplete: true,
};

let factToShow = 'minutes';

console.log(game.factToShow);
console.log(game[factToShow]);
```

```
undefined
90
```

The first one went looking for a key literally called `factToShow`, found no such key, and gave you `undefined`. The second one worked out that `factToShow` holds the string `'minutes'`, then looked up `minutes`. Change `factToShow` to `'title'` and the second line changes what it prints without you touching it.

There is a second situation where brackets are not optional. Some keys are not valid to write after a dot, usually because they contain a space or a hyphen:

```js
const shelfNotes = {
  'last tidied': 'March',
  'donated-by': 'The Hansen family',
};

console.log(shelfNotes['last tidied']);
```

```
March
```

`shelfNotes.last tidied` is not something JavaScript can make sense of. It would try to read a key called `last`, then find a stray word after it, and give you a `SyntaxError`.

Keys like that are best avoided in data you create yourself. You will meet them soon enough in data that arrives from elsewhere.

### Adding, changing and removing

All three use the same notation you already know.

```js
const game = {
  title: 'Fjord Traders',
  minPlayers: 3,
};

game.minutes = 90;
game['isComplete'] = true;

console.log(game);
```

```
{ title: 'Fjord Traders', minPlayers: 3, minutes: 90, isComplete: true }
```

If the key does not exist, JavaScript makes it. If it does exist, the value is overwritten:

```js
game.minutes = 75;

console.log(game.minutes);
```

```
75
```

That means adding and updating look identical, which is convenient and occasionally dangerous. A typo in a key name does not produce an error. It quietly creates a new property and leaves the old one untouched:

```js
game.minuts = 120;

console.log(game.minutes);
console.log(game.minuts);
```

```
75
120
```

Two properties now, one of them a typo, and nothing anywhere said a word about it.

To remove a property completely, use `delete`:

```js
delete game.minuts;

console.log(game);
```

The key and its value both go.

---

## Part 6: What const actually protects

In the first extra lesson, we described a variable name as a tag tied onto a value, rather than a box you put a value into. Here is where that picture earns its keep.

This is an error, exactly as you would expect:

```js
const game = { title: 'Fjord Traders' };

game = { title: 'Snow Line' };
```

```
Uncaught TypeError: Assignment to constant variable.
```

But this is fine:

```js
const game = { title: 'Fjord Traders' };

game.title = 'Snow Line';
game.minutes = 90;

console.log(game);
```

```
{ title: 'Snow Line', minutes: 90 }
```

The `const` is unchanged and yet everything about the object changed. If you thought of `const` as a locked box that nothing can get into, that would make no sense at all.

With the tag picture it is straightforward. **`const` stops the tag being moved to a different object. It says nothing about what happens inside the object the tag is tied to.** The first example tried to move the tag, and was refused. The second left the tag where it was and rearranged the contents.

The same is true of arrays, since an array is a kind of object:

```js
const gamesOnShelf = ['Fjord Traders'];

gamesOnShelf.push('Snow Line');

console.log(gamesOnShelf);
```

```
[ 'Fjord Traders', 'Snow Line' ]
```

`const` and `push` sitting together looks wrong until you know what `const` was protecting.

In practice this means you should declare nearly all your arrays and objects with `const`. You almost never want to swap the whole thing for a different one, and the `const` protects you from doing it by accident while leaving you free to work on the contents.

<details>

<summary>Rabbit hole: two identical objects are not equal</summary>

This surprises everybody the first time.

```js
const gameA = { title: 'Fjord Traders' };
const gameB = { title: 'Fjord Traders' };

console.log(gameA === gameB);
```

```
false
```

Same key, same value, and yet `false`.

The tag picture explains it. `gameA` and `gameB` are two tags tied to two separate objects that happen to look alike. `===` on objects does not compare what is inside them. It asks whether the two tags are tied to the very same object.

Tie a second tag to the same one and you get the answer you expected:

```js
const gameA = { title: 'Fjord Traders' };
const gameC = gameA;

console.log(gameA === gameC);
```

```
true
```

And `gameC` is not a copy. There is one object with two tags on it, so changing it through either name changes the thing both names point at:

```js
gameC.title = 'Snow Line';

console.log(gameA.title);
```

```
Snow Line
```

That catches people out for months. For now, just hold on to the rule: with strings, numbers and booleans, `===` compares the values. With objects and arrays, it compares whether they are the same one.

</details>

---

## Part 7: The shape you will meet everywhere

Now put the two together, because the combination is not an advanced curiosity. It is what nearly all real data looks like.

The shelf is a list, so it is an array. Each game is a thing with named facts, so it is an object. An array of objects:

```js
const shelf = [
  { title: 'Fjord Traders', minPlayers: 3, minutes: 90, isComplete: true },
  { title: 'Hare and Hound', minPlayers: 2, minutes: 20, isComplete: true },
  { title: 'The Lighthouse Keeper', minPlayers: 2, minutes: 45, isComplete: false },
  { title: 'Snow Line', minPlayers: 4, minutes: 60, isComplete: true },
];
```

When you fetch data from the internet later in this course, this is very often exactly what arrives.

Getting at one value means doing the two steps you already know, one after the other. Work from the outside in.

```js
console.log(shelf[2]);
```

```
{ title: 'The Lighthouse Keeper', minPlayers: 2, minutes: 45, isComplete: false }
```

That gave you the whole object. Now read a fact off it:

```js
console.log(shelf[2].title);
```

```
The Lighthouse Keeper
```

`shelf[2].title` is not one operation. It is `shelf[2]`, which produces an object, followed by `.title` on that object. If a long chain like this ever confuses you, split it back into two lines with a variable in the middle, and look at what the middle value actually is.

```js
const thirdGame = shelf[2];

console.log(thirdGame.title);
```

Identical result, and far easier to debug.

### Walking the shelf

An array of objects plus a `for` loop is where this all starts to be worth something.

```js
for (let i = 0; i < shelf.length; i++) {
  console.log(shelf[i].title + ' takes about ' + shelf[i].minutes + ' minutes.');
}
```

```
Fjord Traders takes about 90 minutes.
Hare and Hound takes about 20 minutes.
The Lighthouse Keeper takes about 45 minutes.
Snow Line takes about 60 minutes.
```

Look at the condition: `i < shelf.length`. Not `i <= shelf.length`, and not a hard-coded `4`.

`shelf.length` is 4, and the last game is three places along, so the loop has to stop **before** it reaches 4. That is why it is `<` rather than `<=`. Get this wrong and the final pass asks for `shelf[4]`, which is `undefined`, and then asks `undefined` for its `.title`, which is a genuine error and will stop your program.

And using `.length` rather than typing `4` means the loop keeps working after somebody pushes another game onto the shelf.

Once inside the loop, pulling out the current game first often reads better:

```js
for (let i = 0; i < shelf.length; i++) {
  const game = shelf[i];

  console.log(game.title + ': ' + game.minPlayers + ' players or more.');
}
```

Same output as the equivalent version with `shelf[i]` everywhere, and one fewer thing to hold in your head on each line.

---

## Exercises

Try each one before opening the solution.

### Exercise 1: Choose the shape

No code for this one. For each of the six descriptions below, say whether you would reach for an array, an object, or an array of objects, and say why in one sentence.

1. The names of the six people who turned up to game night.
2. Everything the village hall knows about one game: its title, how many players, how long, whether the pieces are all there.
3. The whole shelf, with all of those details for every game.
4. The hall's opening time on each day of the week.
5. The final scores from one game of Fjord Traders, in the order the players finished.
6. One member's name, the year they joined, and whether they have paid this year's subscription.

<details>

<summary>Solution</summary>

1. **Array.** Six things of the same kind, and there may be a meaningful order, such as who arrived first. Nothing here needs a name.

2. **Object.** One thing, described by several named facts of different types. You would look these up by name, never by position.

3. **Array of objects.** A list, where each entry is itself a thing with named facts. This is the shape from Part 7.

4. **Object.** This one is worth a second look. The days have an obvious order, which makes an array tempting. But you will be asking "when does it open on Thursday", not "when does it open on the fourth day", so the useful handle is the name. Keys of `monday`, `tuesday` and so on.

5. **Array.** Same kind of thing, and the order is the whole point.

6. **Object.** One person, named facts. Same shape as number 2, different subject.

The test that decides most cases: **will you be looking things up by name, or counting through them?** If you catch yourself needing to remember that the playing time lives at position 2, you wanted an object.

</details>

### Exercise 2: Predict the index

Write down what each of these prints before you run any of it.

```js
const shelf = ['Fjord Traders', 'Hare and Hound', 'The Lighthouse Keeper', 'Snow Line'];
```

```js
shelf[1]
```

```js
shelf[shelf.length]
```

```js
shelf[shelf.length - 1]
```

```js
shelf.length
```

```js
shelf[0].length
```

Then a harder one. What is the value of `shelf` after these two lines, and what is its length?

```js
shelf[6] = 'Cabin Fever';
```

<details>

<summary>Solution</summary>

```
shelf[1]                  "Hare and Hound"    one place along
shelf[shelf.length]       undefined           length is 4, and there is nothing four places along
shelf[shelf.length - 1]   "Snow Line"         three places along, the last one
shelf.length              4
shelf[0].length           13                  the number of characters in "Fjord Traders"
```

The second one is the mistake this exercise exists for. `shelf.length` is 4, but there is no item at index 4, because the four items sit at 0, 1, 2 and 3. Writing `array[array.length]` instead of `array[array.length - 1]` is one of the most common off-by-one errors there is.

The last one catches people for a different reason. `.length` is not only an array thing. It works on strings too, where it counts characters instead of items. `shelf[0]` is the string `'Fjord Traders'`, so `.length` counts its 13 characters, including the space in the middle.

Both readings of `.length` mean the same thing underneath: how many pieces is this made of. An array is made of items and a string is made of characters. Strings have a good deal more you can ask of them, and Module 3 opens with exactly that, so this is a first look rather than the whole story.

**The harder one.** After `shelf[6] = 'Cabin Fever'`, the length is **7**.

```
[ 'Fjord Traders', 'Hare and Hound', 'The Lighthouse Keeper', 'Snow Line', <2 empty items>, 'Cabin Fever' ]
```

You put something six places along in an array that only had four items, so JavaScript stretched the array to fit and left holes at 4 and 5. Asking for `shelf[4]` gives `undefined`.

Nothing warned you. This is almost never what anyone wants, and it is what `push` exists to prevent: `push` always puts the item in the next real position, so you cannot leave a gap by accident.

</details>

### Exercise 3: The shelf as a stack

Work this one in the console, one line at a time, checking the shelf after each step.

1. Create an array called `shelf` holding three game titles.
2. Print how many games are on it.
3. Somebody returns a game. Push a fourth title onto the shelf.
4. Print the shelf and its length again.
5. Somebody borrows the game on top. Pop it off and store what you get back in a variable called `borrowed`.
6. Print a message saying which game was borrowed.
7. Print the last game now remaining on the shelf, without typing a number in the brackets.

<details>

<summary>Solution</summary>

```js
const shelf = ['Fjord Traders', 'Hare and Hound', 'The Lighthouse Keeper'];

console.log('Games on the shelf: ' + shelf.length);

shelf.push('Snow Line');

console.log(shelf);
console.log('Games on the shelf: ' + shelf.length);

const borrowed = shelf.pop();

console.log(borrowed + ' has been borrowed.');

console.log('Top of the shelf now: ' + shelf[shelf.length - 1]);
```

```
Games on the shelf: 3
[ 'Fjord Traders', 'Hare and Hound', 'The Lighthouse Keeper', 'Snow Line' ]
Games on the shelf: 4
Snow Line has been borrowed.
Top of the shelf now: The Lighthouse Keeper
```

Step 7 is the point of the exercise. `shelf[shelf.length - 1]` still gives the right answer after a push and a pop have changed the length twice. `shelf[2]` would have been correct at that moment purely by luck, and wrong the next time anybody touched the shelf.

Notice also that `shelf` is a `const` and every one of these lines worked. Nothing here moved the tag.

</details>

### Exercise 4: Build and modify a game

1. Create an object called `game` with a `title` and a `minPlayers`. Use `const`.
2. Print it.
3. Add a `minutes` property using dot notation.
4. Add an `isComplete` property using bracket notation.
5. A piece has turned up in the bottom of the box. Update `isComplete` to `true`.
6. Print the object again.
7. The hall has decided not to record playing times any more. Delete the `minutes` property.
8. Print the object one last time.

Then predict which of these three lines cause an error, before you try them:

```js
game.title = 'Snow Line';
```

```js
game = { title: 'Snow Line' };
```

```js
game.somethingNew = 'anything';
```

<details>

<summary>Solution</summary>

```js
const game = {
  title: 'The Lighthouse Keeper',
  minPlayers: 2,
};

console.log(game);

game.minutes = 45;
game['isComplete'] = false;

game.isComplete = true;

console.log(game);

delete game.minutes;

console.log(game);
```

```
{ title: 'The Lighthouse Keeper', minPlayers: 2 }
{ title: 'The Lighthouse Keeper', minPlayers: 2, minutes: 45, isComplete: true }
{ title: 'The Lighthouse Keeper', minPlayers: 2, isComplete: true }
```

Steps 4 and 5 do the same job by different routes, which is the point: adding a property and updating one are the same operation. Whether the key already existed is the only difference, and JavaScript does not tell you which happened.

**The three predictions.** Only the middle one errors.

```js
game.title = 'Snow Line';        // fine, changes the contents
game = { title: 'Snow Line' };   // TypeError: Assignment to constant variable
game.somethingNew = 'anything';  // fine, adds a property
```

The first and third reach inside the object, which `const` has nothing to say about. The second tries to move the tag to a different object, which is the one thing `const` forbids.

</details>

### Exercise 5: Dot or brackets

Here is an object with a couple of awkward keys.

```js
const shelfNotes = {
  location: 'Back room, left wall',
  'last tidied': 'March',
  'donated-by': 'The Hansen family',
};
```

1. Print the location using dot notation.
2. Print when the shelf was last tidied. Work out why dot notation cannot do this one.
3. Print who donated the shelf.
4. Create a variable called `wanted` holding the string `'location'`. Now print the location again, this time using `wanted`.
5. Explain in one sentence why `shelfNotes.wanted` does not work.

<details>

<summary>Solution</summary>

```js
const shelfNotes = {
  location: 'Back room, left wall',
  'last tidied': 'March',
  'donated-by': 'The Hansen family',
};

console.log(shelfNotes.location);

console.log(shelfNotes['last tidied']);

console.log(shelfNotes['donated-by']);

const wanted = 'location';

console.log(shelfNotes[wanted]);
```

```
Back room, left wall
March
The Hansen family
Back room, left wall
```

**Step 2.** `shelfNotes.last tidied` gives a `SyntaxError`. After a dot, JavaScript expects a single name with no spaces in it, so it reads `last` and then has no idea what to do with the word sitting after it.

**Step 3** has the same problem for a different reason, and a more confusing one. `shelfNotes.donated-by` is not a syntax error. JavaScript reads it as `shelfNotes.donated` minus a variable called `by`, and since no such variable exists you get:

```
Uncaught ReferenceError: by is not defined
```

An error message about a variable you never wrote, caused by a hyphen. If you ever see a `ReferenceError` naming something that looks like half of a key, a hyphen after a dot is the thing to look for.

**Step 5.** `shelfNotes.wanted` looks for a key whose name is literally `wanted`. Written after a dot, the word is part of your program, not a variable to be worked out. Bracket notation is the version that evaluates first and looks up second.

</details>

### Exercise 6: A report on the shelf

Use this shelf.

```js
const shelf = [
  { title: 'Fjord Traders', minPlayers: 3, minutes: 90, isComplete: true },
  { title: 'Hare and Hound', minPlayers: 2, minutes: 20, isComplete: true },
  { title: 'The Lighthouse Keeper', minPlayers: 2, minutes: 45, isComplete: false },
  { title: 'Snow Line', minPlayers: 4, minutes: 60, isComplete: true },
  { title: 'Cabin Fever', minPlayers: 3, minutes: 30, isComplete: false },
];
```

Write one `for` loop that walks the whole shelf and produces all of the following. Work out on paper what the answers should be before you run anything.

1. A numbered list of every game with its playing time, starting the numbering at 1.
2. A count of how many games are missing pieces, printed after the loop.
3. The title of the longest game, printed after the loop. You will need a variable outside the loop that remembers the best one found so far.

<details>

<summary>Solution</summary>

```js
let missingPieces = 0;
let longestTitle = '';
let longestMinutes = 0;

for (let i = 0; i < shelf.length; i++) {
  const game = shelf[i];

  console.log((i + 1) + '. ' + game.title + ' (' + game.minutes + ' minutes)');

  if (game.isComplete === false) {
    missingPieces++;
  }

  if (game.minutes > longestMinutes) {
    longestMinutes = game.minutes;
    longestTitle = game.title;
  }
}

console.log('Games missing pieces: ' + missingPieces);
console.log('Longest game: ' + longestTitle + ' at ' + longestMinutes + ' minutes.');
```

```
1. Fjord Traders (90 minutes)
2. Hare and Hound (20 minutes)
3. The Lighthouse Keeper (45 minutes)
4. Snow Line (60 minutes)
5. Cabin Fever (30 minutes)
Games missing pieces: 2
Longest game: Fjord Traders at 90 minutes.
```

Three things worth pulling out.

**The numbering.** `i` runs 0, 1, 2, 3, 4, and the list needs 1 to 5, so the printed number is `i + 1`. The index stayed a distance; only the display changed.

**The count.** `missingPieces` is declared **before** the loop and with `let`. Declare it inside and it would be created fresh on every pass and be back to zero every time. This pattern, a variable outside a loop that accumulates as the loop runs, turns up constantly.

**The longest game.** Same idea, slightly cleverer. `longestMinutes` starts at 0, and every game that beats the current best replaces it. By the end of the loop it holds the biggest value seen. Starting at 0 works here because no game takes fewer than zero minutes.

You could also write `if (!game.isComplete)` in place of `if (game.isComplete === false)`. Both are correct. The `!` version reads better once you are used to it, and it gets a proper lesson shortly.

</details>

---

## Where this leaves you

An array is for things you count through, and an object is for one thing you look things up on. Choosing between them is a question about how you will use the data, not about what the data is, and it is worth deciding deliberately rather than reaching for whichever you met most recently.

An index is a distance from the front, not a count. That is why the first item is at 0, why the last is at `length - 1`, and why a loop over an array stops at `<` rather than `<=`.

Dot notation is for a name you know as you write. Bracket notation is for a name your program will not know until it runs.

`const` on an object or an array protects the name, not the contents, which is exactly what the tag picture predicts and exactly why `const` is the right default for both.

And an array of objects is not an advanced topic. It is the ordinary shape of real data, and you will be looking at it for the rest of the course.

Next comes the thing that lets you give a name to a piece of work rather than a piece of data.
