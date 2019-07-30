# `search_client.get_secured_api_key_remaining_validity` function

## Context

When a secured API key expires, search breaks. The workaround is more complex
than it needs to be. The options for developers using Algolia require, at every
place where a search operation is performed, catching an operation error,
identify it as a `valid_until` error, then regenerate the API key, and perform
the search operation again.

The idea of `get_secured_api_key_remaining_validity` is a first step to address
this issue. This function tells if a secured API key is expired or not without
making any network call to the Algolia's API. This function may be called
before instantiating a new `search_client` or before performing a
`search_index.search`. Depending on the result, the user can decide to
regenerate on the fly a secured API key to avoid the search to break live.

**Note:** For the JavaScript client, it's recommended to regenerate the secured
API key on server side to avoid security breach, since everyone is able to
modify the filters by modifying the code from the browser.

## Implementation

```ts
interface search_client {

  // get_secured_api_key_remaining_validity takes a secured API key (which is a
  // base64-encoded string) as a parameter and return an integer (or a duration if
  // the language supports it) telling how many seconds are left before the key
  // expiration.
  function get_secured_api_key_remaining_validity(secured_api_key: string) return int // or a duration if the language supports it
  {
      // 1. Check if the secured API key is valid base64 (not null, no whitespace, etc.)
      // 2. Decode the base64-encoded string
      // 3. Find with a regex the pattern `validUntil=XXX` where `XXX` is a
      //    valid UNIX epoch timestamp (in seconds)
      // 4. If no `validUntil` is found, return the following error:
      //    "no `validUntil` parameter found, please make sure the secured API key has one"
      // 5. Otherwise, extract the `XXX` UNIX timestamp (in seconds) and return
      //    the following evaluation: `time.now - XXX`
  }

}
```

## Test plan

 - [CTS test](../common-test-suite#expired-secured-api-keys)
