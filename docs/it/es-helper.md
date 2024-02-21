# ESHelper

---

## makeSearch

The `makeSearch` function is designed to perform a search operation in an Elasticsearch index. It encapsulates the process of initializing an Elasticsearch client, connecting to a given Elasticsearch cluster URI, and executing a search with a specified query. The function handles asynchronous execution through the use of Promises, ensuring that the caller can manage the result or error through promise resolution or rejection. This design pattern promotes cleaner, more readable asynchronous code when working with Elasticsearch in a JavaScript environment.

###  Input Parameters

| Parameter     | Type   | Description                                                           |
|---------------|--------|-----------------------------------------------------------------------|
| `index`       | string | The name of the Elasticsearch index to search.                        |
| `body`        | string | The search query or request body in Elasticsearch's Query DSL.        |
| `Client`      | any    | The Elasticsearch client class for connecting and executing searches. |
| `ELASTIC_URI` | string | The Elasticsearch cluster's URI to which the client will connect.     |

###  Expected Output

The function is expected to return a `Promise<SearchResponse<any>>`. Once the promise is resolved, it will yield a search response from Elasticsearch, which contains the search results and other related information.

###  Explanation of Single Components

- **Promise Construction**: The function returns a new `Promise` object, which is used for asynchronous computation. The promise has two parameters, `resolve` and `reject`, which are functions to be called when the promise is respectively resolved or rejected.

- **Elasticsearch Client Initialization**: Inside the promise, a new instance of the `Client` is created with the `ELASTIC_URI` to establish a connection to the Elasticsearch cluster.

- **Search Operation**: The `client.search` method is called with an object containing the `index` and the search `body`. This method performs the search operation in Elasticsearch.

- **Callback Function**: The callback function provided to `client.search` handles the response or error. If an error occurs (`err` is truthy), it logs the error using `console.log(err)`. If the operation is successful, it resolves the promise with the `body` of the response, which contains the search results.

---

## buildQuery

The `buildQuery` function constructs a complex Elasticsearch query object based on a set of input parameters and a configuration object. It dynamically adapts the query structure to accommodate various types of searches, such as facet searches or full-text searches, by interpreting the provided `data` and `conf`. The function handles sorting, pagination, source field filtering, and aggregations, making it a versatile tool for generating customized queries for Elasticsearch. This flexibility allows for the efficient retrieval of search results tailored to specific application requirements, enhancing the search functionality of the application it supports.

###  Input Parameters

| Parameter | Type     | Description                                                                                                                                                               |
|-----------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `data`    | DataType | An object containing `searchId` and `results`, among other possible fields. `searchId` indicates the type of search, and `results` contains search results or parameters. |
| `conf`    | any      | Configuration object that provides mappings and settings for different search IDs, including base queries, sorting configurations, and other options.                     |
| `type`    | string   | A string indicating the type of query, such as 'facets' for facet searches or other types for different search requirements.                                              |

###  Expected Output

The function returns an object (`main_query`) structured as an Elasticsearch query. This object includes the main query parameters (`query`), sorting instructions (`sort`), pagination controls (`size` and `from`), source filtering (`_source`), and aggregations (`aggregations`). The exact structure and content of this object depend on the input `data`, the `conf` configuration, and the `type` of the query.

###  Explanation of Single Components

- **Data Unpacking**: The function starts by extracting `searchId` and `results` from the `data` object. `searchId` is used to identify the specific configuration in `conf`, and `results` contains the current search results or parameters.

- **Sort Configuration**: The sort order is determined based on whether `results` is present. If `results` exists, its sort order is used; otherwise, the sort order is taken from the top level of `data`.

- **Elasticsearch Query Initialization**: Initializes the main query structure for Elasticsearch, setting up a boolean query and placeholders for sorting and aggregations.

- **Base Query Addition**: If a base query is defined in the configuration for the given `searchId`, it is added to the must conditions of the boolean query.

- **Sorting Logic**: Constructs the sorting part of the Elasticsearch query based on the configuration and the sort parameter extracted earlier. A default sort on `'slug.keyword'` is added as a fallback.

- **Pagination**: If the `type` is 'facets', no results are returned (`size` is set to 0). Otherwise, the size of the results is controlled by `limit`, and the starting point by `offset`.

- **Source Filtering**: Excludes or includes specific fields in the source of the Elasticsearch response based on the configuration.

- **Aggregations and Filters**: Dynamically builds the query based on the provided filters in `data` and the corresponding configurations in `conf`. This includes full-text search conditions, multi-value fields handling, and range queries. It also constructs aggregation queries if specified in the configuration.

## buildAggs

The `buildAggs` function is responsible for constructing the aggregation part of an Elasticsearch query based on the facets requested by the user and the configurations provided. It dynamically adapts to various types of aggregations, such as nested, range, and term aggregations, and incorporates additional configurations like sorting, filtering, and global aggregations. The function ensures that each facet's aggregation is tailored to the specific requirements of the search, enhancing the flexibility and precision of the search functionality. Through careful handling of configurations and requests, `buildAggs` plays a crucial role in generating detailed and customized aggregation queries for Elasticsearch, facilitating effective data analysis and insight generation based on the search results.

###  Input Parameters

| Parameter        | Type   | Description                                                                                                                                                                    |
|------------------|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `facets_request` | object | An object representing the request for facets, containing IDs, limits, offsets, and queries for each facet.                                                                    |
| `query_facets`   | object | An object containing configuration for each facet, such as sort order, whether to show empty facets, and specific settings for nested facets, ranges, and global aggregations. |

###  Expected Output

The function returns an object (`aggregations`) that is structured to be used as the aggregations part of an Elasticsearch query. This object will contain sub-objects for each facet requested, with configurations derived from `facets_request` and `query_facets`.

###  Explanation of Single Components

- **Initialization**: The function initializes `main_query` with an empty `aggregations` object.

- **Iteration over Facets**: It iterates over each facet in `facets_request`, constructing an aggregation query part for each based on the configurations provided in `query_facets`.

- **Aggregation Construction**: For each facet, depending on whether it's a nested facet, a range aggregation, or a standard term aggregation, different aggregation queries are constructed. This involves setting up terms for aggregation, including size (limit + offset), min document count, sort order, and any specific scripts for nested aggregations.

- **Nested Aggregations**: If the facet is nested and has specified nested fields, a nested aggregation query is built using the `buildNested` function.

- **Range Aggregations**: For facets defined with ranges, a range aggregation query is built.

- **Term Aggregations**: For standard term aggregations, the function constructs the aggregation part considering any additional filters (`filterTerm`), global aggregations, and extra configurations.

- **Error Handling and Backward Compatibility**: The function includes a comment about maintaining backward compatibility despite a known error in the case of missing nested fields for nested aggregations.

- **Filter Queries**: For facets that require filtering based on a term or a general filter, a filter query is constructed using the `buildAggsFilter` function.

- **Distinct Terms**: For term aggregations without sub-aggregations, a distinct terms aggregation is added to count unique terms.


## buildAggsFilter

The `buildAggsFilter` function is designed to construct filter queries for Elasticsearch aggregations based on specific terms and general filtering conditions defined in a facet's configuration. By utilizing helper functions (`ASHelper.queryString` and `ASHelper.queryBool`), it dynamically builds a boolean query that includes all necessary filter conditions. This allows for precise control over which data is included in the aggregation results, ensuring that the aggregations accurately reflect the user's filtering choices or predefined criteria. The ability to apply targeted filters to aggregations is essential for creating refined and relevant search experiences, particularly in applications where users need to drill down into data or explore specific subsets of information.

###  Input Parameters

| Parameter    | Type   | Description                                                                                                                                                                                         |
|--------------|--------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `filterTerm` | string | The term to filter the aggregations by. This is typically a user input or a selected filter value.                                                                                                  |
| `facet_conf` | object | The configuration object for the facet, which contains settings such as fields to apply the filter on (`innerFilterField`) and any general filters (`generalFilter`) that should always be applied. |

###  Expected Output

The function returns either an Elasticsearch query object that can be used to filter aggregations based on the provided `filterTerm` and `facet_conf`, or `null` if no filters are to be applied.

###  Explanation of Single Components

- **Initialization**: The function starts by initializing an empty `must_array`, which will hold the conditions that must be met for the filter to apply.

- **Filter Term Query**: If `filterTerm` is provided and not empty, a query string is created using `ASHelper.queryString`, targeting the fields specified by `innerFilterField` in `facet_conf`. This query is then added to `must_array`.

- **General Filter Query**: If `facet_conf` includes a `generalFilter`, another query string is created using `ASHelper.queryString` with the `generalFilter` configuration. This query is also added to `must_array`.

- **Boolean Query Construction**: If there are any conditions in `must_array`, a boolean query is constructed using `ASHelper.queryBool`, which combines all conditions in `must_array` under a `must` clause. The resulting boolean query object's `query` property is returned.

- **Return Value**: If no filters are to be applied (i.e., `must_array` is empty), the function returns `null`.


## buildNested

The `buildNested` function is designed to construct complex nested aggregations for Elasticsearch queries, supporting multiple levels of nesting and various additional configurations. It provides the capability to perform detailed analysis on nested document structures, allowing for the aggregation of terms based on specified fields within those nested documents. The function supports filtering of documents prior to aggregation, inclusion of extra fields in the aggregation, and sorting of aggregation buckets by count or custom criteria. This enables the creation of rich, hierarchical aggregation structures that are essential for analyzing complex data sets with nested attributes, enhancing the ability to extract meaningful insights from nested data in Elasticsearch.

###  Input Parameters

| Parameter     | Type   | Description                                                                                                        |
|---------------|--------|--------------------------------------------------------------------------------------------------------------------|
| `terms`       | array  | An array of strings representing the path to the nested field(s) within the Elasticsearch document.                |
| `search`      | string | The field to be searched within the nested documents.                                                              |
| `title`       | string | An additional field to be included in the script source for aggregation.                                           |
| `size`        | number | (Optional) The maximum number of aggregation buckets to return. If not specified, a default value is used.         |
| `filterTerm`  | string | (Optional) A term used to filter the documents before aggregation. Default is an empty string, implying no filter. |
| `filterField` | string | The field against which the `filterTerm` is applied.                                                               |
| `extraFields` | object | (Optional) Additional fields to be included in the aggregations.                                                   |
| `minDocCount` | number | The minimum count of documents required for a term to appear in the bucket. Default is                           |
| `sort`        | string | The sorting order of the aggregation buckets. Default is `_count`, which sorts by the count of documents.          |

###  Expected Output

The function returns an Elasticsearch aggregation query object structured for nested field aggregations. Depending on the `terms` array's length, it might return a nested object that recursively calls `buildNested` to handle multi-level nested fields or a final aggregation object for the deepest nested field specified.

###  Explanation of Single Components

- **Recursive Structure**: If the `terms` array contains more than one element, indicating multiple levels of nested fields, the function modifies the `terms` array to reflect the next nested path and recursively calls itself to build the nested aggregation structure until it reaches the last term.

- **Base Case**: Once the `terms` array has only one element, the function constructs the aggregation query for that nested field, including the `terms` aggregation with the specified `size`, `min_doc_count`, and `order`. A script is used to combine the values of the `search` and `title` fields in the aggregation buckets.

- **Extra Fields**: If `extraFields` are provided, additional sub-aggregations are added for each specified field.

- **Filtering**: If a `filterTerm` is provided, a filter aggregation is added to the query, filtering the documents based on the `filterTerm` and `filterField` before applying the nested aggregation.

- **Distinct Terms**: An additional aggregation named `distinctTerms` is added to count unique terms for the `search` field.

## buildTerms

The `buildTerm` function is adept at constructing tailored terms aggregations for Elasticsearch queries. It enables the aggregation of documents based on complex criteria involving scripts and the inclusion of additional sub-aggregations for further analysis. The function's support for global aggregations and pre-aggregation filtering extends its utility, allowing for broad or highly focused analyses. This flexibility makes `buildTerm` a valuable tool for generating rich insights from Elasticsearch data, supporting a wide range of search and analysis scenarios.

###  Input Parameters

| Parameter     | Type    | Description                                                                                               |
|---------------|---------|-----------------------------------------------------------------------------------------------------------|
| `term`        | object  | An object containing `search` and `title` fields to specify the term for aggregation.                     |
| `size`        | number  | The maximum number of aggregation buckets to return.                                                      |
| `extra`       | object  | (Optional) Additional fields for sub-aggregations.                                                        |
| `sort`        | string  | The sorting order of the aggregation buckets. Default is `_count`, which sorts by the count of documents. |
| `global`      | boolean | Flag indicating whether the aggregation should be applied globally across all documents.                  |
| `filterQuery` | object  | (Optional) A query object to filter the documents before aggregation.                                     |

###  Expected Output

The function returns an Elasticsearch aggregation query object. Depending on the parameters, this object may include a terms aggregation based on a script, sub-aggregations for extra fields, a global aggregation wrapper, and/or a filter aggregation.

###  Explanation of Single Components

- **Terms Aggregation**: The core of the function is constructing a `terms` aggregation with a script that combines the values of `search` and `title` fields. The `size` and `order` of the buckets are based on the `sort` parameter.

- **Distinct Terms**: A distinct terms aggregation is created to count unique terms for the `search` field.

- **Extra Aggregations**: If the `extra` parameter is provided, additional sub-aggregations are added for each field specified in `extra`, allowing for more detailed breakdowns within each term bucket.

- **Filtering**: If a `filterQuery` is provided, it wraps the terms aggregation (and any extra aggregations) within a filter aggregation, ensuring that only documents matching the filter criteria are included in the aggregation.

- **Global Aggregation**: If the `global` flag is set to `true`, the entire aggregation structure is wrapped within a `global` aggregation, which causes the aggregation to consider all documents in the index, regardless of any query scope that might otherwise limit the documents being aggregated.

## distinctTerms

The `distinctTerms` function is designed to provide a straightforward way to include a distinct term count in an Elasticsearch query. By using the `cardinality` aggregation, it enables an efficient estimation of the number of unique values for a given field within the dataset. This functionality is particularly useful for understanding the diversity of data in a field, which can help in assessing the field's cardinality and guiding data analysis decisions. The `distinctTerms` function simplifies the process of integrating this aggregation into larger Elasticsearch queries, supporting analytics and search features that benefit from knowledge of data uniqueness.

###  Input Parameters

| Parameter | Type   | Description                                                    |
|-----------|--------|----------------------------------------------------------------|
| `term`    | string | The field for which to calculate the number of distinct terms. |

###  Expected Output

The function returns an Elasticsearch aggregation query object that uses the `cardinality` aggregation to calculate the number of distinct terms in the specified field.

###  Explanation of Single Components

- **Cardinality Aggregation**: The `cardinality` aggregation is a single-value metrics aggregation that calculates an approximate count of distinct values. In this function, it's applied to the field specified by the `term` parameter to count the unique terms within that field.


