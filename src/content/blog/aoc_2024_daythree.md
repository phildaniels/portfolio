---
title: 'Advent of Code 2024: Day 3'
description: 'Day three of advent of code'
pubDate: 'Jan 3 2025'
heroImage: '/advent_of_code_2024.webp'
tags: ['aoc', '2024', 'typescript', 'bun']
---

# Advent of Code 2024 Day 3

## The Problem

Last time ([Advent of Code 2024: Day 2](./advent-of-code-2024-day-2)), we had to write a validator for some number sequences. Today we have to strip valid text out of invalid text.

## Part One

This is cut and dry regex parsing of lines. I could get clever and write a parser for this, but let's try regex to start. I need to extract all the cases where the text is mul(digit, digit). I'm not great at regex, but I will do my best.

Let's start with a regex tester, like [Regex 101](https://regex101.com/).

Inserting the example text from the problem in the test string, "xmul(2,4)%&mul[3,7]!@^do_not_mul(5,5)+mul(32,64]then(mul(11,8)mul(8,5))" I need to successfully strip out all the valid multiplications. I sometimes write a little regex for vscode regex search, so my first intuition is

`mul\((\d+),(\d+)\)`

But what is that? Well, it's "mul", then "(" (which we escape), followed by any number of digits, a ",", any more numbers of digits, ending in a ")"

This matches all the cases in the example text, and ignores cases like like "mul(4\*", "mul(6,9!", "?(12,34)", or "mul ( 2 , 4 )".

I use regex so little in my day to day, I ask copilot how to run regex on a string and get back all the digits as matches.

I come up with something like this for getting the matches out

```typescript
const partOne = (text: string) => {
  console.log(text);
  const regex = /mul\((\d+),(\d+)\)/g;
  const matches = [...text.matchAll(regex)];

  for (const match of matches) {
    console.log(match);
  }

  return 0;
};
```

The text output of this is

```
xmul(2,4)%&mul[3,7]!@^do_not_mul(5,5)+mul(32,64]then(mul(11,8)mul(8,5))
[ "mul(2,4)", "2", "4", index: 1, input: "xmul(2,4)%&mul[3,7]!@^do_not_mul(5,5)+mul(32,64]then(mul(11,8)mul(8,5))",
  groups: undefined ]
[ "mul(5,5)", "5", "5", index: 29, input: "xmul(2,4)%&mul[3,7]!@^do_not_mul(5,5)+mul(32,64]then(mul(11,8)mul(8,5))",
  groups: undefined ]
[ "mul(11,8)", "11", "8", index: 53, input: "xmul(2,4)%&mul[3,7]!@^do_not_mul(5,5)+mul(32,64]then(mul(11,8)mul(8,5))",
  groups: undefined ]
[ "mul(8,5)", "8", "5", index: 62, input: "xmul(2,4)%&mul[3,7]!@^do_not_mul(5,5)+mul(32,64]then(mul(11,8)mul(8,5))",
  groups: undefined ]
```

So the digits are neatly placed into the second and third spot in the array as strings. We can now one liner them into our result with reduce, something like

```typescript
const partOne = (text: string) => {
  const regex = /mul\((\d+),(\d+)\)/g;
  const matches = [...text.matchAll(regex)];
  return matches.reduce(
    (previous, [_, firstString, secondString]) =>
      previous + +firstString * +secondString,
    0
  );
};
```

So what's that reduce doing? it takes all the matches, which is an array of arrays with, in the inner array, the string that was matched at index 0, the first number inside the parentheses in index 1, and the second number inside the parentheses (after the comma) in index 2, followed by more stuff we don't care about for this problem

Reduce takes a few arguments, the first is a function to run against every item in the array, with the first argument being the result of the previous calculation, the second argument being that inner array, and the third argument being the default calculation for the first run.

I use javascript object decomposition to assign the first element nothing, the second element to a variable called "first string", and the second element to a variable called "second string". Inline, I use the "+" sign to turn them into numbers from strings, multiple them together, then add them to the output of the previous run of the function.

IE I am, in one shot, extracting out the two digits, converting them to integers, multiplying them, and adding them all up. A more readable version of what I'm doing is

```typescript
let sum = 0;
for (const element of matches) {
    const firstString = element[1];
    const secondString = element[2];
    const first = +firstString;
    const second = +secondString;
    const product = first * second;
    sum = sum + product;
}
```

But one-liner-ing it feels pretty good.

Testing my solution, I get 161 for the example text, and 168539636 for my puzzle input, both of which are correct. Bun runs this instantly along with every previous solution, every time, which is pretty crazy. I do have a thread ripper, but regex parsing is usually pretty slow. I suppose Zig is a pretty amazing language to build your runtime on.

## Part Two

Without reading part two, the elf messed up and forgot there was some other thing that I need to do, and I need to redo the whole problem one to cover this other case.

Reading it through, I was close, I need to handle "do's" and "don'ts"

I would have been better off writing a parser from the beginning, it would have been easier to extend. But we're stuck at regex, so let's try and extend that.

My first instinct is just to like "or" the regex, where it matches the original input, OR "do()", OR "don't()". Like:

`do\(\)|don\'t\(\)|mul\((\d+),(\d+)\)`

Exploring a little, against text "xmul(2,4)&mul[3,7]!^don't()\_mul(5,5)+mul(32,64](mul(11,8)undo()?mul(8,5))" on Regex 101, my new regex matches all the terms as expected. I run this code:

```typescript
const partTwo = (text: string) => {
  const regex = /do\(\)|don\'t\(\)|mul\((\d+),(\d+)\)/g;
  const matches = [...text.matchAll(regex)];
  matches.forEach(([match, firstNumber, secondNumber]) =>
    console.log(match, firstNumber, secondNumber)
  );
  return 0;
};
```

And get this result

```
Part One: 161
mul(2,4) 2 4
don't() undefined undefined
mul(5,5) 5 5
mul(11,8) 11 8
do() undefined undefined
mul(8,5) 8 5
Part Two: 0
```

So now, going through the matches one by one, I can keep track of if we're "enabled" (IE should process the instruction) or not (should not process the instruction); If I encounter a "do()", set enabled to true and continue. If I encounter a "don't()", set enabled to false and continue. Otherwise, add the match to the running list of valid matches if we're enabled. Like so:

```typescript
const validMatches: RegExpExecArray[] = [];
for (const match of matches) {
  const [matchedString] = match;
  if (matchedString === "do()") {
    enabled = true;
    continue;
  }

  if (matchedString === `don't()`) {
    enabled = false;
    continue;
  }

  if (enabled) {
    validMatches.push(match);
  }
}
validMatches.forEach(([match, firstNumber, secondNumber]) =>
  console.log(match, firstNumber, secondNumber)
);
```

The output of which for their example is:

```
mul(2,4) 2 4
mul(8,5) 8 5
```

Which is the correct ones to perform the calculation on. So now, I abstract out my reduce function from part one:

```typescript
const multiplyAndSumMatches = (matches: RegExpExecArray[]) =>
  matches.reduce(
    (previous, [, firstString, secondString]) =>
      previous + +firstString * +secondString,
    0
  );
```

Making finally part one and part two:

```typescript
const partOne = (text: string) => {
  const regex = /mul\((\d+),(\d+)\)/g;
  const matches = [...text.matchAll(regex)];
  return multiplyAndSumMatches(matches);
};

const partTwo = (text: string) => {
  const regex = /do\(\)|don\'t\(\)|mul\((\d+),(\d+)\)/g;
  const matches = [...text.matchAll(regex)];
  let enabled = true;
  const validMatches: RegExpExecArray[] = [];
  for (const match of matches) {
    const [matchedString] = match;
    if (matchedString === "do()") {
      enabled = true;
      continue;
    }

    if (matchedString === `don't()`) {
      enabled = false;
      continue;
    }

    if (enabled) {
      validMatches.push(match);
    }
  }

  return multiplyAndSumMatches(validMatches);
};
```

Looking back at this, the only thing I could have realistically done better is inline the calculation while I went, but I'm alright doing a little extra looping, Bun is just crazy fast, I don't need to worry about micro-optimizations

Running this against the example, I get 48 for the example text, and for the puzzle input I get 97529391, which is correct! 3 days down! 😎 🙌 🚀
