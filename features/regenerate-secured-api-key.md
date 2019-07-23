# Feature Proposal :  Regenerate Expired SecuredAPIKeys

### 💼 Customers Feedback

>When a secured API keys expire, search breaks. The workaround is more complex
than it needs to be.
>The options for developers using Algolia require, at every place where a search
operation is performed, catching an operation error, identify it as a
"ValidUntil" error, then regenerate the API key, and perform the search
operation again.

### 👨‍💻👩‍💻 Feature Proposal

The idea is to define a method which tells if a `SecuredAPIKey` is expired or not **without making a single call to Algolia's API**. We could call this method before instantiating a new `SearchClient` or before performing a `Search` and depending on the result let our users decide to regenerate *on the fly* a `SecuredAPIKey` to avoid the search to break live.

> Note : For the JS clients, it's recommended for our users to regenerate the `SecuredAPIKey` on server side to avoid security breach, since someone is able to modify the filters by modifying the code from the browser.

## 💻 Language Agnostic Implementation Proposal

### 📝 Implementation

```java
/**
 *  Takes the encoded string (base 64) securedAPIKey as parameter.
 *  Returns an int telling how many seconds are left before the expiration date.
 */
function int get_secured_api_key_remaining_validity(securedAPIKey: string)
{
    /** 1. Check if the securedAPIKey parameter is null/empty/white-spaces/encoded in base64 */

    /** 2. Decode (from base 64) the securedAPIKey parameter */

    /**
     *  3. Find with a regex the 'validUntil=0000000000' parameter.
     *  Note : validUntil is a UnixTimeStamp in seconds.
     *  Its maximum lenght is 10, this can be found AlgoliaSaas code base with -> rg 'validUntil'
     *  Proposed regex to extract the parameter:
     *  regex = 'validUntil=\d{10}'
     * If not valindUntil parameter -> throw or return an error: "noValidUntil parameter found, please make sure the securedAPIKey has one"
     */

    /**
     *  4. Get the UnixTimeStamp from the extracted string.
     *  Proposed way of doing it: removing 'validUntil=' in the extraced string.
     */

    /**
     *  5. Return the following evaluation :
     */

    return extractedTimeStamp - unixTimeStamp.now
}
```

### 🧪 Tests

```java
function secured_api_key_should_not_be_expired()
{
    /**
     *  1. Generate a SecuredAPIKey expiring in 10 minutes
     */

    /**
     *  2. Call get_secured_api_key_remaining_validity with the given SecuredAPIKey
     */

    /**
     *  3. Assert that the result is > 0
     */
}
```

```java
function secured_api_key_should_be_expired()
{
    /**
     *  1. Generate a SecuredAPIKey which expired 10 minutes ago.
     */

    /**
     *  2. Call get_secured_api_key_remaining_validity with the given SecuredAPIKey
     */

    /**
     *  3. Assert that the result is < 0
     */
}
```

```java
function test_parameters_validity()
{
    /**
     *  1. Call get_secured_api_key_remaining_validity with a non Base64 string
     */

    /**
     *  2. Assert that get_secured_api_key_remaining_validity is retuning or throwing an error.
     */

    /**
     *  3. Call get_secured_api_key_remaining_validity with an empty string
     */

    /**
     *  4. Assert that get_secured_api_key_remaining_validity is retuning or throwing an error.
     */

    /**
     *  5. Call get_secured_api_key_remaining_validity with a null string
     */

    /**
     *  6. Assert that get_secured_api_key_remaining_validity is retuning or throwing an error.
     */
}
```

### 🏄‍ C# Implementation Draft

The draft of the C# implementation can be found [here](https://gist.github.com/Ant-hem/5b47e1f92f750c6aea8197dd36ab0c08).
