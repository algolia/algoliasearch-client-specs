# Feature Proposal: Index Exists

## 💼 Customers Feedback

> Saving an object will either create a new index (if it didn't exist) or save into an existing index. We don't have any method to ensure we're working on an index that doesn't exist.

## 👨‍💻👩‍💻 Feature Proposal

The idea is to define a method which tells whether a `SearchIndex` exists or not, at a given time. Under the hood, we would perform a `GetSettings` and catch the 404 response from the engine.

[We've agreed](https://github.com/algolia/algoliasearch-client-specs/pull/26#issuecomment-507689959) that the method should live on the `SearchIndex`, and should be called `Exists`.

## 📝 Implementation

```
/**
 * Checks if the index exists.
 *
 * Returns a boolean telling if the index exists or not.
 */
function bool exists()
{
    /**
     *  1. Try/catch a `GetSettings` on the index.
     */

    /**
     *  2. If the engine returns a 404 error, return `false`.
     */

    /**
     *  3. Otherwise, return `true`.
     */
}
```

## 🧪 Tests

```
function index_exists()
{
    /**
     *  1. Call `exists` on the index and assert that the result is `false`.
     */

    /**
     *  2. Save an object to the index to create it.
     */

    /**
     *  3. Call `exists` on the index and assert that the result is `true`.
     */
}
```

> **Note**: this tests perform two assertions in one, therefore isn't atomic. The upsides are that it can be ran in parallel (it doesn't rely on a state left by a previous test) and it doesn't strain the API (we create the index once).

## 🏄 ‍PHP Implementation Draft

The draft of the PHP implementation can be found [here](https://gist.github.com/sarahdayan/c6fecddbfb0741dfa832e40e1f474ae2).
