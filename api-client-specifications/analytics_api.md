# Analytics API clients specifications

## Table of Contents

- [`analytics_client` interface](#analytics_client-interface)
- [Objects](#objects)
- [Responses](#responses)

## `analytics_client` interface

```java
function init_analytics_client(appid: string, api_key: string) returns analytics_client
function init_analytics_client_with_config(config: analytics_configuration) returns analytics_client

interface analytics_client {

    function add_ab_test(ab: ab_test, opts: request_options) return task_ab_test_response
    function get_ab_test(id: int, opts: request_options) return get_ab_test_response
    function get_ab_tests(opts: request_options) return get_ab_tests_response
    function stop_ab_test(id: int, opts: request_options) return task_ab_test_response
    function delete_ab_test(id: int, opts: request_options) return task_ab_test_response

}
```

## Objects

```ts
struct ab_test {
    name: string
    variants: [variant]
    endAt: string // Format ISO8601 "2006-01-02T15:04:05Z"
}

struct variant {
    index: string
    trafficPercentage: int
    description: string (optional)
    customSearchParameters: map<string, object> (optional) // Accepts any search_parameter
}
```

## Responses

```ts
struct task_ab_test_response {
    abTestID: int
    index: string
    taskID: int
}

struct get_ab_tests_response {
    abtests: [ab_test_response]
    count: int
    total: int
}

struct ab_test_response {
    abTestID: int
    clickSignificance: int
    conversionSignificance: float
    createdAt: string
    endAt: string
    name: string
    status: string
    variants: [variant_response]
}

struct variant_response {
    averageClickPosition: int
    clickCount: int
    clickThroughRate: float
    conversionCount: int
    conversionRate: float
    description: string
    index: string
    noResultCount: int
    searchCount: int
    trafficPercentage: int
    userCount: int
}
```
