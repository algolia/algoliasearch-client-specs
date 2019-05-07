# Insights API clients specifications

## Table of Contents

- [`insights_client` interface](#insights_client-interface)
- [`insights_user_client` interface](#insights_user_client-interface)
- [Objects](#objects)
- [Responses](#responses)

## `insights_client` interface

```java
interface insights_client {
    function user(userToken: string) return insights_user_client
    function send_event(event: event, opts: request_options) return status_message_response
    function send_events(events: [event], opts: request_options) return status_message_response
}
```

## `insights_user_client` interface

```java
interface insights_user_client {
    function clicked_object_ids(eventName: string, indexName: string, objectIDs: [string], opts: request_options) return status_message_response
    function clicked_object_ids_after_search(eventName: string, indexName: string, objectIDs: [string], positions: [int], queryID: string, opts: request_options) return status_message_response
    function clicked_filters(eventName: string, indexName: string, filters: [string], opts: request_options) return status_message_response

    function converted_object_ids(eventName: string, indexName: string, objectIDs: [string], opts: request_options) return status_message_response
    function converted_object_ids_after_search(eventName: string, indexName: string, objectIDs: [string], queryID: string, opts: request_options) return status_message_response
    function converted_filters(eventName: string, indexName: string, filters: [string], opts: request_options) return status_message_response

    function viewed_object_ids(eventName: string, indexName: string, objectIDs: [string], opts: request_options) return status_message_response
    function viewed_filters(eventName: string, indexName: string, filters: [string], opts: request_options) return status_message_response
}
```

## Objects

```java
struct event {
    eventType: string
    eventName: string
	index:     string
	userToken: string
	timestamp: int      // Unix epoch timestamp (in second)
	objectIDs: [string]
	positions: [int]
	queryID:   string
	filters:   [string]
}
```

## Responses

```java
struct status_message_response {
    status: int
    message: string
}
```
