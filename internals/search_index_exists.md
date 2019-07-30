# `search_index.exists` function

## Context

Because saving an object will either create a new index (if it didn't exist) or
save into an existing index, we don't have any method to ensure we're working
on an index that doesn't exist.

The idea is to define a method which tells whether a `search_index` exists or
not, at a given time. Under the hood, we would perform a `get_settings` and
catch the 404 response from the engine.

## Implementation

```ts
interface search_index {

  // exists return a boolean telling whether the index exists or not.
  function exists() return bool
  {
      // 1. Perform a `get_settings` on the index
      // 2. If the engine answered with a 404, return `false`, otherwise return
      //    `true`
      // 3. If a network issue occured, bubble up the error to the user
      //    (exception or error value depending on the language)
  }

}
```

## Test plan

 - [CTS test](../common-test-suite#exists)
