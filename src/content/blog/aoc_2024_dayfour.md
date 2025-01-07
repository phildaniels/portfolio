---
title: 'Advent of Code 2024: Day 4'
description: 'Day three of advent of code'
pubDate: 'Jan 6 2025'
heroImage: '/advent_of_code_2024.webp'
tags: ['aoc', '2024', 'typescript', 'bun']
---

# Advent of Code 2024 Day 4

## The Problem

Last time ([Advent of Code 2024: Day 3](/blog/advent-of-code-2024-day-3)), we had to write an instruction simulator with regex. Today, we need to find all occurrences of the word "xmas" in a word search like fashion

## Part One

My intuition here is to iterate through every letter in the word search, then check every direction for xmas. First, I'll need the lines pushed into an array, so I can easily check via 2D array indexing. So getting the lines, I get the following for my skeleton of the problem

```typescript
import { getProblemLinesFromText } from "../utils";

const partOne = (lines: string[]) => {
  return 0;
};

const partTwo = (lines: string[]) => {
  return 0;
};

const lines = await getProblemLinesFromText(__dirname);

export default {
  partOne: () => partOne(lines),
  partTwo: () => partTwo(lines),
};

```

Now, I believe I'll need the "x" position and "y" position of the character so I can check every direction for the word xmas, looking only for xs. I'll use a traditional for loop

```typescript
let xmasCount = 0;
for (let xPosition = 0; xPosition < lines.length; xPosition++) {
  for (let yPosition = 0; yPosition < line[xPosition].length; yPosition++) {
    const char = lines[xPosition][yPosition];
    if (char !== 'X') {
      continue;
    }
  }
}
```

Now I need a helper that checks all four directions. We could do something like:

```typescript
const checkAllEighthDirectionsForXmas = (
  xPosition: number,
  yPosition: number,
  lines: string[]
) => {
  let countOfXmasAtPosition = 0;
  // right
  if (
    lines[xPosition + 1]?.[yPosition] === "M" &&
    lines[xPosition + 1]?.[yPosition] === "A" &&
    lines[xPosition + 1]?.[yPosition] === "S"
  ) {
    countOfXmasAtPosition++;
  }

  // left
  if (
    lines[xPosition]?.[yPosition] === "M" &&
    lines[xPosition]?.[yPosition] === "A" &&
    lines[xPosition]?.[yPosition] === "S"
  ) {
    countOfXmasAtPosition++;
  }

  // up
  if (
    lines[xPosition]?.[yPosition - 1] === "M" &&
    lines[xPosition]?.[yPosition - 1] === "A" &&
    lines[xPosition]?.[yPosition - 1] === "S"
  ) {
    countOfXmasAtPosition++;
  }

  // down
  if (
    lines[xPosition]?.[yPosition + 1] === "M" &&
    lines[xPosition]?.[yPosition + 1] === "A" &&
    lines[xPosition]?.[yPosition + 1] === "S"
  ) {
    countOfXmasAtPosition++;
  }

  // diagonal up, right
  if (
    lines[xPosition + 1]?.[yPosition - 1] === "M" &&
    lines[xPosition + 1]?.[yPosition - 1] === "A" &&
    lines[xPosition + 1]?.[yPosition - 1] === "S"
  ) {
    countOfXmasAtPosition++;
  }

  // ...
  return countOfXmasAtPosition;
```

It's readable, but a little lengthy and verbose. If there was an xDelta, yDelta, we could check all four directions more concisely. By this I mean:

```typescript
const checkAllFourDirectionsForXmas = (
  xPosition: number,
  yPosition: number,
  lines: string[]
) => {
  const xDeltas = [0, -1, 1];
  const yDeltas = [0, -1, 1];
  let countOfXmasAtPosition = 0;
  for (const xDelta of xDeltas) {
    for (const yDelta of yDeltas) {
      if (lines[xPosition + xDelta]?.[yPosition  + yDelta] === "M"
        && lines[xPosition + (xDelta * 2)]?.[yPosition  + (yDelta * 2)] === "A"
        && lines[xPosition + (xDelta * 3)]?.[yPosition  + (yDelta * 3)] === "S"
      ) {
        countOfXmasAtPosition++;
      }
    }
  }

  return countOfXmasAtPosition;
};
```

Something like that should simulate the more verbose logic at the cost of readability, which is a desirable tradeoff in this case I think. I recognize it loops one two many times comparing the position to itself (xDelta = 0, yDelta = 0), but otherwise it's good enough for a program that needs to give an output once.

Then, finally:

```typescript
const partOne = (lines: string[]) => {
  let xmasCount = 0;
  for (let xPosition = 0; xPosition < lines.length; xPosition++) {
    const line = lines[xPosition];
    for (let yPosition = 0; yPosition < line.length; yPosition++) {
      const char = line[yPosition];
      if (char !== "X") {
        continue;
      }

      xmasCount += checkAllEightDirectionsForXmas(xPosition, yPosition, lines);
    }
  }

  return xmasCount;
};
```

On the example input, I get the expected 18. On my puzzle input, I get 2521, which is correct

## Part Two

I thought about this for a bit. I think the most clever thing I could come up with is find all the instances of "MAS", find all the positions of the A's, and if that position is found more than once, we must have an "X". Modifying the original problem, I get something like

```typescript
const partTwo = (lines: string[]) => {
 let xmasCount = 0;
 const keyToCountLookup = new Map<string, number>();
 for (let xPosition = 0; xPosition < lines.length; xPosition++) {
   const line = lines[xPosition];
   for (let yPosition = 0; yPosition < line.length; yPosition++) {
     const char = line[yPosition];
     if (char !== "M") {
       continue;
     }

     const keys = getIndexesOfAllAsInAllMASConfigurations(
       xPosition,
       yPosition,
       lines
     );

     for (const key of keys) {
       if (!keyToCountLookup.has(key)) {
         keyToCountLookup.set(key, 1);
       } else {
         const current = keyToCountLookup.get(key)!;
         keyToCountLookup.set(key, current + 1);
       }
     }
   }
 }

 let total = 0;
 for (const value of keyToCountLookup.values()) {
   if (value > 1) {
     total += value - 1;
   }
 }

 return total;
};

const checkAllEightDirectionsForXmas = (
 xPosition: number,
 yPosition: number,
 lines: string[]
) => {
 const xDeltas = [0, -1, 1];
 const yDeltas = [0, -1, 1];
 let countOfXmasAtPosition = 0;
 for (const xDelta of xDeltas) {
   for (const yDelta of yDeltas) {
     if (
       lines[xPosition + xDelta]?.[yPosition + yDelta] === "M" &&
       lines[xPosition + xDelta * 2]?.[yPosition + yDelta * 2] === "A" &&
       lines[xPosition + xDelta * 3]?.[yPosition + yDelta * 3] === "S"
     ) {
       countOfXmasAtPosition++;
     }
   }
 }

 return countOfXmasAtPosition;
};
```

Basically, I modified this to keep track of all the times the "A" in MAS was found more than one time. I get the wrong answer though doing this against my puzzle input (it found too many "X-MAS").

Okay, first miss of the year, that's alright. I add some debug logs

Here:

```typescript
for (const [key, value] of keyToCountLookup.entries()) {
  if (value > 1) {
    console.log("found X-MAS A at", ...key.split("_"), value, "times");
    total += value - 1;
  }
}
```

And here:

```typescript
const getIndexesOfAllAsInAllMASConfigurations = (
  xPosition: number,
  yPosition: number,
  lines: string[]
) => {
  const xDeltas = [0, -1, 1];
  const yDeltas = [0, -1, 1];
  let indexesOfA: string[] = [];
  for (const xDelta of xDeltas) {
    for (const yDelta of yDeltas) {
      if (
        lines[xPosition + xDelta]?.[yPosition + yDelta] === "A" &&
        lines[xPosition + xDelta * 2]?.[yPosition + yDelta * 2] === "S"
      ) {
        console.log("MAS found at", xPosition, yPosition);
        indexesOfA.push(`${xPosition + xDelta}_${yPosition + yDelta}`);
      }
    }
  }

  return indexesOfA;
};
```

I pretty print an input like so

```
   0 1 2 3 4 5 6 7 8 9
0  . . . . . . M A S .
1  . S A M . M S . . .
2  . . . S . . A . . .
3  . . A . A . M S . .
4  . M A S A M . . M M
5  . . . . . . . A . A
6  S . S . S . S . S S
7  . A . A . A . A . A
8  . . M . M . M . M M
9  . . . . . . M A S .
```

Then get the following logs:

```
MAS found at 0 6
MAS found at 1 3
MAS found at 1 5
MAS found at 3 6
MAS found at 4 1
MAS found at 4 1
MAS found at 4 5
MAS found at 4 5
MAS found at 4 8
MAS found at 4 9
MAS found at 8 2
MAS found at 8 2
MAS found at 8 4
MAS found at 8 4
MAS found at 8 6
MAS found at 8 6
MAS found at 8 8
MAS found at 8 9
MAS found at 9 6
found X-MAS A at 2 6 2 times
found X-MAS A at 7 3 2 times
found X-MAS A at 7 5 2 times
found X-MAS A at 7 7 2 times
```

Looking at the grid (X and Y are presentationally swapped), I'm correctly finding all the "MAS" instances at a glance. So my assumption about the "X" must be incorrect

Looking all the places it detected an "X", We see 3/4 are correct, but it detected an X here:

```
  5 6 7
1 M S .
2 . A .
3 . M S
```

So it's definitely an X, but a little lopsided. So to make an "X", "MAS" has to be diagonal, any other "MAS" would result in a "lopsided" "X" like above. Modifying my helper function to only look for diagonals, I simply remove the 0s from my deltas, like so

```typescript
const getIndexesOfAllAsInAllMASConfigurations = (
  xPosition: number,
  yPosition: number,
  lines: string[]
) => {
  const xDeltas = [-1, 1];
  const yDeltas = [-1, 1];
  let indexesOfA: string[] = [];
  for (const xDelta of xDeltas) {
    for (const yDelta of yDeltas) {
      if (
        lines[xPosition + xDelta]?.[yPosition + yDelta] === "A" &&
        lines[xPosition + xDelta * 2]?.[yPosition + yDelta * 2] === "S"
      ) {
        console.log("MAS 'M' found at", xPosition, yPosition);
        indexesOfA.push(`${xPosition + xDelta}_${yPosition + yDelta}`);
      }
    }
  }

  return indexesOfA;
};
```

Running again, it excludes the lopsided case and finds 9 (correct) for their example input. I clean up the problem, making my final part one and two solution this:

```typescript
import { getProblemLinesFromText } from "../utils";

const checkAllEightDirectionsForXmas = (
  xPosition: number,
  yPosition: number,
  lines: string[]
) => {
  const xDeltas = [0, -1, 1];
  const yDeltas = [0, -1, 1];
  let countOfXmasAtPosition = 0;
  for (const xDelta of xDeltas) {
    for (const yDelta of yDeltas) {
      if (
        lines[xPosition + xDelta]?.[yPosition + yDelta] === "M" &&
        lines[xPosition + xDelta * 2]?.[yPosition + yDelta * 2] === "A" &&
        lines[xPosition + xDelta * 3]?.[yPosition + yDelta * 3] === "S"
      ) {
        countOfXmasAtPosition++;
      }
    }
  }

  return countOfXmasAtPosition;
};

const getIndexesOfAllAsInAllMASConfigurations = (
  xPosition: number,
  yPosition: number,
  lines: string[]
) => {
  const xDeltas = [-1, 1];
  const yDeltas = [-1, 1];
  let indexesOfA: string[] = [];
  for (const xDelta of xDeltas) {
    for (const yDelta of yDeltas) {
      if (
        lines[xPosition + xDelta]?.[yPosition + yDelta] === "A" &&
        lines[xPosition + xDelta * 2]?.[yPosition + yDelta * 2] === "S"
      ) {
        indexesOfA.push(`${xPosition + xDelta}_${yPosition + yDelta}`);
      }
    }
  }

  return indexesOfA;
};

const partOne = (lines: string[]) => {
  let xmasCount = 0;
  for (let xPosition = 0; xPosition < lines.length; xPosition++) {
    const line = lines[xPosition];
    for (let yPosition = 0; yPosition < line.length; yPosition++) {
      const char = line[yPosition];
      if (char !== "X") {
        continue;
      }

      xmasCount += checkAllEightDirectionsForXmas(xPosition, yPosition, lines);
    }
  }

  return xmasCount;
};

const partTwo = (lines: string[]) => {
  const keyToCountLookup = new Map<string, number>();
  for (let xPosition = 0; xPosition < lines.length; xPosition++) {
    const line = lines[xPosition];
    for (let yPosition = 0; yPosition < line.length; yPosition++) {
      const char = line[yPosition];
      if (char !== "M") {
        continue;
      }

      const keys = getIndexesOfAllAsInAllMASConfigurations(
        xPosition,
        yPosition,
        lines
      );

      for (const key of keys) {
        if (!keyToCountLookup.has(key)) {
          keyToCountLookup.set(key, 1);
        } else {
          const current = keyToCountLookup.get(key)!;
          keyToCountLookup.set(key, current + 1);
        }
      }
    }
  }

  let xmasCount = 0;
  for (const value of keyToCountLookup.values()) {
    if (value > 1) {
      xmasCount += value - 1;
    }
  }

  return xmasCount;
};

const lines = await getProblemLinesFromText(__dirname);
export default {
  partOne: () => partOne(lines),
  partTwo: () => partTwo(lines),
};
```

On my problem input, I get 1912 which is correct! A small hiccup at day four, day 5 I'm watch out!
