---
title: 'Advent of Code 2024: Day 2'
description: 'Day two of advent of code'
pubDate: 'Jan 1 2025'
heroImage: '/advent_of_code_2024.webp'
tags: ['aoc', '2024', 'typescript', 'bun']
---

# Advent of Code 2024 Day 2

## Modularizing the days

Last time ([Advent of Code 2024: Day 1](/blog/advent-of-code-2024-day-1)), I talked about how I planned to have an orchestration layer that printed all the days. Nothing is intensive enough to cache answers (yet), but here's a crack at that modularization

```typescript
import dayOne from "./dayone";
import dayTwo from "./daytwo";

const problems = [
  {
    partOne: dayOne.partOne,
    partTwo: dayOne.partTwo,
  },
  {
    partOne: dayTwo.partOne,
    partTwo: dayTwo.partTwo,
  },
];

for (const [index, problem] of problems.entries()) {
  console.log(`Day ${index + 1}`);
  console.log(`Part One: ${problem.partOne()}`);
  console.log(`Part Two: ${problem.partTwo()}`);
  console.log();
}
```

Where my exports can be curried if they need arguments, like so:

(day one)

```typescript
export default { partOne: () => partOne(lines), partTwo: () => partTwo(lines) };
```

Now I can just modify this and add files as necessary, might refactor this later but this works fine for now.

## The problem

This one is simple, I need to read in each line, and perform some validation logic on the line. Let's start by getting the lines, and while I'm at it, move the line logic to a utils file. Moving this into utils.ts, and adding a new function (accounting for the current working directory)

```typescript
// in utils.ts
export const getProblemText = async (cwd: string) => {
  const data = Bun.file(`${cwd}/data.txt`);
  const text = await data.text();
  return text;
};

export const getProblemLines = (text: string) => text.split("\n");

export const getProblemLinesFromText = async (cwd: string) => {
  const text = await getProblemText(cwd);
  const lines = getProblemLines(text);
  return lines;
};
```

So here's the lines skeleton of the solution, part one and two

```typescript
import { getProblemLinesFromText } from "../utils";

const partOne = (lines: string[]) => {
  return 0;
};

const partTwo = (lines: string[]) => {
  return 0;
};

const lines = await getProblemLinesFromText(__dirname);

export default { partOne: () => partOne(lines), partTwo: () => partTwo(lines) };
```

## Part 1

Now, splitting on the separator, mapping to convert the string to an int, we get

```typescript
for (const line of lines) {
    const numbers = line.split(" ").map((numberString) => +numberString);
}
```

Now, I need to determine if the numbers are increasing or decreasing (IE, from the problem, the first condition "The levels are either all increasing or all decreasing"). I can do a little trick and do a for loop starting at index 1, going until the second to last element. That way, at any given time, I'm looking at the "current", "previous", and "next" number in the list.

```typescript
for (const line of lines) {
    const numbers = line.split(" ").map((numberString) => +numberString);
    let lineWasValid = true;
    for (let i = 1; i < numbers.length - 1; i++) {
      const previousNumber = numbers[i - 1];
      const currentNumber = numbers[i];
      const nextNumber = numbers[i + 1];
    }
}
```

Now, if the numbers are decreasing, the "previous" number will be less than the "current" number, and the "current" number will be less than the "next" number. You can apply similar logic for decreasing, IE just swap the comparison. Something like

```typescript
for (let i = 1; i < numbers.length - 1; i++) {
  const previousNumber = numbers[i - 1];
  const currentNumber = numbers[i];
  const nextNumber = numbers[i + 1];
  if (
    !(
      (previousNumber < currentNumber && currentNumber < nextNumber) ||
      (previousNumber > currentNumber && currentNumber > nextNumber)
    )
  ) {
    // sequence was not decreasing or increasing
  }
}
```

I basically want to, as soon as I find a non-increasing / non-decreasing sequence, break out of the loop; that line is "invalid". But because the loop could exit organically, I need to keep track of if the rule was violated, I can do this with a boolean. I also need to introduce a counter for the valid line count. There's probably some simpler logical solution I'm not seeing, but this makes sense in my head. So the two loops become:

```typescript
let validLines = 0;
for (const line of lines) {
  const numbers = line.split(" ").map((numberString) => +numberString);
  let lineWasValid = true;
  for (let i = 1; i < numbers.length - 1; i++) {
    const previousNumber = numbers[i - 1];
    const currentNumber = numbers[i];
    const nextNumber = numbers[i + 1];
    if (
      !(
        (previousNumber < currentNumber && currentNumber < nextNumber) ||
        (previousNumber > currentNumber && currentNumber > nextNumber)
      )
    ) {
      lineWasValid = false;
      break;
    }
  }

  if (lineWasValid) {
    validLines++;
  }
}
```

Almost done, we have the second condition now, "Any two adjacent levels differ by at least one and at most three.". What's nice, is the first part of that, "differ by at least one", is covered by my previous logic. Maybe you could have interpreted those "<" symbols to be "<=", but either way my previous code covers this. So now, just need to check if the difference between the "current" number and the two adjacent numbers ("previous" and "next") are greater than 3, and if so, mark that line as invalid, break out of the loop. So, final solution for the problem:

```typescript
const partOne = (lines: string[]) => {
  let validLines = 0;
  for (const line of lines) {
    const numbers = line.split(" ").map((numberString) => +numberString);
    let lineWasValid = true;
    for (let i = 1; i < numbers.length - 1; i++) {
      const previousNumber = numbers[i - 1];
      const currentNumber = numbers[i];
      const nextNumber = numbers[i + 1];
      if (
        !(
          (previousNumber < currentNumber && currentNumber < nextNumber) ||
          (previousNumber > currentNumber && currentNumber > nextNumber)
        )
      ) {
        lineWasValid = false;
        break;
      }

      if (
        Math.abs(currentNumber - previousNumber) > 3 ||
        Math.abs(currentNumber - nextNumber) > 3
      ) {
        lineWasValid = false;
        break;
      }
    }

    if (lineWasValid) {
      validLines++;
    }
  }

  return validLines;
};
```

Which works on the example input and generates two. Grabbing my puzzle input, I get 379 valid lines, which is correct! On to part 2

## Part 2

So reading over this, the problem is identical, but we can remove any particular number from the sequence then run the validation. If it passes with any particular number removed, it passes.

We can reuse much of our existing code, but now we need to try variations of the line. Just straight up brute forcing it, we can just, for every index in the array, create a copy of the array without that index, then run the validation. This may not be performant enough for the puzzle input, but it's worth a shot.

The first thing to make this easier, is to abstract out the validation logic. Something like (rewriting part one):

```typescript
const arrayIsValid = (numbers: number[]) => {
  for (let i = 1; i < numbers.length - 1; i++) {
    const previousNumber = numbers[i - 1];
    const currentNumber = numbers[i];
    const nextNumber = numbers[i + 1];
    if (
      !(
        (previousNumber < currentNumber && currentNumber < nextNumber) ||
        (previousNumber > currentNumber && currentNumber > nextNumber)
      )
    ) {
      return false;
    }

    if (
      Math.abs(currentNumber - previousNumber) > 3 ||
      Math.abs(currentNumber - nextNumber) > 3
    ) {
      return false;
    }
  }

  return true;
};

const partOne = (lines: string[]) => {
  let validLines = 0;
  for (const line of lines) {
    const numbers = line.split(" ").map((numberString) => +numberString);
    const lineIsValid = arrayIsValid(numbers);
    if (lineIsValid) {
      validLines++;
    }
  }

  return validLines;
};
```

Okay, so part two, I need to check every "version" of the array with 0 or more elements missing. I can write a helper function that will generate all permutations of the array. Then, splice out each index. Something like:

```typescript
const generateAllPermutationsOfArrayWithZeroOrOneElementRemoved = (numbers: number[]) => {
  const permutations = [numbers];
  for (let i = 0; i < numbers.length; i++) {
    const copy = [...numbers];
    copy.splice(i, 1);
    permutations.push(copy);
  }

  return permutations;
}
```

Then, for every permutation, exit as soon as a passing permutation is found.

```typescript
const partTwo = (lines: string[]) => {
  let validLines = 0;
  for (const line of lines) {
    const numbers = line.split(" ").map((numberString) => +numberString);
    const permutations =
      generateAllPermutationsOfArrayWithZeroOrOneElementRemoved(numbers);
    for (const permutation of permutations) {
      const lineIsValid = arrayIsValid(permutation);
      if (lineIsValid) {
        validLines++;
        break;
      }
    }
  }

  return validLines;
};
```

Running this, we get the correct answer on their example text, and on my problem input I get 430, which is correct. But, wanted to tangent out here...

## Performance

The memory and performance footprint of my solution is definitely sub-optimal. Bun ran it nearly instantly, and on the size of input I had, it was fine. But how could be do better?

Chicken scratching out the complexity, it's something like (see comments, where N is the number of lines, and M is the number of numbers on a line):

```typescript
const partTwo = (lines: string[]) => {
  let validLines = 0;
  for (const line of lines) { // N
    const numbers = line
      .split(" ") // Something like 2M - 1, it needs to iterate over each character in the line, and if the numbers are large this would just increase this by a constant
      .map((numberString) => +numberString); // 2 M
    const permutations =
      generateAllPermutationsOfArrayWithZeroOrOneElementRemoved(numbers); // this is 2M^2, runs n times, in the outer loop, then makes a copy (M), then the splice runs about M times
    for (const permutation of permutations) { // there are M + 1 permutations, will run between 1 and  M + 1 times
      const lineIsValid = arrayIsValid(permutation); // this runs between 1 - M-1 times
      if (lineIsValid) {
        validLines++;
        break;
      }
    }
  }

  return validLines;
};
```

So taking that above analysis, in the worst and best case scenario we get something like

```
Worst Case: (N)((2M - 1) + (2M) + (2M^2))(M+1)(M-1) -> ~(N)(2M^2)(M)(M) -> ~N(M^4)
Best Case: (N)((2M - 1) + (2M) + (2M^2))(1)(1) -> ~(N)(2M^2) -> ~N(M^2)
```

Optimizing out some of those extra loops, and inlining the helper, we can do:

```typescript
const partTwoMorePerformant = (lines: string[]) => {
  let validLines = 0;
  for (const line of lines) {
    const numbers = line.split(" ").map((numberString) => +numberString);
    const lineIsValid = arrayIsValid(numbers);
    if (lineIsValid) {
      validLines++;
      continue;
    }

    for (let i = 0; i < numbers.length; i++) {
      const copy = [...numbers];
      copy.splice(i, 1);
      const permutationIsValid = arrayIsValid(copy);
      if (permutationIsValid) {
        validLines++;
        break;
      }
    }
  }

  return validLines;
};
```

This is better, but still has to create a copy of the array for every permutation (until a valid one is found). We improve out best case a little, but out worst case is just as bad. Again, the killer thing on the complexity is creating M copies of the array.

I don't think there's a way to generically check the permutations without having to increase the time complexity, we are shaving off polynomial exponent cardinality and constants in front of the complexity if we optimize further.

I suppose, we could be super clever and try to do the permutation logic in place, I even gave it a crack for a good 20 minutes but didn't come to a quick answer. All for a slightly better solution that already finishes in milliseconds (for me). So, for now, I think we'll call this "good enough" for Advent of Code.

Next on deck, day 3!
