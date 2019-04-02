# Search API clients specifications

## Table of Contents

* [`search_client` interface](#search_client-interface)
* [`search_index` interface](#search_index-interface)
* [Objects](#objects)
* [Responses](#responses)

## `search_client` interface

* Inter-application **but** cross-index operations

```java
interface search_client {

    // Misc
    function init_index(index_name: string) return search_index
    function list_indexes(opts: request_options) return list_indexes_response
    function get_logs(offset: int = 0, length: int = 10, type: log_type) return get_logs_response

    // Copy index operations
    function copy_rules(source: string, destination: string, opts: request_options) return task_updated_response
    function copy_settings(source: string, destination: string, opts: request_options) return task_updated_response
    function copy_synonyms(source: string, destination: string, opts: request_options) return task_updated_response
    function copy_index(source: string, destination: string, scopes: [scope], opts: request_options) return task_updated_response

    // Move index methods
    function move_rules(source: string, destination: string, opts: request_options) return task_updated_response
    function move_settings(source: string, destination: string, opts: request_options) return task_updated_response
    function move_synonyms(source: string, destination: string, opts: request_options) return task_updated_response
    function move_index(source: string, destination: string, scopes: [scope], opts: request_options) return task_updated_response

    // API key methods
    function get_api_key(keyID: string, opts: request_options) return key
    function add_api_key(key: key, opts: request_options) return key_created_response
    function update_api_key(key: key, opts: request_options) return key_updated_response
    function delete_api_key(key: key, opts: request_options) return deleted_response
    function restore_api_key(keyID: string, opts: request_options) return created_response
    function list_api_keys(opts: request_options) return list_api_keys_response

    // Multiple* methods
    function multiple_batch(operations: [indexed_operation], opts: request_options) return multiple_batch_response
    function multiple_get_objects<T>(requests: [indexed_get_object], opts: request_options) return <T>
    function multiple_queries(queries: [indexed_query], opts: request_options) return multiple_queries_response

    // Multi-Cluster Management (MCM) methods
    function assign_user_id(userID: string, clusterName: string, opts: request_options) return created_response
    function get_top_user_id(opts: request_options) return get_top_user_id_response
    function get_user_id(userID: string, opts: request_options) return get_user_id_response
    function list_clusters(opts: request_options) return list_clusters_response
    function list_user_ids(page: int = 0, hitsPerPage: int = 20, opts: request_options) return list_user_ids_response
    function remove_user_id(userID: string, opts: request_options) return deleted_response
    function search_user_ids(query: string, clusterName: string = null, page: int = 0, hitsPerPage: int = 20, opts: request_options) return search_user_ids_response

}
```

## `search_index` interface

* Inter-application **and** inter-index operations

```js
interface search_index {
}
```

## Objects

```js
enum log_type { "all", "query", "build", "error" }

enum scope { "rules", "settings", "synonyms" }

struct request_options {
    extra_headers: map<string, string>
    extra_url_params: map<string, string>
}

struct key // https://www.algolia.com/doc/api-reference/api-methods/add-api-key/#parameters

struct indexed_operation {
    indexName: string
    // + all batch_operation fields
}

struct indexed_get_object {
    indexName: string
    objectID: string
    attributesToRetrieve: string // Comma-separated string
}

struct indexed_query {
    indexName: string
    // + all query fields
}
```

## Responses

```js
struct search_response https://www.algolia.com/doc/api-reference/api-methods/search/#response

struct multiple_batch_response {
    objectIDs: [string]
    taskID: map<string, int> // Mapping of (index name -> taskID)
}

struct multiple_queries_response {
    results: [multiple_queries_query_response]
}

struct multiple_queries_query_response {
    processed: bool
    // + all query_response fields
}

struct get_top_user_id_response // https://www.algolia.com/doc/api-reference/api-methods/get-top-user-id/#response
struct get_user_id_response     // https://www.algolia.com/doc/api-reference/api-methods/get-user-id/#response
struct list_clusters_response   // https://www.algolia.com/doc/api-reference/api-methods/list-clusters/#response
struct list_user_ids_response   // https://www.algolia.com/doc/api-reference/api-methods/list-user-id/#response
struct search_user_ids_response // https://www.algolia.com/doc/api-reference/api-methods/search-user-id/#response

struct list_indexes_response // https://www.algolia.com/doc/api-reference/api-methods/list-indices/#response
struct get_logs_response     // https://www.algolia.com/doc/api-reference/api-methods/get-logs/#response

struct task_updated_response {
    taskID: int
    uddatedAt: string // Format RFC3339 "2006-01-02T15:04:05Z07:00"
}

struct created_response {
    createdAt: string // Format RFC3339 "2006-01-02T15:04:05Z07:00"
}

struct deleted_response {
    deletedAt: string // Format RFC3339 "2006-01-02T15:04:05Z07:00"
}

struct key_created_response {
    key: string
    createdAt: string // Format RFC3339 "2006-01-02T15:04:05Z07:00"
}

struct key_updated_response {
    key: string
    updatedAt: string // Format RFC3339 "2006-01-02T15:04:05Z07:00"
}

struct list_api_keys_response https://www.algolia.com/doc/api-reference/api-methods/list-api-keys/?language=csharp#response
```
