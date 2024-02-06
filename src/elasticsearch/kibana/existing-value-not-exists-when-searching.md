# Existing value not exist when searching log in kibana

<!-- toc -->

- [Existing value not exist](#existing-value-not-exist)
- [Ingest data](#ingest-data)
- [Search using filter](#search-using-filter)
- [Reason why existing data not exist](#reason-why-existing-data-not-exist)
- [Conclution](#conclution)
- [refs](#refs)

<!-- tocstop -->

Recently, when searching for application error logs in Kibana, I encountered a particularly intriguing challenge. Some of the expected search results were missing, when using `error` as filter field and `exists` as filter operator in kibana(Click `+Add Filter` button). After scratching my head for a while and doing a deep dive into why this was happening, I stumbled upon a lifesaver feature in Elasticsearch - the `ignore_above` parameter.

Below are the samples for `error` field in log records.

```json
[
  "你所在的区域无法查看特定的内容，请联系相关部门进行处理",
  "author is not authenticated missing parameters in request, request=id_is:Kp3mE5QJwVHkM1Wl3xCg client_id_is:MoNRe2Qxvz4nR8R9oZtA payment_number:\"jXb9q8flSY6vQIpV3GtE\" order_detail_id:L3R9mU1W8pN9gKuH6Wz deliver_timestamp:{seconds:QG4cP2a0QK6SxI6D0EbR} resp=internal_code:INTERNAL_CODE_PAYMENT_NOT_FOUND result_description:\"INTERNAL_CODE_PAYMENT_NOT_FOUND\"",
  "author is not authenticated missing parameters in request, request=user_id_is:T7L3b4HnE1pM6L9QeXy5 client_id_is:w1L6i0uJ3S0A9X6K4G5 payment_number:\"K0l6W0kR2rH9Y0O5Z0x8\" order_detail_id:J2A0B1Z4uE9X7Y2W7L0E deliver_timestamp:{seconds:1I3lC6nT8Q4N7j0X4I6v} resp=internal_code:INTERNAL_CODE_PAYMENT_NOT_FOUND result_description:\"INTERNAL_CODE_PAYMENT_NOT_FOUND\"",
  "author is not authenticated missing parameters in request, request=user_id_is:G5B9kS6pJ3C7H0D8K6M4 client_id_is:w9R0V6mF3A8D4H1G2G6 payment_number:\"Z3R0b8D7uJ2Z4Q5W7X6E\" order_detail_id:Y7T2R8X1vI6E0G3C7M3E deliver_timestamp:{seconds:4K6G9O5N3fP1K6S7R8Y} resp=internal_code:INTERNAL_CODE_PAYMENT_NOT_FOUND result_description:\"INTERNAL_CODE_PAYMENT_NOT_FOUND\"",
  "author is not authenticated missing parameters in request, request=user_id_is:U8T0D9hE8C3L6H9K5V7 client_id_is:O4xH2L3K5R0C9X2O9M0 merchant_id_iss:z5fN6qO0R4eV9tK8E2bB payment_number:\"D9H4fQ2O5W3C7R8E2C6\" order_detail_id:W8sI5kQ2dP3W1C7G9T5 deliver_timestamp:{seconds:R0U7X1Q3U5sE2Y4O8D} resp=internal_code:INTERNAL_CODE_PAYMENT_NOT_FOUND result_description:\"INTERNAL_CODE_PAYMENT_NOT_FOUND\""
]
```

The question is: Why doesn't the error log saying "author is not ..." appear in the search results?

# Ingest data

Now, let's do some investigation!

First, we creates an index named `tmp-1` with a single field called `error`. The `error` field is of type `text` for full-text search and has a sub-field named `keyword` of type `keyword` for exact matching, with a maximum length of `20` characters.

```json
PUT /tmp-1
{
  "mappings": {
    "properties": {
      "error": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 20
          }
        }
      }
    }
  }
}
```

Then, we ingest some data to this index:

```json
PUT /tmp-1/_doc/1
{
  "error": "This is a test.",
  "@timestamp": "2024-02-05T12:35:35Z"
}

PUT /tmp-1/_doc/2
{
  "error": "12345678901234567890",
  "@timestamp": "2024-02-05T12:35:35Z"

}

PUT /tmp-1/_doc/3
{
  "error": "12345678901234567890X",
  "@timestamp": "2024-02-05T12:35:35Z"

}
```

Notice, for doc three, the length of error field is longer than 20, which will cause problem when filtering doc in kibana.

# Search using filter

First, let's search with `error.keyword` exist(doc 3 is missing):

![Search with error.keyword exist](https://raw.githubusercontent.com/lichuan6/i/main/es/Screenshot%202024-02-05%20at%2020.48.52.png)

Let's see what is the result when searching with `error exist`(all docs are searchable):

![Search with error exist](https://github.com/lichuan6/i/blob/main/es/Screenshot%202024-02-05%20at%2020.49.15.png?raw=true)

# Reason why existing data not exist

In Elasticsearch, `ignore_above` is a parameter that can be used in the mapping of string fields, specifically for keyword types. The role of this parameter is to **ignore** (i.e., not index) any string which length is above the specified number of characters.

Let's consider an example:

```json
"mappings": {
  "properties": {
    "name": {
      "type": "keyword",
      "ignore_above": 20
    }
  }
}
```

In the above mapping, any `name` field that has more than `20` characters will not be indexed and therefore, will not be **searchable**.

It's important to note that the `ignore_above` only influences the indexing process and not the `_source` field (`_source`), which means the value of the field remains untouched when returned from a query.

As for how it affects search results:

If you depend on a keyword search for a specific field and that field's length exceeds the `ignore_above` value, you **won't** retrieve any information regarding that field in your search results because Elasticsearch didn't index it.

This feature is especially useful for preventing huge keywords from being indexed (e.g., when a text field is mistakenly indexed as a keyword), effectively **saving disk space** in your Elasticsearch cluster.

# Conclution

The conclusion is, Elasticsearch's `ignore_above` parameter is beneficial when you want to prevent the indexing of excessively long keywords. This can effectively save storage space in your Elasticsearch cluster. However, you need to be aware that if a field's length exceeds the `ignore_above` value, it **won't be indexed** and, therefore, you **won't be able to search** for it in your search results - even though it will still exist in the `_source` field. It's crucial to set the `ignore_above` parameter carefully, considering the nature and requirements of your data and searches.
