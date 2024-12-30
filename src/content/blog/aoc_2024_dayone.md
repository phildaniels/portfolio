---
title: 'Advent of Code: Day 1'
description: 'Day one of advent of code and project setup'
pubDate: 'Dec 29 2024'
heroImage: '/advent_of_code_2024.webp'
tags: ['aoc', '2024', 'typescript', 'bun']
---

# Advent of Code 2024 Day 1

## Updating my astro site

First thing I did was get astro updated an up and running (which runs my personal website). It took a bit, but following their public docs and some googling did the trick. Lucky for me I was on Astro 4, and upgrading to 5 was pretty seamless

## Project Setup

### Installing Bun

I'm deciding to use bun, and TypeScript this year, so learning something new and something familiar. I may go back and do this again in something like Go or Rust, but as a father with limited free time, starting with TypeScript.

First thing is first, installing Bun. Going to their public docs, it's as easy as (on my Ubuntu 24.04 WSL2 Instance)

```bash
curl -fsSL https://bun.sh/install | bash
```

Well, not quite, had to install "unzip" first (give me a break, I work for Microsoft, this WSL instance has barely been touched, I live in Powershell and Windows)

### Setting up the project

I create a new folder in my advent of code github repo, and I bet dollars to donuts the init command is

```bash
bun init
```

Which works great. It creates a package.json, an index.ts, a README and a few others.

I'm dying to get going on this, so instead of thinking about project architecture up front, I figure I'll refine it as I go. So, we'll do something simple to start. For directory structure, something like:

```
index.ts // runs all the days, maybe caches the answers
utils.ts // I wrote a bunch of duplicative code last time, let's actually do a little DRY
dayone
   index.ts
   data.txt  // that way the import is always the same
daytwo
   index.ts
   data.txt
...
dayn
  ...
```

Easy enough, I'll add that to utils later, so let's with dayone/index.ts start reading the problem

### The Problem

Being real, I skimmed the narrative, and got to work.

Reading the problem, seems simple enough, there are two columns of text that need matched up from smallest to largest, then the sum of their difference. Going to assume the lists will be the same size, so I first come up with my solution to parsing the data.

### The unknown (parsing text)

So the first thing I need is something that can parse text, I see if bun has built in file system APIs (asking copilot)

Phil: Does bun have built in file system APIs?

Copilot: Yes, Bun has built-in file system APIs. You can use the fs module in Bun to interact with the file system. Here is an example of how to use it:

```typescript
import { readFile, writeFile } from 'bun:fs';

// Reading a file
const data = await readFile('example.txt', 'utf8');
console.log(data);

// Writing to a file
await writeFile('example.txt', 'Hello, Bun!');
```

Okay not too bad, but pasting that in, I get an error

`Cannot find module 'bun:fs' or its corresponding type declarations.ts(2307)`

Maybe this is like node? I try a

```bash
bun install @types/bun
```

no luck...

Turns out copilot hallucinated. Here is from the bun docs on how to read in a file

```typescript
const foo = Bun.file("foo.txt");
await foo.text(); // contents as a string
await foo.stream(); // contents as ReadableStream
await foo.arrayBuffer(); // contents as ArrayBuffer
await foo.bytes(); // contents as Uint8Array
```

In this case, I can't seem to trust copilot, maybe the bun api used to use "bun:fs", not sure, but it's not right now. I'll lean towards the docs in the future

Either way, so constructing this

```typescript
const data = Bun.file("data.txt");
const text = await data.text();
console.log(text);
```

I get, for their example input

```
$ bun index.ts
3   4
4   3
2   5
1   3
3   9
3   3
```

Awesome, the problem is comparatively cake now, I need to parse out the two columns, sort them, and sum their difference.

### Part One

I'll split the text on the newline character, then split the line on the separator (three spaces), convert each side into an integer

```typescript
const lines = text.split("\n");
for (const line of lines) {
  const [leftNumber, rightNumber] = line
    .split("   ")
    .map((numberString) => +numberString) as [number, number];
  left.push(leftNumber);
  right.push(rightNumber);
}
```

Next, I need to sort the arrays, I'll do so in place

```typescript
left.sort();
right.sort();
```

Then, finally, summing their differences. I could do this a variety of ways, but a plain old for loop makes the most sense to my brain

```typescript
let sum = 0;
for (let i = 0; i < left.length; i++) {
    const leftNumber = left[i];
    const rightNumber = right[i];
    const difference = Math.abs(leftNumber - rightNumber);
    sum += difference;
}

console.log(sum);
```

Making the final solution

```typescript
const data = Bun.file("data.txt");
const text = await data.text();

const left: number[] = [];
const right: number[] = [];

const lines = text.split("\n");
for (const line of lines) {
  const [leftNumber, rightNumber] = line
    .split("   ")
    .map((numberString) => +numberString) as [number, number];
  left.push(leftNumber);
  right.push(rightNumber);
}

left.sort();
right.sort();

let sum = 0;
for (let i = 0; i < left.length; i++) {
    const leftNumber = left[i];
    const rightNumber = right[i];
    const difference = Math.abs(leftNumber - rightNumber);
    sum += difference;
}

console.log(sum);
```

Okay, running it, I get

```
$ bun index.ts
11
```

Okay, that's the same as the reported answer, now to try on my puzzle input, crossing my fingers for no corner cases or gotcha's I didn't think of

```
$ bun index.ts
765748
```

My answer happened to be a little less than a million, 765748, and that was correct!

On to part two, let's see if my solution can be extended

### Part Two

Okay, so reading through the fluff, I need to figure out the frequency that numbers in the left list appear in the right list, multiply the frequency by the number, then sum that. I can reuse some of what I had so far, cleaning it up a bit, I separate them out into part one and part two functions, as well as a function that gets me the lines and text.

```typescript
const getProblemText = async () => {
  const data = Bun.file("data.txt");
  const text = await data.text();
  return text;
};

const getProblemLines = (text: string) => text.split("\n");

const partOne = (lines: string[]) => {
  const left: number[] = [];
  const right: number[] = [];
  for (const line of lines) {
    const [leftNumber, rightNumber] = line
      .split("   ")
      .map((numberString) => +numberString) as [number, number];
    left.push(leftNumber);
    right.push(rightNumber);
  }
  left.sort();
  right.sort();
  let sum = 0;
  for (let i = 0; i < left.length; i++) {
    const leftNumber = left[i];
    const rightNumber = right[i];
    const difference = Math.abs(leftNumber - rightNumber);
    sum += difference;
  }

  return sum;
};

const partTwo = (lines: string[]) => {
  // implement me!
  return 0;
};

const text = await getProblemText();
const lines = getProblemLines(text);

console.log("part one:", partOne(lines));
console.log("part two:", partTwo(lines));
```

Working on part two, I could start with two lists, and manually iterate through to get a count, but I'd like to do some pre-work and build up a map of frequencies in advance. Something like:

```typescript
const rightNumberToFrequencyLookup = new Map<number, number>();
const left: number[] = [];
for (const line of lines) {
   const [leftNumber, rightNumber] = line
   .split("   ")
   .map((numberString) => +numberString) as [number, number];
   left.push(leftNumber);

   if (!rightNumberToFrequencyLookup.has(rightNumber)) {
      rightNumberToFrequencyLookup.set(rightNumber, 1);
   } else {
      const current = rightNumberToFrequencyLookup.get(rightNumber)!;
      rightNumberToFrequencyLookup.set(rightNumber, current + 1);
   }
}
```

Now it's as simple as taking the numbers in the left, iterating one by one, looking them up in the map, and performing the calculation. Something like

```typescript
let sum = 0;
for (const item of left) {
   const frequencyInRight = rightNumberToFrequencyLookup.get(item) ?? 0;
   sum += frequencyInRight * item;
}

return sum;
```

Making my whole part two solution:

```typescript
const partTwo = (lines: string[]) => {
  const rightNumberToFrequencyLookup = new Map<number, number>();
  const left: number[] = [];
  for (const line of lines) {
    const [leftNumber, rightNumber] = line
      .split("   ")
      .map((numberString) => +numberString) as [number, number];
    left.push(leftNumber);

    if (!rightNumberToFrequencyLookup.has(rightNumber)) {
      rightNumberToFrequencyLookup.set(rightNumber, 1);
    } else {
      const current = rightNumberToFrequencyLookup.get(rightNumber)!;
      rightNumberToFrequencyLookup.set(rightNumber, current + 1);
    }
  }

  let sum = 0;
  for (const item of left) {
    const frequencyInRight = rightNumberToFrequencyLookup.get(item) ?? 0;
    sum += frequencyInRight * item;
  }

  return sum;
};
```

Running it on the example, I get 31, which is the expected sum

```b $ bun index.ts
part one: 11
part two: 31
```

Trying on my problem input, I get 27732508, which is correct

Done and dusted 🙌 😎
