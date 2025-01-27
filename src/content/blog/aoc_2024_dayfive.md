---
title: 'Advent of Code 2024: Day 4'
description: 'Day three of advent of code'
pubDate: 'Jan 7 2025'
heroImage: '/advent_of_code_2024.webp'
tags: ['aoc', '2024', 'typescript', 'bun']
---

# Advent of Code 2024 Day 5

## The Problem

Last time ([Advent of Code 2024: Day 4](/blog/advent-of-code-2024-day-4)), we had to find the words "XMAS" and "MAS" in a word search like manner in a string of text. Today, we need to validate updates to a machine and if the updates violate a set of rules

## Part One

This one took me a long time to understand, I had to read it and think about it a bit. My idea is construct a directed graph of the rules, then read in each number and all the previous numbers in the update. If we follow the directed graph, and any of the previous numbers appear in "front" of the current number, we have a violation.

So let's get the raw rules and updates to start, here's my skeleton

```typescript
import { getProblemText } from "../utils";

const partOne = (text: string) => {
  console.log(text);
  return 0;
};

const partTwo = (text: string) => {
  console.log(text);
  return 0;
};

const text = await getProblemText(__dirname);
export default {
  partOne: () => partOne(text),
  partTwo: () => partTwo(text),
};
```

Here's me getting the lines of rules and lines of updates. Rules are stored as an array of two wide number tuples, updates are a list of numbers

```typescript
const getRulesAndUpdates = (text: string) => {
  const [rulesSection, updatesSection] = text.split('\n\n');
  const rulesLines = rulesSection.split('\n');
  const updatesLines = updatesSection.split('\n');

  const rules = rulesLines.map(line => line.split('|').map(numberString => +numberString) as [number, number]);
  const updates = updatesLines.map(line => line.split(',').map(numberString => +numberString));

  return {rules, updates};
}
```

Now, if I take the rules, I can construct a graph. A javascript map will do just fine, of number to Set of numbers (to remove duplicate nodes)

```typescript
type Graph = Map<number, Set<number>>
const generateDirectedGraphFromRules = (rules: Array<[number, number]>) => {
  const graph: Graph = new Map<number, Set<number>>();

  for (const [before, after] of rules) {
    if (!graph.has(before)) {
      graph.set(before, new Set([after]));
    } else {
      const current = graph.get(before)!;
      current.add(after);
      graph.set(before, current);
    }
  }

  return graph;
};
```

Now I can do a depth or breadth first search to implement my idea. Here's how you do it,

1. start with an array "toExplore" with a starting node (or nodes), and a list or set of things you've seen so far (empty)

2. while the array is not empty, pop an item of the queue

3. if you've seen this before, continue

4. If you haven't seen this before, add it to the list of things you've seen

5. Check the current for what you're looking for

6. Get the connected nodes to the current based off of the current, and push them into the queue if there are any

What's great about this approach, is you can instead of popping, you can shift, and make it a breadth first search

Here's how I do it:

```typescript
const modifiedDFS = (startingNode: number, previousNumbers: number[], graph: Graph) => {
  const toExplore = [startingNode];
  const seen = new Set<number>();
  while (toExplore.length > 0) {
    const current = toExplore.pop()!;

    if (seen.has(current)) {
      continue;
    }
    seen.add(current);

    if (theThingImLookingForIsTrueForCurrent) {
      // found what I'm looking for, either exit or continue if you're
      // looking for multiple things
    }

    const next = graph.get(current);
    if (next) {
      toExplore.push(...next);
    }
  }
}
```

The theThingImLookingForIsTrueForCurrent is something like

```typescript
if (previousNumbers.includes(current)) {
  return false;
}
```

Then finally

```typescript
const previousNumbersAppearAfterStartingNode = (
  startingNode: number,
  previousNumbers: number[],
  graph: Graph
) => {
  const toExplore = [startingNode];
  const seen = new Set<number>();
  while (toExplore.length > 0) {
    const current = toExplore.pop()!;

    if (seen.has(current)) {
      continue;
    }
    seen.add(current);

    if (previousNumbers.includes(current)) {
      return false;
    }

    const next = graph.get(current);
    if (next) {
      toExplore.push(...next);
    }
  }

  return true;
};
```

So, we now need to keep track of all the numbers in the update we've processed, and add them to the running list of previous numbers, then run the previous algorithm we developed against every number in the update to figure out if the update is valid

```typescript
for (const update of updates) {
  const numbersProcessedSoFar = [];
  let updateValid = true;
  for (const page of update) {
    if (numbersProcessedSoFar.length === 0) {
      numbersProcessedSoFar.push(page);
      continue;
    }

    updateValid = previousNumbersAppearAfterStartingNode(
      page,
      numbersProcessedSoFar,
      graph
    );
    if (!updateValid) {
      break;
    }
  }

  if (updateValid) {
    // calculate the middle number of the array and add it to the running total
  }
}
```

Now just need to find the middle number and add it to the sum if the update was valid. The middle number, on an odd length array, should be

`arr[Math.floor(arr.length / 2)]`

Making my final solution

```typescript
const partOne = (text: string) => {
  const { rules, updates } = getRulesAndUpdates(text);
  const graph = generateDirectedGraphFromRules(rules);
  let sum = 0;
  for (const update of updates) {
    const numbersProcessedSoFar = [];
    let updateValid = true;
    for (const page of update) {
      if (numbersProcessedSoFar.length === 0) {
        numbersProcessedSoFar.push(page);
        continue;
      }
      updateValid = previousNumbersAppearAfterStartingNode(
        page,
        numbersProcessedSoFar,
        graph
      );
      if (!updateValid) {
        break;
      }

      numbersProcessedSoFar.push(page);
    }

    if (updateValid) {
      sum += update[Math.floor(update.length / 2)];
    }
  }

  return sum;
};
```

Using this code, I get 143 (correct) for their example input, and for my problem input I get 0. Oh no...

## Part One, take two

I added the following debug logs to my code

```typescript
const previousNumbersAppearAfterStartingNode = (
  startingNode: number,
  previousNumbers: number[],
  graph: Graph
) => {
  const toExplore = [[startingNode, []]] as Array<[number, number[]]>;
  const seen = new Set<number>();
  while (toExplore.length > 0) {
    const [current, parents] = toExplore.pop()!;
    if (seen.has(current)) {
      console.log("seen", current, "skipping");
      console.log();
      continue;
    }
    seen.add(current);

    if (previousNumbers.includes(current)) {
      console.log("previous numbers", previousNumbers);
      console.log("chain", parents.join(" -> "), "current", current);
      console.log();
      return false;
    }

    const next = graph.get(current);
    if (next) {
      next.forEach((item) => toExplore.push([item, [...parents, current]]));
    }
  }

  return true;
};
```

And found out, eventually, there was always an eventual rule chain that led to the first number in the update to violate. I reread the problem, maybe I over complicated things. Maybe I don't need to do a DFS and chain the rules, maybe I just need to check all the previous numbers against the current number, no traversal, maybe the problem is only one rule deep and I'm doing too much.

So, pivoting, I rename my previousNumbersAppearAfterStartingNode to validateRulesForNode, and my Graph type becomes "RulesLookup". I simplify the logic and do something like

```typescript
const validateRulesForNode = (
  numberToCheck: number,
  previousNumbers: number[],
  rulesLookup: RulesLookup
) => {
  return previousNumbers.every(
    (previousNumber) =>
      !(rulesLookup.get(numberToCheck)?.has(previousNumber) ?? false)
  );
};
```

And, again, get the correct value for their example input, and 5927 for my puzzle input, which is too low...

Alright, again, what if I'm doing to much, instead of checking all the previous numbers against the current number, just the immediate previous number may do it. I can't explain why my intuition told me this was the answer but I was pretty sure at this point. My validator function becomes

```typescript
const validateRulesForNode = (
  numberToCheck: number,
  previousNumber: number,
  rulesLookup: RulesLookup
) => {
  return !(rulesLookup.get(numberToCheck)?.has(previousNumber) ?? false);
};
```

And my partOne becomes

```typescript
const partOne = (text: string) => {
  const { rules, updates } = getRulesAndUpdates(text);
  const rulesLookup = generateRulesLookupFromRules(rules);
  let sum = 0;
  for (const update of updates) {
    let updateValid = true;
    for (let i = 1; i < updates.length; i++) {
      const previousPage = update[i - 1];
      const currentPage = update[i];
      updateValid = validateRulesForNode(currentPage, previousPage, rulesLookup);
      if (!updateValid) {
        break;
      }
    }

    if (updateValid) {
      sum += update[Math.floor(update.length / 2)];
    }
  }

  return sum;
};
```

This gives me the answer 143 again for the problem example input, and 5955 for my puzzle input, which is correct. A little dicey today, let's hope part two isn't too bad

## Part Two

I was stuck on this for a long time, and even tried another DFS style solution. Then I remembered, like in part one, maybe I'm doing too much. So let's start with rewriting the problem a bit. I rewrite part one to simultaneous calculate the solution and keep track of invalid updates like so:

```typescript
const partOne = (
  updates: number[][],
  rulesLookup: RulesLookup,
  invalidUpdates: number[][]
) => {
  let sum = 0;
  for (const update of updates) {
    if (validateUpdate(update, rulesLookup)) {
      sum += update[Math.floor(update.length / 2)];
    } else {
      invalidUpdates.push(update);
    }
  }

  return sum;
};

...

const text = await getProblemText(__dirname);
const { rules, updates } = getRulesAndUpdates(text);
const rulesLookup = generateRulesLookupFromRules(rules);
const invalidUpdates: number[][] = [];

const partTwo = (invalidUpdates: number[][], rulesLookup: RulesLookup) => {
  let sum = 0;
  for (const invalidUpdate of invalidUpdates) {
    const validUpdate = reorderUpdateInCorrectOrder(invalidUpdate, rulesLookup);
    sum += validUpdate[Math.floor(validUpdate.length / 2)];
  }

  return sum;
};

export default {
  partOne: () => partOne(updates, rulesLookup, invalidUpdates),
  partTwo: () => partTwo(invalidUpdates, rulesLookup),
};

```

Now I just need to implement `reorderUpdateInCorrectOrder`.

I'm going to be honest here, I needed some help at this point. I perused some other solutions and saw that people were cleverly sorting based on the rules. I spent a long time on a DFS graph style search solution with no luck, then again, realized I was doing too much. I took inspiration from other solutions and modified it to fit my own code. Essentially, within a sort callback, taking the "a" and "b" ("left" and "right" in my code), if there exists a rule that puts the first argument before the second, move the first one forward in the list (return -1), or if the opposite is true, move it up in the list (return 1). Otherwise, leave it where it is (return 0). It looks like this:

```typescript
const reorderUpdateInCorrectOrder = (
  update: number[],
  rulesLookup: RulesLookup
) => {
  return update.sort((left, right) => {
    if (rulesLookup.get(left)?.has(right)) {
      return -1;
    }

    if (rulesLookup.get(right)?.has(left)) {
      return 1;
    }

    return 0;
  });
};
```

This returns 123 as expected for the example input and 4030 for my problem input. Feeling a little defeated today that I needed inspiration from other solutions only on day 5, I march forward to day 6.
