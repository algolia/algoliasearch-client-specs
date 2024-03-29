# Personalization API clients specifications

## Table of Contents

  - [`personalization_client` interface](#personalization_client-interface)
  - [Objects](#objects)
  - [Responses](#responses)

## `personalization_client` interface

```java
interface personalization_client {
    function get_personalization_profile(userToken: string, opts: request_options)
    return get_personalization_profile_response

    function delete_personalization_profile(userToken: string, opts: request_options)
    return delete_personalization_profile_response

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
struct get_personalization_profile_response {
  userToken: string,
  lastEventAt: datetime,
  scores: object
}
```

```java
struct delete_personalization_profile_response {
  userToken: string,
  deletedUntil: datetime
}
```

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
