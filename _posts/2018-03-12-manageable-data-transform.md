---
layout: post
title:  "Transformer"
date:   2018-03-12 20:23:30 +1100
categories: javascript
---
No matter how the data in an application is structured, chances are it will need to be structured differently in order for another system to consume it. When these two structures are large and completely different, it can make the code that switches between them difficult to test and maintain.

There is a pattern that I have used that helps break the problem down into manageable chunks that can be easily unit tested in isolation. I refer to it as the transformer pattern, and key is to break the transformation down into small, pure functions that take the source object as a parameter and then merge the results of all of them.

See it below, implemented in JS.

```javascript
import { merge } from 'lodash';

const transformAlpha = ({ a }) => ({ alpha: a });
const transformBeta = ({ b }) => ({ beta: [{ bigB: b}] });
const transformCharlie = ({ c: { realC } }) => ({ charlie: realC });

const defaultTransformers = [
  transformAlpha,
  transformBeta,
  transformCharlie,
];

const transform = transformers => data => transformers.reduce(
  (acc, transformer) => merge(acc, transformer(data)),
  {}
);
export default transform(defaultTransformers);
```

If we call the exported function with `{ a: 1, b: 2, c: { realC: 3 } }` we can expect it to return `{ alpha: 1, beta: [{ bigB: 2 }], charlie: 3 }`.

There is a lot happening here, so I will go through it bit by bit.

To start, a function that performs a deep merge is imported. `merge` from the `lodash` library is being used here, but there are countless alternatives.
```javascript
import { merge } from 'lodash';
```

Next the functions that will perform the transformations are defined. Each function takes in the entirety of the source object and unpacks the fields that are relevant using parameter destructuring. They return objects that represent a small subset of the final object that will be constructed.
Each of these can be exported separately and tested in isolation, being provided only the values they need.
```javascript
const transformAlpha = ({ a }) => ({ alpha: a });
const transformBeta = ({ b }) => ({ beta: [{ bigB: b}] });
const transformCharlie = ({ c: { realC } }) => ({ charlie: realC });

```

They are then all placed into an array of transformers. Ideally these operations should be atomic, so order here shouldn't matter.
```javascript
const defaultTransformers = [
  transformAlpha,
  transformBeta,
  transformCharlie,
];
```

Finally, the transform function, which takes a list of transformers and produces an anonymous function that takes the source object. Each transformer is called with the source object `data` and the result is merged with the accumulated object, which starts as an empty object `{}`.
```javascript
const transform = transformers => data => transformers.reduce(
  (acc, transformer) => merge(acc, transformer(data)),
  {}
);
```

This should yield the desired object, ready to be sent away and consumed.