# Common Test Suite

## Table of Contents

* [Rationale](#rationale)
* [API clients](#api-clients)
* [Test suite](#test-suite)
  * [Environment variables](#environment-variables)
  * [Index naming convention](#index-naming-convention)
  * [Tests (index)](#tests-index)
    * [Indexing](#indexing)
    * [Settings](#settings)
    * [Search](#search)
    * [Synonyms](#synonyms)
    * [Query rules](#query-rules)
    * [Batching](#batching)
    * [Replacing](#Replacing)
  * [Tests (client)](#tests-client)
    * [Copy index](#copy-index)
    * [Multi Cluster Management (MCM)](#multi-cluster-management-mcm)
    * [API keys](#api-keys)
    * [Get logs](#get-logs)
    * [Multiple Operations](#multiple-operations)
    * [DNS timeout](#dns-timeout)
    * [Personalization Strategy](#personalization-strategy)
  * [Tests (account)](#tests-account)
    * [Copy index](#copy-index-1)
  * [Tests (secured API keys)](#tests-secured-api-keys)
    * [Generate secured API keys](#generate-secured-api-keys)
  * [Tests (analytics)](#tests-analytics)
    * [AB testing](#ab-testing)
    * [AA testing](#aa-testing)
  * [Tests (backward compatibility)](#tests-backward-compatibility)
    * [Old settings](#old-settings)
    * [Query rules v1](#query-rules-v1)

## Rationale

This document intends to specify all tests that must be implemented for all
Algolia API clients. Note that API clients may implement more tests, depending
on implementations. The list is to be considered as a required minimum.

## API clients

* C#
* Go
* Java
* PHP
* Python
* Ruby
* Scala

## Test suite

### Environment variables

To ease the testing process, all configuration must be passed via environment
variables. To run the tests, the following environment variables must be set:

| Name                         | Value                    |
| ---------------------------- | ------------------------ |
| `ALGOLIA_APPLICATION_ID_1`   | `NOCTT5TZUU`             |
| `ALGOLIA_ADMIN_KEY_1`        | (Personify `NOCTT5TZUU`) |
| `ALGOLIA_SEARCH_KEY_1`       | (Personify `NOCTT5TZUU`) |
| `ALGOLIA_APPLICATION_ID_2`   | `UCX3XB3SH4`             |
| `ALGOLIA_ADMIN_KEY_2`        | (Personify `UCX3XB3SH4`) |
| `ALGOLIA_APPLICATION_ID_MCM` | `5QZOBPRNH0`             |
| `ALGOLIA_ADMIN_KEY_MCM`      | (Personify `5QZOBPRNH0`) |


Application `NOCTT5TZUU` must be used for all the tests except the MCM tests
that needs to be done using application `5QZOBPRNH0` for which MCM has been
enabled. The second regular application `UCX3XB3SH4` is used to test the
`AccountClient` features which are performing cross-application operations.

### Index naming convention

For each of the following test, one or more index may be used. In order to
avoid tests to compromise each other when running in parallel, the following
naming convention should be used:

`LANG_DATE_TIME_INSTANCE_TEST`

Where:
* `LANG` corresponds to the API client language or integration name (go, php,
  java, ruby, rails, etc.) which is unique for each client/integration.
* `DATE` corresponds to the current time according to the following format:
  * `YYYY-MM-DD`
* `TIME` corresponds to the current time according to the following format:
  * `HH:mm:ss`
* `INSTANCE` corresponds to
  * the `TRAVIS_JOB_NUMBER` environment variable if available, otherwise
  * the current username on the system running the test if available, otherwise "unknown"
* `TEST` is the name of the current test

Note that you need to make sure your code is using UTC timezone.

Let’s take some examples: if I, Anthony, was run the "indexing" test on my
machine, as of today (September, 11th 2018, 6:05:39 PM) for the Go client, the
name of the index that would be created should be:

`go_2018-09-11_18:05:39_anthony_indexing`

This naming enables us to run tests in parallel, across different environments
(local computer, CI pipelines, etc.) without impacting other tests.

On top of that, the test suite of each client should include a cleaning
function that takes care of removing all indices older than one day (i.e.
whose DATE time does not correspond to the current day) for the current API
client/integration. This procedure ensure that we are not leaving undeleted
indices in each account. This is a two-step process where we first delete the
old non-replica indices and then the replica ones.

### Tests (index)

#### Indexing

* Instantiate the client and index `indexing`
* Add 1 record with **saveObject** with an objectID and collect taskID/objectID
* Add 1 record with **saveObject** without an objectID and collect taskID/objectID
* Add 2 records with **saveObjects** with an objectID and collect taskID/objectID
* Add 2 records with **saveObjects** without an objectID and collect taskID/objectID
* Sequentially send 10 batches of 100 objects with objectID from 1 to 1000 with **batch** and collect taskIDs/objectIDs
* Wait for all collected tasks to terminate with **waitTask**
* Retrieve the 6 first records with **getObject** and check their content against original records
* Retrieve the 1000 remaining records with **getObjects** with objectIDs from 1 to 1000 and check their content against original records
* Browse all records with **browseObjects** and make sure we have browsed 1006 records, and check that all objectIDs are found
* Alter 1 record with **partialUpdateObject** and collect taskID/objectID
* Alter 2 records with **partialUpdateObjects** and collect taskID/objectID
* Wait for all collected tasks to terminate with **waitTask**
* Retrieve all the previously altered records with **getObject** and check their content against the modified records
* Delete the first record with **deleteObject** and collect taskID
* Delete the 5 remaining first records with **deleteObjects** and collect taskID
* Delete the 1000 remaining records with **clearObjects** and collect taskID
* Wait for all collected tasks to terminate
* Browse all objects with **browseObjects** and make sure that no records are returned

#### Settings

* Instantiate the client and index `settings` and keep the generated index name (which will be used as a prefix of the replica indices)
* Add one record to create the index with **saveObject**
* Set the settings with the following parameters with **setSettings** and collect the taskID

| Setting                             | Value                                                                                                                       |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `searchableAttributes`              | `[ "attribute1", "attribute2", "attribute3", "ordered(attribute4)", "unordered(attribute5)" ]`                              |
| `attributesForFaceting`             | `[ "attribute1", "filterOnly(attribute2)", "searchable(attribute3)" ]`                                                      |
| `unretrievableAttributes`           | `[ "attribute1", "attribute2" ]`                                                                                            |
| `attributesToRetrieve`              | `[ "attribute3", "attribute4" ]`                                                                                            |
| `ranking`                           | `[ "asc(attribute1)", "desc(attribute2)", "attribute", "custom", "exact", "filters", "geo", "proximity", "typo", "words" ]` |
| `customRanking`                     | `[ "asc(attribute1)", "desc(attribute1)" ]`                                                                                 |
| `replicas`                          | `[ indexname + "_replica1", indexname + "_replica2" ]`                                                                      |
| `maxValuesPerFacet`                 | `100`                                                                                                                       |
| `sortFacetValuesBy`                 | `"count"`                                                                                                                   |
| `attributesToHighlight`             | `[ "attribute1", "attribute2" ]`                                                                                            |
| `attributesToSnippet`               | `[ "attribute1:10", "attribute2:8" ]`                                                                                       |
| `highlightPreTag`                   | ``"<strong>"``                                                                                                              |
| `highlightPostTag`                  | ``"</strong>"``                                                                                                             |
| `snippetEllipsisText`               | ``" and so on."``                                                                                                           |
| `restrictHighlightAndSnippetArrays` | `true`                                                                                                                      |
| `hitsPerPage`                       | `42`                                                                                                                        |
| `paginationLimitedTo`               | `43`                                                                                                                        |
| `minWordSizefor1Typo`               | `2`                                                                                                                         |
| `minWordSizefor2Typos`              | `6`                                                                                                                         |
| `typoTolerance`                     | `false`                                                                                                                     |
| `allowTyposOnNumericTokens`         | `false`                                                                                                                     |
| `ignorePlurals`                     | `true`                                                                                                                      |
| `disableTypoToleranceOnAttributes`  | `[ "attribute1", "attribute2" ]`                                                                                            |
| `disableTypoToleranceOnWords`       | `[ "word1", "word2" ]`                                                                                                      |
| `separatorsToIndex`                 | `"()[]"`                                                                                                                    |
| `queryType`                         | `"prefixNone"`                                                                                                              |
| `removeWordsIfNoResults`            | `"allOptional"`                                                                                                             |
| `advancedSyntax`                    | `true`                                                                                                                      |
| `optionalWords`                     | `[ "word1", "word2" ]`                                                                                                      |
| `removeStopWords`                   | `true`                                                                                                                      |
| `disablePrefixOnAttributes`         | `[ "attribute1", "attribute2" ]`                                                                                            |
| `disableExactOnAttributes`          | `[ "attribute1", "attribute2" ]`                                                                                            |
| `exactOnSingleWordQuery`            | `"word"`                                                                                                                    |
| `enableRules`                       | `false`                                                                                                                     |
| `numericAttributesForFiltering`     | `[ "attribute1", "attribute2" ]`                                                                                            |
| `allowCompressionOfIntegerArray`    | `true`                                                                                                                      |
| `attributeForDistinct`              | `"attribute1"`                                                                                                              |
| `distinct`                          | `2`                                                                                                                         |
| `replaceSynonymsInHighlight`        | `false`                                                                                                                     |
| `minProximity`                      | `7`                                                                                                                         |
| `responseFields`                    | `[ "hits", "hitsPerPage" ]`                                                                                                 |
| `maxFacetHits`                      | `100`                                                                                                                       |
| `camelCaseAttributes`               | `[ "attribute1", "attribute2" ]`                                                                                            |
| `decompoundedAttributes`            | `{ "de": ["attribute1", "attribute2"], "fi": ["attribute3"] }`                                                              |
| `keepDiacriticsOnCharacters`        | `"øé"`                                                                                                                      |

* Wait for the collected tasks to terminate with **waitTask**
* Get the settings with **getSettings** and check that they correspond to the ones that were inserted
* Set the settings with the following parameters with **setSettings** and collect the taskID

| Setting           | Value          |
| ----------------- | -------------- |
| `typoTolerance`   | `"min"`        |
| `ignorePlurals`   | `["en", "fr"]` |
| `removeStopWords` | `["en", "fr"]` |
| `distinct`        | `true`         |

* Wait for the collected tasks to terminate with **waitTask**
* Get the settings with **getSettings** and check that they correspond to the ones that were inserted and overridden

#### Search

* Instantiate the client and index `search`
* Add the following records with **saveObjects** and collect the taskID

```
[
  {"company": "Algolia", "name": "Julien Lemoine"},
  {"company": "Algolia", "name": "Nicolas Dessaigne"},
  {"company": "Amazon", "name": "Jeff Bezos"},
  {"company": "Apple", "name": "Steve Jobs"},
  {"company": "Apple", "name": "Steve Wozniak"},
  {"company": "Arista Networks", "name": "Jayshree Ullal"},
  {"company": "Google", "name": "Larry Page"},
  {"company": "Google", "name": "Rob Pike"},
  {"company": "Google", "name": "Serguey Brin"},
  {"company": "Microsoft", "name": "Bill Gates"},
  {"company": "SpaceX", "name": "Elon Musk"},
  {"company": "Tesla", "name": "Elon Musk"},
  {"company": "Yahoo", "name": "Marissa Mayer"}
]
```

* Set the following settings with **setSettings** and collect the taskID

| Setting                 | Value                     |
| ----------------------- | ------------------------- |
| `attributesForFaceting` | `["searchable(company)"]` |

* Wait for the collected tasks to terminate using **waitTask**
* Perform a search query using **search** with the query `"algolia"` and no parameter and check that the number of returned hits is equal to 2
* Perform a search using **search** with the query `"elon"` and the following parameter and check that the queryID field from the response is not empty

| Parameter        | Value  |
| ---------------- | ------ |
| `clickAnalytics` | `true` |

* Perform a faceted search using **search** with the query `"elon"` and the following parameters and check that the number of returned hits is equal to 1

| Parameter      | Value             |
| -------------- | ----------------- |
| `facets"`      | `"*"`             |
| `facetFilters` | `"company:tesla"` |

* Perform a filtered search using **search** with the query `"elon"` and the following parameters and check that the number of returned hits is equal to 2

| Parameter | Value                                 |
| --------- | ------------------------------------- |
| `facets"` | `"*"`                                 |
| `filters` | `"(company:tesla OR company:spacex)"` |

* Perform a facet search using **searchForFacetValue** with the facet `"company"` and the query `"a"` and no parameter and check that the facetHits field from the response contains the following values, in any order:
  * `"Algolia"`
  * `"Amazon"`
  * `"Apple"`
  * `"Arista Networks"`

#### Synonyms

* Instantiate the client and index `synonyms`
* Add the following records using **saveObjects** and collect the taskID

```
[
  {"console": "Sony PlayStation <PLAYSTATIONVERSION>"},
  {"console": "Nintendo Switch"},
  {"console": "Nintendo Wii U"},
  {"console": "Nintendo Game Boy Advance"},
  {"console": "Microsoft Xbox"},
  {"console": "Microsoft Xbox 360"},
  {"console": "Microsoft Xbox One"}
]
```

* Add the following regular (n-way) synonym using **saveSynonym** and collect the taskID

| ObjectID | Synonym                                            |
| -------- | -------------------------------------------------- |
| `"gba"`  | `[ "gba", "gameboy advance", "game boy advance" ]` |

* Add the following synonyms using **saveSynonyms** and collect the taskID

| ObjectID                            | Kind                             | Synonym                                                                                      |
| ----------------------------------- | -------------------------------- | -------------------------------------------------------------------------------------------- |
| `"wii_to_wii_u"`                    | One way synonym                  | `"wii"` → `["wii U"]`                                                                        |
| `"playstation_version_placeholder"` | Placeholder synonym              | Placeholder: `"<PLAYSTATIONVERSION>"` Replacements: `[ "1", "One", "2", "3", "4", "4 Pro" ]` |
| `"ps4"`                             | Alternative correction (1 typo)  | `"ps4"` → `["playstation4"]`                                                                 |
| `"psone"`                           | Alternative correction (2 typos) | `"psone"` → `["playstationone"]`                                                             |

* Wait for the collected tasks to terminate using **waitTask**
* Retrieve the 5 added synonyms with **getSynonym** and check they are correctly retrieved
* Perform a search query using **searchSynonyms** with the empty and check that the number of returned hits is equal to 5
* Using **browseSynonyms** and iterate over all the synonyms and check that those collected synonyms are the same as the 5 originally saved
* Delete the synonym with objectID `"gba"` using **deleteSynonym** and wait for the task to terminate using waitTask with the returned taskID
* Try to get the synonym with **getSynonym** with objectID `"gba"` and check that the synonym does not exist anymore
* Clear all the synonyms using **clearSynonyms** and wait for the task to terminate using waitTask with the returned taskID
* Perform a synonym search using **searchSynonyms** with an empty query and check that the number of returned synonyms is equal to 0

#### Query rules

* Instantiate the client and index `rules`
* Add the following records using **saveObjects** and collect the taskID

```
[
  {"objectID": "iphone_7", "brand": "Apple", "model": "7"},
  {"objectID": "iphone_8", "brand": "Apple", "model": "8"},
  {"objectID": "iphone_x", "brand": "Apple", "model": "X"},
  {"objectID": "one_plus_one", "brand": "OnePlus", "model": "One"},
  {"objectID": "one_plus_two", "brand": "OnePlus", "model": "Two"},
]
```

* Set **attributesForFaceting** to `["brand"]` using **setSettings** and collect the taskID
* Save the following rule using **saveRule** and collect the taskID

```
{
  "objectID": "brand_automatic_faceting",
  "enabled": false,
  "condition": {"anchoring": "is", "pattern": "{facet:brand}"},
  "consequence": {
    "params": {
      "automaticFacetFilters": [
        {"facet": "brand", "disjunctive": true, "score": 42},
      ]
    }
  },
  "validity": [
    {
      "from": 1532439300, // 07/24/2018 13:35:00 UTC
      "until": 1532525700 // 07/25/2018 13:35:00 UTC
    },
    {
      "from": 1532612100, // 07/26/2018 13:35:00 UTC
      "until": 1532698500 // 07/27/2018 13:35:00 UTC
    }
  ],
  "description": "Automatic apply the faceting on `brand` if a brand value is found in the query"
}
```

* Save the following rules using **batchRules** and collect the taskID

```
[
  {
    "objectID": "query_edits",
    "condition": {"anchoring": "is", "pattern": "mobile phone"},
    "consequence": {
      "params": {
        "query": {
          "edits": [
            {"type": "remove", "delete": "mobile"},
            {"type": "replace", "delete": "phone", "insert": "iphone"},
          ]
        }
      }
    }
  }
]
```

* Wait for the previous tasks to complete using **waitTask**
* Retrieve all the rules using **getRule** and check that they were correctly saved
* Retrieve all the rules using **searchRules** with {query: ""} and check that they were correctly saved
* Iterate over all the rules using **ruleIterator** and check that they were correctly saved
* Delete the first rule using **deleteRule** and check that it was correctly deleted using a **getRule**.
* Clear all the remaining rules using **clearRules** and check that all rules have been correctly removed using **searchRules** with an empty query
* As an extra test, to make sure we do support deserialization of Query Rule v1 format, try to deserialize the following JSON to the Query Rule structure or class (depending on the language) to ensure that pre-v2 query rules are still correctly deserialized and handled:

```
{
  "objectID": "query_edits",
  "condition": {"anchoring": "is", "pattern": "mobile phone"},
  "consequence": {
    "params": {
      "query": {
       "remove": ["mobile", "phone"]
      }
    }
  }
}
```

#### Batching

* Instantiate the client and index `index_batching`
* Add the following objects using **saveObjects** and wait for its completion using **waitTask**

```
[
  {"objectID": "one", "key": "value"},
  {"objectID": "two", "key": "value"},
  {"objectID": "three", "key": "value"},
  {"objectID": "four", "key": "value"},
  {"objectID": "five", "key": "value"},
]
```

* Send the following batch using **batch** and wait for its completion using **waitTask**

```
[
  {"action": "addObject", "body": {"objectID": "zero", "key": "value"}},
  {"action": "updateObject", "body": {"objectID": "one", "k": "v"}},
  {"action": "partialUpdateObject", "body": {"objectID": "two", "k": "v"}},
  {"action": "partialUpdateObject", "body": {"objectID": "two_bis", "key": "value"}},
  {"action": "partialUpdateObjectNoCreate", "body": {"objectID": "three", "k": "v"}},
  {"action": "deleteObject", "body": {"objectID": "four"}}
]
```

* Browse the entire index using **browseObjects** and check that it exactly contains the following objects

```
[
  {"objectID": "zero", "key": "value"},
  {"objectID": "one", "k": "v"},
  {"objectID": "two", "key": "value", "k": "v"},
  {"objectID": "two_bis", "key": "value"},
  {"objectID": "three", "key": "value", "k": "v"},
  {"objectID": "five", "key": "value"},
]
```

#### Replacing

* Instantiate the client and index `replacing`
* Add the following record to the index using **saveObject** and collect the taskID

```
{"objectID": "one"}
```

* Add the following rule to the index using **saveRule** and collect the taskID

```
{
  "objectID": "one",
  "condition": { "anchoring": "is", "pattern": "pattern"},
  "consequence": {
    "params": {
      "query": {
        "edits": [
          {"type": "remove", "delete": "pattern"}
        ]
      }
    }
  }
}
```

* Add the following synonym to the index using **saveSynonym** and collect the taskID

```
{"objectID": "one", "type": "synonym", "synonyms": ["one", "two"]}
```

* Wait for all the previous tasks to terminate using **waitTask**
* Replace the existing object with the following one using **replaceAllObjects** and collect the taskID

```
{"objectID": "two"}
```

* Replace the existing rule with the following one using **replaceAllRules** and collect the taskID

```
{
  "objectID": "two",
  "condition": { "anchoring": "is", "pattern": "pattern"},
  "consequence": {
    "params": {
      "query": {
        "edits": [
          {"type": "remove", "delete": "pattern"}
        ]
      }
    }
  }
}
```

* Replace the existing synonym with the following one using **replaceAllSynonyms** and collect the taskID

```
{"objectID": "two", "type": "synonym", "synonyms": ["one", "two"]}
```

* Wait for all the previous tasks to terminate using **waitTask**
* Check that record with `objectID="one"` doesn’t exist using **getObject**
* Check that record with `objectID="two"` does exist using **getObject**
* Check that rule with `objectID="one"` doesn’t exist using **getRule**
* Check that rule with `objectID="two"` does exist using **getRule**
* Check that synonym with `objectID="one"` doesn’t exist using **getSynonym**
* Check that synonym with `objectID="two"` does exist using **getSynonym**

---

### Tests (client)

#### Copy index

* Instantiate the client and index `copy_index` and keep the generated index name (which will be used as a prefix of the copied/moved indices)
* Add the following records with **saveObjects** and collect the taskID

```
[
  {"objectID": "one", "company": "apple"},
  {"objectID": "two", "company": "algolia"}
]
```

* Set the following settings with **setSettings** and collect the taskID

```
{
    "attributesForFaceting": ["company"]
}
```

* Add the following synonym with **saveSynonym** and collect the taskID

```
{
    "objectID": "google_placeholder",
    "type": "placeholder",
    "placeholder": "<GOOG>",
    "replacements": ["Google", "GOOG"]
}
```

* Add the following query rule with **saveRule** and collect the taskID

```
{
  "objectID":  "company_auto_faceting",
    "condition": {
      "anchoring":"contains",
      "pattern":"{facet:company}",
    },
    "consequence": {
      "params": {"automaticFacetFilters": ["company"]}
    }
}
```

* Wait for the collected tasks to terminate with **waitTask**
* Copy `copy_index` index' settings to a new `copy_index_settings` index using **copySettings** and collect the taskID
* Copy `copy_index` index' rules to a new `copy_index_rules` index using **copyRules** and collect the taskID
* Copy `copy_index` index' synonyms to a new `copy_index_synonyms` index using **copySynonyms** and collect the taskID
* Fully copy `copy_index` index into `copy_index_full_copy` with **copyIndex** and collect taskID
* Wait for the collected tasks to terminate with **waitTask**
* Check that `copy_index_settings` only contains the same settings as the original index with **getSettings**
* Check that `copy_index_rules` only contains the same rules as the original index with **getRule**
* Check that `copy_index_synonyms` only contains the same synonyms as the original index with **getSynonym**
* Check that `copy_index_full_copy` contains both the same settings, rules and synonyms as the original index with **getSettings**, **getRule** and **getSynonym**

#### Multi Cluster Management (MCM)

* Instantiate a new client with the MCM testing application `5QZOBPRNH0` (shared between all API client testing apps)
* Retrieve all the clusters with **listClusters**
* Check that there are at least 2 clusters in the list
* Assign the following userID with **assignUserID** with the first cluster from the list we just retrieved:

```
LANG-DATE-TIME-INSTANCE

Where:
 * LANG is the API client language (e.g. scala/go/php/etc.)
 * DATE corresponds to the current time according to the following format: YYYY-MM-DD
 * TIME corresponds to the current time according to the following format: HH-mm-ss
 * INSTANCE corresponds to:
   * the TRAVIS_JOB_NUMBER environment variable if available, otherwise
   * the current username on the system running the test if available, otherwise
   * "unknown"

For instance:  python-2019-01-01-10-10-05-unknown
```

* Loop until the assigned userID is found using **getUserID**
* Search of the assigned userID using **searchUserIDs**
* List all userIDs using **listUserIDs** and check that the result set is not empty
* Retrieve the top10 userIDs using **getTopUserIDs** and check that the result set is not empty
* Remove the assigned userID using **removeUserID** (loop until the call succeeds, this is the only way to make this work)
* Loop until the removed userID cannot be found anymore using **getUserID** (loop until the call succeeds, this is the only way to make this work)
* List all the userIDs using **listUserIDs** and and collect all the ones with the following prefix:

```
LANG-DATE

Where:
 * LANG is the API client language (e.g. scala/go/php/etc.)
 * DATE corresponds to a date following to the following format YYYY-MM-DD but which is not today.

For example, if today’s date is November 28th, 2018 and the API client is Go, we should collect userIDs with prefixes "go-2018-11-26", "go-2018-11-27" but not "php-2018-11-26" nor "go-2018-11-28".
```

* Delete all the collected userIDs using **removeUserID**

#### API keys

* Instantiate the client
* Create a new API key using **addAPIKey** with the following ACL and parameters and collect the returned key value (its ID actually)

| Parameter | Value                                                                                                                                                                                                 |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| acl       | ["search"]                                                                                                                                                                                            |
| params    | { "description": "A description", "indexes": "index"], "maxHitsPerQuery": 1000, "maxQueriesPerIPPerHour": 1000, "queryParameters": "typoTolerance=strict", "referers": ["referer"], "validity": 600 } |

* Optionally, defer the deletion of the key using **deleteAPIKey** at the end of your test (using tearDown/afterTest-like methods) if possible with the testing framework of the language. This extra step guarantee that if the test fail before step 8., the inserted key will still be removed.
* Retrieve the added key using **getAPIKey** with the key value that was collected earlier (loop until the call succeeds, this is the only way to make this work) and check that the ACL and params do match (expect for `createdAt` field that was generated and `validity` that may have decreased since the key was inserted)
* List all the API keys using **listAPIKeys** and check that the added key is found as well
* Update the `maxHitsPerQuery` field to 42 of the key using **updateAPIKey**
* Retrieve the added key using **getAPIKey** with the key value that was collected earlier (loop until the call succeeds, this is the only way to make this work) and check that the `maxHitsPerQuery` field of the returned key has indeed changed
* Remove the key using **deleteAPIKey**
* Loop until the removed key cannot be found anymore using **getAPIKey** (loop until the call succeeds, this is the only way to make this work)
* Restore the previous deleted key using **restoreApiKey** method.
* Retrieve the restored key using **getAPIKey** with the key value that was collected earlier (loop until the call succeeds, this is the only way to make this work)
* Remove the key using **deleteAPIKey**

#### Get logs

* Instantiate the client
* Perform 2 times a **listIndices** operation to ensure at least two logs will be available from the application
* Retrieve the logs with the following parameters using **getLogs**:

```
{
  "length": 2,
  "offset": 0,
  "type":   "all"
}
```

* Ensure that the length of the retrieved logs is equal to 2

#### Multiple Operations

* Instantiate the client and index1 `multiple_operations` and index2 `multiple_operations_dev`
* Add the following objects using **multipleBatch** to the appropriate index and collect the return objectIDs and taskIDs per index:

```
[
  {"indexName": indexName1, "action": "addObject", "body": {"firstname": "Jimmie"}},
  {"indexName": indexName1, "action": "addObject", "body": {"firstname": "Jimmie"}},
  {"indexName": indexName2, "action": "addObject", "body": {"firstname": "Jimmie"}},
  {"indexName": indexName2, "action": "addObject", "body": {"firstname": "Jimmie"}}
]
```

* For all the collected taskIDs per index, wait on them using the **waitTask** operation of the appropriate index
* Retrieve all the inserted objects using **multipleGetObjects** from their respective index (the collected objectIDs are in order, meaning that they correspond to the order they were inserted into their respective indices during the **multipleBatch**) and ensure the 4 objects are correctly returned.
* Perform the two following search operations using **multipleQueries** with strategy="none" and ensure that the response contains two results, both containing two hits:

```
[
  {"indexName": indexName1, "params": {"query": "", "hitsPerPage": 2}},
  {"indexName": indexName2, "params": {"query": "", "hitsPerPage": 2}}
]
```

* Perform the two following search operations using **multipleQueries** with `strategy="stopIfEnoughMatches"` and ensure that the response contains two results, the first one containing two hits and the second none:

```
[
  {"indexName": indexName1, "params": {"query": "", "hitsPerPage": 2}},
  {"indexName": indexName2, "params": {"query": "", "hitsPerPage": 2}}
]
```

#### DNS timeout

* Instantiate the client with the regular credentials but provides the following host (in that order):

```
[
  "algolia.biz",
  appID + "-1.algolianet.com",
  appID + "-2.algolianet.com",
  appID + "-3.algolianet.com"
]
```

* Start a timer
* Perform 10 sequential calls to Algolia using the client’s **listIndices** methods and check that no error happened at each call
* Stop the timer
* Check that the timer’s delta is lower than 5 seconds

#### Personalization Strategy

* Instantiate the client
* Get the personalization strategy using the **getPersonalizationStrategy** method and check if the `taskID` key
is inside the api response
* Optionally, and at unit level, set the personalization strategy below using the **setPersonalizationStrategy** and make sure a POST to `1/recommendation/personalization/strategy` was made containing the following body:
```
{
    'eventsScoring': {
        'Add to cart': {'score': 50, 'type': 'conversion'},
        'Purchase': {'score': 100, 'type': 'conversion'}
    },
    'facetsScoring': {
        'brand': {'score': 100},
        'categories': {'score': 10}
    }
}
```
---

### Tests (account)

#### Copy index

* Instantiate the client1 and index1 `copy_index` (for app `NOCTT5TZUU`)
* Instantiate the client2 and index2 `copy_index_2` (for app `NOCTT5TZUU`)
* Try to perform an **Account.CopyIndex** from index1 to index2 and make sure that it fails because the application ID is the same
* Re-instantiate the client2 and index2 `copy_index` (for app `UCX3XB3SH4`)
* Add the following record to index1 using **saveObject** and collect the taskID

```
{"objectID": "one"}
```

* Add the following rule to index1 using **saveRule** and collect the taskID

```
{
  "objectID": "one",
  "condition": { "anchoring": "is", "pattern": "pattern"},
  "consequence": {
    "params": {
      "query": {
        "edits": [
          {"type": "remove", "delete": "pattern"}
        ]
      }
    }
  }
}
```

* Add the following synonym to index1 using **saveSynonym** and collect the taskID

```
{"objectID": "one", "type": "synonym", "synonyms": ["one", "two"]}
```

* Set the following settings to index1 using **setSettings** and collect the taskID

```
{"searchableAttributes": ["objectID"]}
```

* Wait for all the previous tasks to terminate using **waitTask**
* Perform a cross-application index copy using **accountClient.copyIndex** from index1 to index2
* Check that record with `objectID="one"` is present on index2 using **getObject**
* Check that rule with `objectID="one"` is present on index2 using **getRule**
* Check that synonym with `objectID="one"` is present on index2 using **getSynonym**
* Check that `searchableAttributes` setting is set to `["objectID"]` on index2 using **getSettings**
* Try to perform an **accountClient.copyIndex** from index1 to index2 and make sure that it fails because the destination index already exists

---

### Tests (secured API keys)

#### Generate secured API keys

* Instantiate the client and index1 `secured_api_keys`
* Instantiate index2 with name `secured_api_keys_dev`
* Add the following object to both indices using **saveObject** and wait for both indexing operations to terminate using **waitTask**:

```
{"objectID": "one"}
```

* Generate a new secured API key using **generateSecuredAPIKey** with the `ALGOLIA_SEARCH_KEY_1` as source key and the following parameters:

```
{
  "validUntil": now + 10 min (in epoch seconds),
  "restrictIndices": indexName1
}
```

* Instantiate a new client with the generated key
* Perform an empty search operation to index1 using **search** and check that no error happens
* Perform an empty search operation to index2 using **search** and check that an error do happen

---

### Tests (analytics)

#### AB testing

* Instantiate the client and index1 `ab_testing` and index2 `ab_testing_dev`
* Instantiate the analytics client
* Iterate over all the existing AB tests whose name matches the following prefix, using **getABTests**, and collect their abTestID:

```
LANG-DATE-

Where:
 * LANG corresponds to the current programming language
 * DATE corresponds to any date BUT today (this way, current day tests are not removing each other)
```

* Remove all the AB tests thanks to the collected abTestIDs using **deleteABTest**
* Add the following dummy object to index1 using **saveObject** and wait for this task to complete using the **waitTask** method of index1:

```
{"objectID": "one"}
```

* Add the following dummy object to index2 using **saveObject** and wait for this task to complete using the **waitTask** method of index2:

```
{"objectID": "one"}
```

* Add the following AB test using **addABTest**, collect the returned abTestID from the response and wait for this task to complete using the **waitTask** method of the analytics client:

```
{
  "name": abTestName,
  "variants": [
    {"index": indexName1, "trafficPercentage": 60, "description": "a description"},
    {"index": indexName2, "trafficPercentage": 40}
  ],
  "endAt": now + 24h
}

Where:
 * abTestName corresponds to the LANG-DATE-TIME-INSTANCE we already used previously (please refer to other sections of this document)
 * indexName1 and indexName2 correspond to the respective names of the generated indices
 * now + 24h corresponds to tomorrow’s time at the exact same time (if the API client doesn’t support date/time automatic serialization, the expected format is ISO8601 which roughly corresponds to something like "2006-01-02T15:04:05Z" as a string.
```

* Retrieve the added AB test using **getABTest** thanks to its collected abTestID and check that the retrieved AB test's `name`, `endAt` and variants’ `index`, `trafficPercentage` and `description` fields correspond to the ones of the original AB test (other fields are generated so you shouldn’t have to check them). Finally check that the `status` field of the retrieved AB test is not `stopped`.
* Iterate over all the existing AB tests using **getABTests** and ensure that the added AB test is also found there, using the same comparison as in the previous step
* Stop the AB test using **stopABTest** and wait for this task to complete using the **waitTask** method of the analytics client.
* Retrieve the AB test using **getABTest** and ensure that the `status` field of the AB test is now set to `stopped`
* Remove the AB test using **deleteABTest** and wait for this task to complete using the **waitTask** method of the analytics client
* Ensure that the AB test cannot be retrieved anymore using **getABTest** by expecting an error


#### AA testing

* Instantiate the client and index `aa_testing`
* Instantiate the analytics client
* Iterate over all the existing AB tests whose name matches the following prefix, using **getABTests**, and collect their abTestID

```
LANG-DATE-

Where:
 * LANG corresponds to the current programming language
 * DATE corresponds to any date BUT today (this way, current day tests are not removing each other)
```

* Remove all the AB tests thanks to the collected abTestIDs using **deleteABTest**
* Add the following dummy object to the index using **saveObject** and wait for this task to complete using the **waitTask** method

```
{"objectID": "one"}
```

* Add the following AB test using **addABTest**, collect the returned abTestID from the response and wait for this task to complete using the **waitTask** method of the analytics client:

```
{
  "name": abTestName,
  "variants": [
    {"index": indexName, "trafficPercentage": 90},
    {"index": indexName, "trafficPercentage": 10, "customSearchParameters": {"ignorePlurals": true}}
  ],
  "endAt": now + 24h
}

Where:
 * abTestName corresponds to the LANG-DATE-TIME-INSTANCE we already used previously (please refer to other sections of this document)
 * indexName correspond to the name of the generated index
 * now + 24h corresponds to tomorrow’s time at the exact same time (if the API client doesn’t support date/time automatic serialization, the expected format is ISO8601 which roughly corresponds to something like "2006-01-02T15:04:05Z" as a string.
```

* Retrieve the added AB test using **getABTest** thanks to its collected abTestID and check that the retrieved AB test’s `name`, `endAt` and variants’ `index`, `trafficPercentage` and `customSearchParameters` fields correspond to the ones of the original AB test (other fields are generated so you shouldn’t have to check them). Finally check that the `status` field of the retrieved AB test is not `stopped`.

---

### Tests (backward compatibility)

The following tests are a bit special in the sense that they could not be
implemented for every API client as of today. Our API clients need to reflect
the changes of the actual Algolia REST API. However, we do not want to ask our
users to cope with breaking changes every time they update their clients. This
is why we implement breaking changes in a forward compatible manner. Those
tests are there to make sure we hide the complexity of non-breaking-compatible
changes to the final users while still offering the very last features, as
tested by the rest of the Common Test Suite.

#### Old settings

Try to deserialize the following JSON string of settings into the setting
representation of the API client and ensure they map to the new correct name:

The following JSON string representation of a settings map:

```
{
  "attributesToIndex":        ["attr1", "attr2"],
  "numericAttributesToIndex": ["attr1", "attr2"],
  "slaves":                   ["index1", "index2"]
}
```

Should deserialize to a settings object like:

```
Settings {
  SearchableAttributes           = ["attr1", "attr2"],
  NumericAttributesForFiltering  = ["attr1", "attr2"],
  Replicas                       = ["index1", "index2"],
}
```


#### Query rules v1

The following JSON string representation of a query rule v1:

```
{
  "objectID": "query_edits",
  "condition": {"anchoring": "is", "pattern": "mobile phone"},
  "consequence": {
    "params": {
      "query": {
       "remove": ["mobile", "phone"]
      }
    }
  }
}
```

Should deserialize to an query rule v2 object like:

```
Rule {
  ObjectID  = "query_edits",
  Condition = {
    Anchoring = "is",
    Pattern   = "mobile phone",
  },
  Consequence = {
    Params = {
      Query = {
        Edits = [
          Edit{Type="remove", Delete="mobile"},
          Edit{Type="remove", Delete="phone"},
        ]
      }
    }
  }
}
```
