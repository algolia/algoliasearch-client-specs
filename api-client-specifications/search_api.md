# Search API clients specifications

## Table of Contents

- [`account_client` interface](#account_client-interface)
- [`search_client` interface](#search_client-interface)
- [`search_index` interface](#search_index-interface)
- [Objects](#objects)
- [Responses](#responses)

## `search_client` interface

- Cross-application **and** cross-index operations

```java
interface account_client {

    function copy_index(source: search_index, destination: search_index) returns waitable

}
```

## `search_client` interface

- Inter-application **but** cross-index operations

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
    function generate_secured_api_key(parentKey: string, restriction: secured_api_key_restriction) return secured_api_key
    function get_secured_api_key_remaining_validity(secured_api_key: string) return int // or a duration, see `internals/`

    // Multiple* methods
    function multiple_batch(operations: [indexed_operation], opts: request_options) return multiple_batch_response
    function multiple_get_objects<T>(requests: [indexed_get_object], opts: request_options) return [<T>]
    function multiple_queries(queries: [indexed_query], opts: request_options) return multiple_queries_response

    // Multi-Cluster Management (MCM) methods
    function assign_user_id(userID: string, clusterName: string, opts: request_options) return created_response
    function get_top_user_id(opts: request_options) return get_top_user_id_response
    function get_user_id(userID: string, opts: request_options) return get_user_id_response
    function list_clusters(opts: request_options) return list_clusters_response
    function list_user_ids(page: int = 0, hitsPerPage: int = 20, opts: request_options) return list_user_ids_response
    function remove_user_id(userID: string, opts: request_options) return deleted_response
    function search_user_ids(query: string, clusterName: string = null, page: int = 0, hitsPerPage: int = 20, opts: request_options) return search_user_ids_response

    function set_personalization_strategy(strategy: strategy, opts: request_options) return set_personalization_strategy_response
    function get_personalization_strategy(opts: request_options) return get_personalization_strategy_response

}
```

## `search_index` interface

- Inter-application **and** inter-index operations

```ts
interface search_index {

    // Misc
    function wait_task(taskID: int)
    function get_status(taskID: int) return task_status_response
    function get_app_id() return string
    function clear(opts: request_options) return task_updated_response
    function delete(opts: request_options) return task_deleted_response
    function delete_replica(replica_name: string, opts: request_options)

    // Indexing
    function get_object<T>(objectID: string, opts: request_options) return <T>
    function get_objects<T>(objectIDs: [string], opts: request_options) return [<T>]
    function save_object<T>(object: <T>, opts: request_options) return task_created_object_id_response
    function save_objects<T>(objects: [<T>], opts: request_options) return group_batch_response
    function partial_update_object<T>(object: <T>, opts: request_options) return task_updated_response
    function partial_update_objects<T>(objects: [<T>], opts: request_options) return group_batch_response
    function delete_object(objectID: string, opts: request_options) return task_deleted_response
    function delete_objects(objectIDs: [string], opts: request_options) return batch_response
    function delete_by(opts: request_options) return task_updated_response
    function batch(operations: [batch_operation], opts: request_options) return batch_response

    // Query rules
    function get_rule(objectID: string, opts: request_options) return rule
    function save_rule(rule: rule, forward_to_replicas: bool = false, opts: request_options) return task_updated_response
    function save_rules(rule: [rule], forward_to_replicas: bool = false, clear_existing_rules: bool = false, opts: request_options) return task_updated_response
    function clear_rules(forward_to_replicas: bool = false, opts: request_options) return task_updated_response
    function delete_rule(objectID: string, forward_to_replicas: bool = false, opts: request_options) return task_updated_response

    // Synonyms
    function get_synonym(objectID: string, opts: request_options) return synonym
    function save_synonym(synonym: synonym, forward_to_replicas: bool = false, opts: request_options) return task_updated_response
    function save_synonyms(synonym: [synonym], forward_to_replicas: bool = false, replace_existing_synonyms: bool = false, opts: request_options) return task_updated_response
    function clear_synonyms(forward_to_replicas: bool = false, opts: request_options) return task_updated_response
    function delete_synonym(objectID: string, forward_to_replicas: bool = false, opts: request_options) return task_deleted_response

    // Browsing
    function browse(query string, params: [browse_parameter], opts: request_options) return browse_response
    function browse_objects(params: [query_browse_parameter], opts: request_options) return object_iterator
    function browse_rules(opts: request_options) return rule_iterator
    function browse_synonyms(opts: request_options) return synonym_iterator

    // Replacing
    function replace_all_objects<T>(objects: <T>, opts: request_options)
    function replace_all_rules(rules: [rule], forward_to_replicas: bool = false, opts: request_options) return task_updated_response
    function replace_all_synonyms(synonyms: [synonym], forward_to_replicas: bool = false, opts: request_options) return task_updated_response

    // Searching
    function search(query string, params: [search_parameter], opts: request_options) return query_response
    function search_for_facet_values(facet_name: string, facet_query: string, params: [search_parameter], opts: request_options) return query_response
    function search_rules(query string, params: [rule_search_parameter], opts: request_options) return rule_query_response
    function search_synonyms(query string, params: [synonym_search_parameter], opts: request_options) return synonym_query_response

    // Settings
    function get_settings(opts: request_options) return settings
    function set_settings(settings: settings, opts: request_options) return task_updated_response

    // Exists
    function exists() return bool

    // find_first_object search iteratively through the search response `hits`
    // field to find the first response hit that would match against the given
    // `filter_func` function.
    //
    // If no object has been found within the first result set, the function
    // will perform a new search operation on the next page of results, if any,
    // until a matching object is found or the end of results, whichever
    // happens first.
    //
    // To prevent the iteration through pages of results, `do_not_paginate`
    // parameter can be set to true. This will stop the function at the end of
    // the first page of search results even if no object does match.
    function find_first_object<T>(
        filter_func: function (object: T) return bool, // function matching a specific record
        query: string,                                    // the search query
        do_not_paginate: bool = false,                    // prevent pagination
        params: map[string, object]                       // any search parameter
    ) return object_with_position<T>
}
```

## Objects

```ts
enum log_type { "all", "query", "build", "error" }

enum scope { "rules", "settings", "synonyms" }

struct request_options {
    extra_headers: map<string, string>
    extra_url_params: map<string, string>
}

struct browse_parameter         // https://www.algolia.com/doc/api-reference/api-methods/browse/#method-param-browseparameters
struct search_parameter         // https://www.algolia.com/doc/api-reference/api-methods/search/#method-param-searchparameters
struct rule_search_parameter    // https://www.algolia.com/doc/api-reference/api-methods/search-rules/#parameters
struct synonym_search_parameter // https://www.algolia.com/doc/api-reference/api-methods/search-synonyms/#parameters

struct query_browse_parameter {
    query: string
    // + all browse_parameter fields
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

struct object_iterator<T>        // Language-specific representation of an iterator on arbitrary objects
struct rule_iterator<rule>       // Language-specific representation of an iterator on rules
struct synonym_iterator<synonym> // Language-specific representation of an iterator on synonyms

struct rule     // https://www.algolia.com/doc/api-reference/api-methods/save-rule/#method-param-rule
struct settings // https://www.algolia.com/doc/api-reference/settings-api-parameters/
struct synonym  // https://www.algolia.com/doc/api-reference/api-methods/save-synonym/#method-param-synonym-object

struct strategy {
    eventsScoring: map<string, events_scoring>
    facetsScoring: map<string, facets_scoring>
}

struct events_scoring {
    score: int
    type: string
}

struct facets_scoring {
    score: int
}

struct secured_api_key_restriction // https://www.algolia.com/doc/api-reference/api-methods/generate-secured-api-key/#parameters

struct object_with_position<T> {
  object: T
  position: int
  page: int
}
```

## Responses

All response objects are implemented the `waitable` interface in order for the
user to be able to wait their completion without dealing with the low-level
details.

```ts
interface waitable {

    function wait() return error

}
```

```ts
struct query_response {    // https://www.algolia.com/doc/api-reference/api-methods/search/#response
    // get_object_id_position returns the position (0-based) within the `hits`
    // result list of the record matching against the given objectID. If the
    // objectID is not found, -1 is returned.
    function get_object_id_position(objectID: string) return int
}

struct rule_query_response // https://www.algolia.com/doc/api-reference/api-methods/search-rules/#response

struct group_batch_response {
    responses: [batch_response]
}

struct batch_response {
    objectIDs: [string]
    taskID: int
}

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
    updatedAt: string // Format RFC3339 "2006-01-02T15:04:05Z07:00"
}

struct task_deleted_response {
    taskID: int
    deletedAt: string // Format RFC3339 "2006-01-02T15:04:05Z07:00"
}

struct task_status_response {
    status: string
    pendingTask: bool
}

struct task_created_object_id_response {
    createdAt: string // Format RFC3339 "2006-01-02T15:04:05Z07:00"
    objectID: string
    taskID: string
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

struct list_api_keys_response // https://www.algolia.com/doc/api-reference/api-methods/list-api-keys/#response

struct browse_response // https://www.algolia.com/doc/api-reference/api-methods/browse/#response

struct set_personalization_strategy_response {
    updatedAt: string // Format RFC3339 "2006-01-02T15:04:05Z07:00"
}

struct get_personalization_strategy_response {
    taskID: int
    // + all fields fields
}
```
