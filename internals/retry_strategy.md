# Retry strategy

- Lets us test the retry strategy as a separate component
- No network involved so it could easily be unit-tested

```java
function request(method, path, body, call_kind, request_options) return body_response {

  for host in retry_strategy.get_tryable_hosts(call_kind) {
    request  = build_request(method, host.host, path, body, request_options)
    response = http_requester.request(request, connect_timeout, write_timeout)
    outcome  = retry_strategy.decide(response)

    switch outcome {
      case success -> return response.body
      case failure -> return response.body
    }

    // Do not handle the retry case in the switch,
    // so that we will loop to the next tryable_host.
  }

  // Having looped without early returning with neither
  // a success or a failure, it means all tryable hosts
  // have been exhausted. Hence an error has to be raised.
  return exhausted_hosts
}

interface retry_strategy {
  function get_retryable_hosts(call_kind) return [tryable_host]
  function decide(response) return outcome
  function set_timeouts(read_timeout, write_timeout)
}

struct tryable_host {
  host (string)
  timeout (duration)
}

enum call_kind {
  read
  write
}

enum outcome {
  success
  failure
  retry
}

interface http_requester {
  function request(request, connect_timeout, total_timeout) return response
}

struct request {
  http_method (string)
  uri (string)
  headers (map)
  body (string or reader)
}

struct response {
  http_code (int)
  body (string or reader)
  is_timeout_error (bool)
  is_network_error (bool)
  error (string or reader)
}
```
