# ES Presentation Demo Resources

## Create Index and Mapping
```
PUT /es-product-demo
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 3
  },
  "mappings": {
    "properties": {
      "id": { "type": "keyword" },
      "name": { "type": "text" },
      "tags": { "type": "keyword" },
      "price": { "type": "float" },
      "active": { "type": "boolean" },
      "createdDate": { "type": "date" },
      "variantStock": {
        "type": "nested",
        "properties": {
          "size": { "type": "keyword" },
          "stock": { "type": "integer" }
        }
      },
      "reviews": {
        "properties": {
          "userName": { "type": "text" },
          "rating": { "type": "integer" }
        }
      }
    }
  }
}
```

## Bulk Insert Data
```
POST /es-product-demo/_bulk
{"index":{"_id":1}}
{"id":1,"name":"Cool Red Shirt","tags":["Shirt","Red"],"price":25.99,"active":true,"createdDate":"2021-09-08","variantStock":[{"size":"Small","stock":15},{"size":"Medium","stock":42},{"size":"Large","stock":3}],"reviews":[{"userName":"John Doe","rating":4},{"userName":"Sarah Leg","rating":3}]}
{"index":{"_id":2}}
{"id":2,"name":"Amazing Jazzy Shorts","tags":["Shorts"],"price":55.50,"active":true,"createdDate":"2021-09-06","variantStock":[{"size":"Small","stock":0},{"size":"Medium","stock":15},{"size":"Large","stock":32}],"reviews":[{"userName":"Bob Builder","rating":5},{"userName":"Sofine Signer","rating":4}]}
{"index":{"_id":3}}
{"id":3,"name":"Red Amazing Hat","tags":["Red","Hat"],"price":29.99,"active":true,"createdDate":"2021-09-06","variantStock":[{"size":"Small","stock":20},{"size":"Medium","stock":1},{"size":"Large","stock":12}],"reviews":[{"userName":"Jonny Jackhammer","rating":2},{"userName":"Bobby Bad","rating":1}]}
{"index":{"_id":4}}
{"id":4,"name":"Supreme Blue Jacket","tags":["Blue","Jacket"],"price":82,"active":false,"createdDate":"2021-09-02","variantStock":[{"size":"Small","stock":0},{"size":"Medium","stock":0},{"size":"Large","stock":3}]}
{"index":{"_id":5}}
{"id":5,"name":"Sneaky Sneakers","tags":["Sneakers"],"price":149.99,"active":true,"createdDate":"2021-09-08","variantStock":[{"size":"Small","stock":35},{"size":"Medium","stock":18},{"size":"Large","stock":2}],"reviews":[{"userName":"Roger Rabbit","rating":2},{"userName":"Mary Maine","rating":5}]}
{"index":{"_id":6}}
{"id":6,"name":"Soft Gloves Red","tags":["Gloves", "Red"],"price":34.80,"active":true,"createdDate":"2021-09-01","variantStock":[{"size":"Small","stock":84},{"size":"Medium","stock":2},{"size":"Large","stock":41}],"reviews":[{"userName":"Supper Man","rating":4},{"userName":"John Doe","rating":2}]}
```

## Get all products
```
GET /es-product-demo/_search
{
  "query": {
    "match_all": {}
  }
}
```

## Single Match Query
#### Analyses the search term `SHIRT !red!` --> `["shirt", "red"]`, and find all documents that contains 1 or more of these terms in the `name` field
```
GET /es-product-demo/_search
{
  "_source": [
    "name",
    "tags",
    "price"
  ],
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "SHIRT !red!"
          }
        }
      ]
    }
  }
}
```

## Analyse Text Field
#### Analyses the text `SHIRT !red!` --> `["shirt", "red"]` since the `name` property is of type `text` which executes in the Query Context
```
GET /es-product-demo/_analyze
{
  "field": "name",
  "text": "SHIRT !red!"
}
```

## Analyse HTML with Email Text Field
### The goal is to get the text to only contain the upper case version of the emails
#### Standard Analysing
```
GET /es-product-demo/_analyze
{
  "text": "<a href='mailto:info@mycompany.com'>Info@mycompany.com | Sales@MyCompany.com</a>",
  "char_filter": [],
  "tokenizer": "standard",
  "filter": ["lowercase"]
}
```
#### Strip HTML elements
```
GET /es-product-demo/_analyze
{
  "text": "<a href='mailto:info@mycompany.com'>Info@mycompany.com | Sales@MyCompany.com</a>",
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "filter": ["lowercase"]
}
```
#### Use the Email tokenizer
```
GET /es-product-demo/_analyze
{
  "text": "<a href='mailto:info@mycompany.com'>Info@mycompany.com | Sales@MyCompany.com</a>",
  "char_filter": ["html_strip"],
  "tokenizer": "uax_url_email",
  "filter": ["lowercase"]
}
```
#### Uppercase all tokens with a token filter
```
GET /es-product-demo/_analyze
{
  "text": "<a href='mailto:info@mycompany.com'>Info@mycompany.com | Sales@MyCompany.com</a>",
  "char_filter": ["html_strip"],
  "tokenizer": "uax_url_email", 
  "filter": ["uppercase"]
}
```

## Analyse Keyword Field
#### Analyses the text `SHIRT !red!` --> `["SHIRT !red!"]` since the `tags` property is of type `keyword` which executes in the Filter Context
```
GET /es-product-demo/_analyze
{
  "field": "tags",
  "text": "SHIRT !red!"
}
```

## Match with Should query to boost relevance scoring
#### Matches all the documents containing the term `red` in the `name` property, and boosts the relevance score if the term `soft` appears in the `name` field
```
GET /es-product-demo/_search
{
  "_source": [
    "name",
    "tags",
    "price"
  ],
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "red"
          }
        }
      ],
      "should": [
        {
          "match": {
            "name": "SOFT"
          }
        }
      ]
    }
  }
}
```

## Exclude all products with `Shirt` tag
#### Same as above, but excludes all documents that contain the term `Shirt` as a tag
```
GET /es-product-demo/_search
{
  "_source": [
    "name",
    "tags",
    "price"
  ],
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "red"
          }
        }
      ],
      "should": [
        {
          "match": {
            "name": "SOFT"
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "tags": {
              "value": "Shirt"
            }
          }
        }
      ]
    }
  }
}
```

## Only show products with price less than or equal to 30
#### Same as above, but filters documents where the price is less than or equal to 30
```
GET /es-product-demo/_search
{
  "_source": [
    "name",
    "tags",
    "price"
  ],
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "red"
          }
        }
      ],
      "should": [
        {
          "match": {
            "name": "SOFT"
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "tags": {
              "value": "Shirt"
            }
          }
        }
      ],
      "filter": [
        {
          "range": {
            "price": {
              "lte": 30
            }
          }
        }
      ]
    }
  }
}
```

## Search Reviews for `John` with rating 4 or higher
#### Attempts to find all documents where there is a review made by `john` with a rating of more than 3. This will not return the expected result as the array is of a normal `object` type, which does not maintain relationships of the object properties. Thus all documents containing a review by `john`, and with any review with a rating of 3 or more, will be matched
```
GET /es-product-demo/_search
{
  "_source": [
    "name",
    "reviews"
  ],
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "reviews.userName": "JOHN"
          }
        },
        {
          "range": {
            "reviews.rating": {
              "gt": 3
            }
          }
        }
      ]
    }
  }
}
```

## Search Variant Stock for `Large` with more than 30 in stock
#### Matches all documents that has the more than 30 `Large` items
```
GET /es-product-demo/_search
{
  "_source": [
    "name",
    "variantStock"
  ],
  "query": {
    "bool": {
      "must": [
        {
          "nested": {
            "path": "variantStock",
            "query": {
              "bool": {
                "must": [
                  {
                    "match": {
                      "variantStock.size": "Large"
                    }
                  }
                ],
                "filter": [
                  {
                    "range": {
                      "variantStock.stock": {
                        "gt": 30
                      }
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  }
}
```

# BONUS

## Create Index with nested reviews
```
PUT /es-product-nested-reviews-demo
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 3
  },
  "mappings": {
    "properties": {
      "id": { "type": "keyword" },
      "name": { "type": "text" },
      "tags": { "type": "keyword" },
      "price": { "type": "float" },
      "active": { "type": "boolean" },
      "createdDate": { "type": "date" },
      "totalStock": { "type": "integer" },
      "averageRating": { "type": "float" },
      "variantStock": {
        "type": "nested",
        "properties": {
          "size": { "type": "keyword" },
          "stock": { "type": "integer" }
        }
      },
      "reviews": {
        "type": "nested",
        "properties": {
          "userName": { "type": "text" },
          "rating": { "type": "integer" }
        }
      }
    }
  }
}
```

## Re-Index using the ReIndex API
#### Added bonus to calculate newly added fields `totalStock` and `averageRating`
```
POST /_reindex
{
  "source": {
    "index": "es-product-demo"
  },
  "dest": {
    "index": "es-product-nested-reviews-demo"
  },
  "script": {
    "source": """
    def totalStock = 0;
    for (def i = 0; i < ctx._source.variantStock.length; i++) {
      totalStock += ctx._source.variantStock[i].stock;
    }
    ctx._source.totalStock = totalStock;
    
    def totalRatings = 0;
    if (ctx._source.reviews == null) {
      return;
    }
    for (def i = 0; i < ctx._source.reviews.length; i++) {
      totalRatings += ctx._source.reviews[i].rating;
    }
    ctx._source.averageRating = Math.round(totalRatings / ctx._source.reviews.length * 100) / 100;
    """
  }
}
```

## Get all re-indexed documents
```
GET /es-product-nested-reviews-demo/_search
{
  "query": {
    "match_all": {}
  }
}
```

## Search Reviews for `John` with rating 4 or higher
#### This will now return expected results, since reviews are of type nested
```
GET /es-product-nested-reviews-demo/_search
{
  "_source": [
    "name",
    "reviews"
  ],
  "query": {
    "bool": {
      "must": [
        {
          "nested": {
            "path": "reviews",
            "query": {
              "bool": {
                "must": [
                  {
                    "match": {
                      "reviews.userName": "JOHN"
                    }
                  },
                  {
                    "range": {
                      "reviews.rating": {
                        "gt": 3
                      }
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  }
}
```
