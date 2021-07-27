# Recommendations API clients specifications

## Table of Contents

- [`recommendations_client` interface](#recommendations_client-interface)
- [Objects](#objects)
- [Responses](#responses)

## `recommendations_client` interface

```java
interface recommendations_client {
    function get_recommendations(queries: [recommendations_query]) return get_recommendations_response
    function get_related_products(queries: [related_products_query]) return get_recommendations_response
    function get_frequently_bought_together(queries: [frequently_bought_together_query]) return get_recommendations_response
}
```

## Objects

```java
struct recommendations_search_options {
    // all algolia search parameters except the pagination: page, hitsPerPage, offset, length
    // https://www.algolia.com/doc/api-reference/search-api-parameters/
}
```

```java
struct recommendations_query {
    indexName: string,
    model: 'bought-together' | 'related-products',
    objectID: string,
    // optional
    threshold: int,
    maxRecommendations: int,
    queryParameters: recommendations_search_options,
    fallbackParameters: recommendations_search_options
}
```

```java
struct related_products_query {
    indexName: string,
    objectID: string,
    // optional
    threshold: int,
    maxRecommendations: int,
    queryParameters: recommendations_search_options,
    fallbackParameters: recommendations_search_options
}
```

```java
struct frequently_bought_together_query {
    indexName: string,
    objectID: string,
    // optional
    threshold: int,
    maxRecommendations: int,
    queryParameters: recommendations_search_options,
}
```

## Responses

```java
struct get_recommendations_reponse_hit {
    _score: float, // https://www.algolia.com/doc/api-reference/api-methods/get-recommendations/#method-response-_score
    // + all search query response hit: https://www.algolia.com/doc/api-reference/api-methods/search/#method-response-hits
}
```

```java
struct get_recommendations_result {
    hits: [get_recommendations_reponse_hit]
    // + all search query response options: https://www.algolia.com/doc/api-reference/api-methods/search/#response
}
```

```java
struct get_recommendations_reponse {
    results: [get_recommendations_result]
}
```
