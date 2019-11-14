# Recommendation API clients specifications

## Table of Contents

  - [`recommendation_client` interface](#recommendation_client-interface)
  - [Objects](#objects)
  - [Responses](#responses)

## `recommendation_client` interface

```java
interface recommendation_client {

    function get_personalization_strategy(opts: request_options)
    return get_personalization_strategy_response

    function set_personalization_strategy(set_strategy_request: strategy, opts: request_options)
    return set_personalization_strategy_response
}
```

## Objects

```java
struct set_strategy_request {
    eventsScoring: events_scoring[]
    facetsScroing: facets_scoring[]
    personalizationImpact: int
}
```

```java
struct events_scoring {
    eventName: string
    eventType: string
    score: int
}
```

```java
struct facets_scoring {
    facetName: string
    score: int
}
```

## Responses

```java
struct get_personalization_strategy_response {
    eventsScoring: events_scoring[]
    facetsScroing: facets_scoring[]
    personalizationImpact: int
}
```

```java
struct set_personalization_strategy_response {
    status: int
    message: string
}
```
