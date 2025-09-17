# Delete index

`DELETE /fragen`

# Mapping index

```json
PUT /fragen
{
  "settings": {
    "index": {
      "max_ngram_diff": 20
    },
    "analysis": {
      "analyzer": {
        "ngram_analyzer": {
          "tokenizer": "ngram_tokenizer",
          "filter": ["lowercase"]
        }
      },
      "tokenizer": {
        "ngram_tokenizer": {
          "type": "ngram",
          "min_gram": 3,
          "max_gram": 20,
          "token_chars": ["letter"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "question": { "type": "text", "analyzer": "ngram_analyzer" },
      "question_translation": { "type": "text", "index": false },
      "options": {
        "type": "nested",
        "properties": {
          "text": { "type": "text", "analyzer": "ngram_analyzer" },
          "translation": { "type": "text", "index": false }
        }
      },
      "correct": { "type": "text", "analyzer": "ngram_analyzer" },
      "state": { "type": "keyword" }
    }
  }
}
```

# Bulk insert

Use
`POST /fragen/_bulk` command and put `fragen_bulk.ndjson` file below of it and run it. Or use this command in terminal:

`curl -X POST "http://localhost:9200/fragen/_bulk" -H "Content-Type: application/x-ndjson" --data-binary @fragen_bulk.ndjson`

and empty format for adding another question:

```json

{ "index": { "_id": "3" } }
{
  "question": "",
  "question_translation": "",
  "options": [
    { "text": "", "translation": "" },
    { "text": "", "translation": "" },
    { "text": "", "translation": "" },
    { "text": "", "translation": "" }
  ],
  "correct": "",
  "state": "Deutschland"
}
```

# Query

To search questions:

## Get all

Size is optional.

```json
GET /fragen/_search
{
  "query": { "match_all": {} },
  "size": 100
}
```

## Search in questions

```json
GET /fragen/_search
{
  "query": {
    "match": {
      "question": "Regi"
    }
  }
}
```

## Search in question and answers

```json
GET /fragen/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "question": "Regierung"
          }
        },
        {
          "nested": {
            "path": "options",
            "query": {
              "match": {
                "options.text": "Religion"
              }
            }
          }
        }
      ]
    }
  }
}
```

with should

```json
GET /fragen/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "question": "Regierung"
          }
        },
        {
          "nested": {
            "path": "options",
            "query": {
              "match": {
                "options.text": "Religion"
              }
            }
          }
        }
      ]
    }
  }
}
```

## Search correct answer

```json
GET /fragen/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "correct": "frei"
          }
        }
      ]
    }
  }
}
```
