---
id: Basic Usage
sidebar_position: 2
---
import {WorkerWrapperComponent} from '@site/src/components/WorkerWrapper.mdx';

## Setting Up Your Main File

```javascript tab={"label":"Javascript"} title='main.js'
const path = require("path");
const Piscina = require("piscina");

// Create a new Piscina instance pointing to your worker file
const piscina = new Piscina({
  filename: path.resolve(__dirname, "worker.js"),
});

// Run a task using Piscina
(async () => {
  const result = await piscina.run({ a: 4, b: 6 });
  console.log(result); // prints 10
})();
```

```javascript tab={"label":"Typescript"} title='main.ts'
import Piscina from "piscina";
import { resolve } from "path";
import { filename } from "./worker";

const piscina = new Piscina({
  filename: resolve(__dirname, "./workerWrapper.js"),
  workerData: { fullpath: filename },
});

(async () => {
  const result = await piscina.run({ a: 2, b: 3 }, { name: "addNumbers" });
  console.log("Result:", result);
})();
```

## Using a Worker Wrapper File in a TypeScript Project

When working with TypeScript, you need to use a `workerWrapper.js` file to load your worker file correctly. This file checks if the provided file path ends with `.ts` and registers the `ts-node` compiler to handle TypeScript files.

<WorkerWrapperComponent/>

## Creating a Worker File

```javascript tab={"label":"Javascript"} title='worker.js'
// A simple worker function that adds two numbers
module.exports = ({ a, b }) => a + b;
```

```javascript tab={"label":"Typescript"} title='worker.ts'
// A simple worker function that adds two numbers
const { resolve } = require("path");

export const filename = resolve(__filename);
interface Inputs {
  a: number;
  b: number;
}
export function addNumbers({ a, b }: Inputs): number {
  return a + b;
}
```

## Using Async Functions or Promises

Workers can be asynchronous or return a promise:

```javascript tab={"label":"Javascript"}
const { setTimeout } = require("timers/promises");

// An async worker function with simulated delay
module.exports = async ({ a, b }) => {
  // Fake some async activity with a delay
  await setTimeout(100);
  return a + b;
};
```

```javascript tab={"label":"Typescript"}
const { setTimeout } = require("timers/promises");

interface WorkerInput {
  a: number;
  b: number;
}
// An async worker function with simulated delay
module.exports = async ({ a, b }: WorkerInput): Promise<number> => {
  // Fake some async activity with a delay
  await setTimeout(100);
  return a + b;
};
```

## Supporting ECMAScript Modules (ESM)

Piscina.js also works with ECMAScript modules:

```javascript tab={"label":"Javascript"}
import { Piscina } from "piscina";

const piscina = new Piscina({
  // The URL must be a file:// URL
  filename: new URL("./worker.mjs", import.meta.url).href,
});

const result = await piscina.run({ a: 4, b: 6 });
console.log(result); // Prints 10
```

```javascript tab={"label":"Typescript","span":2}
import { Piscina } from "piscina";
import { filename } from "./worker";

const workerWrapperURL = new URL("workerWrapper.js", import.meta.url);

const piscina = new Piscina({
  filename: workerWrapperURL.href,
  workerData: { fullpath: filename },
});
(async () => {
  const result = await piscina.run({ a: 4, b: 6 }, { name: "addNumbers" });
  console.log(result); // Prints 10
})();
```
<WorkerWrapperComponent/>

In your worker module file:

```javascript tab={"label":"Javascript"} title="worker.mjs"
// Default export of an addition function
export default ({ a, b }) => a + b;
```

```javascript tab={"label":"Typescript"} title="worker.ts"
// Default export of an addition function
interface Inputs {
  a: number;
  b: number;
}

export default function addNumbers({ a, b }: Inputs): number {
  return a + b;
}
```

## Exporting Multiple Worker Functions

A single worker file may export multiple named handler functions:

```javascript tab={"label":"Javascript"} title="worker.js"
"use strict";

function add({ a, b }) {
  return a + b;
}

function multiply({ a, b }) {
  return a * b;
}

add.add = add;
add.multiply = multiply;

module.exports = add;
```

```javascript tab={"label":"Typescript"} title="worker.ts"
import { resolve } from "path";

export const filename = resolve(__filename);

function add({ a, b }: { a: number, b: number }): number {
  return a + b;
}

function multiply({ a, b }: { a: number, b: number }): number {
  return a * b;
}

export { multiply, add };
```

The export to target can then be specified when the task is submitted:

```javascript tab={"label":"Javascript"} title="main.js"
"use strict";

const Piscina = require("piscina");
const { resolve } = require("path");

// Initialize Piscina with the worker file
const piscina = new Piscina({
  filename: resolve(__dirname, "worker.js"),
});

// Run multiple tasks concurrently
(async () => {
  const [sum, product] = await Promise.all([
    piscina.run({ a: 4, b: 6 }, { name: "add" }),
    piscina.run({ a: 4, b: 6 }, { name: "multiply" }),
  ]);
})();
```

```javascript tab={"label":"Typescript","span":2} title="main.ts"
import Piscina from "piscina";
import { resolve } from "path";
import { filename } from "./worker";

const piscina = new Piscina({
  filename: resolve(__dirname, "./workerWrapper.js"),
  workerData: { fullpath: filename },
});

// Run multiple tasks concurrently
(async () => {
  const [sum, product] = await Promise.all([
    piscina.run({ a: 4, b: 6 }, { name: "add" }),
    piscina.run({ a: 4, b: 6 }, { name: "multiply" }),
  ]);
})();
```
<WorkerWrapperComponent/>

## Cancellable Tasks

Piscina supports task cancellation even if the task has already been submitted to a worker and is being actively processed. This can be used, for instance, to cancel tasks that are taking too long to run.

Tasks can be canceled using an `AbortController` or an `EventEmitter`.

### Using `AbortController`

```javascript tab={"label":"Javascript"} title="main.js"
"use strict";

const Piscina = require("piscina");
const { resolve } = require("path");

// Set up Piscina with the worker file
const piscina = new Piscina({
  filename: resolve(__dirname, "worker.js"),
});

// Run a task and cancel it using AbortController
(async () => {
  const abortController = new AbortController();
  const { signal } = abortController;
  const task = piscina.run({ a: 4, b: 6 }, { signal });
  abortController.abort(); // Cancel the task

  try {
    await task;
  } catch (err) {
    console.log("The task was canceled"); // Handle the cancellation
  }
})();
```

```javascript tab={"label":"Typescript","span":2} title="main.ts"
import Piscina from "piscina";
import { resolve } from "path";
import { filename } from "./worker";

// Set up Piscina with the worker and workerWrapper file
const piscina = new Piscina({
  filename: resolve(__dirname, "./workerWrapper.js"),
  workerData: { fullpath: filename },
});

// Run a task and cancel it using AbortController
(async () => {
  const abortController = new AbortController();
  const { signal } = abortController;
  const task = piscina.run({ a: 4, b: 6 }, { signal });
  abortController.abort(); // Cancel the task

  try {
    await task;
  } catch (err) {
    console.log("The task was canceled"); // Handle the cancellation
  }
})();
```
<WorkerWrapperComponent/>

### Using `EventEmitter`

A plain Node.js EventEmitter can also be used here, in which case, emitting the abort event on it will have the same effect.

```javascript tab={"label":"Javascript"} title="main.js"
"use strict";

const Piscina = require("piscina");
const EventEmitter = require("events");
const { resolve } = require("path");

// Initialize Piscina with the worker file
const piscina = new Piscina({
  filename: resolve(__dirname, "worker.js"),
});

// Run a task and cancel it using an EventEmitter
(async () => {
  const ee = new EventEmitter();
  const task = piscina.run({ a: 4, b: 6 }, { signal: ee });
  ee.emit("abort"); // Emit an 'abort' event to cancel the task

  try {
    await task;
  } catch (err) {
    console.log("The task was canceled"); // Handle the cancellation
  }
})();
```

```javascript tab={"label":"Typescript","span":2} title="main.ts"
import Piscina from "piscina";
import { resolve } from "path";
import { filename } from "./worker";

// Initialize Piscina with the worker and workerWrapper file
const piscina = new Piscina({
  filename: resolve(__dirname, "./workerWrapper.js"),
  workerData: { fullpath: filename },
});

// Run a task and cancel it using an EventEmitter
(async () => {
  const ee = new EventEmitter();
  const task = piscina.run({ a: 4, b: 6 }, { signal: ee });
  ee.emit("abort"); // Emit an 'abort' event to cancel the task

  try {
    await task;
  } catch (err) {
    console.log("The task was canceled"); // Handle the cancellation
  }
})();
```

<WorkerWrapperComponent/>

Find more use cases in the [Examples](../category/examples/) section of the documentation