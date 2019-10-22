---
title: mongodb 聚合查询之多维分组
categories:
- database
---
mongodb aggregation 分组操作符
<!--more--> 

### 一、$sortByCount
$sortByCount 等效于 $group + $sort
```
db.movies.aggregate([
  {
    "$group": {
      "_id": "$imdb.rating",
      "count": { "$sum": 1 }
    }
  },
  {
    "$sort": { "count": -1 }
  }
])
<!-- 等同于 -->
db.movies.aggregate([
  {
    "$sortByCount": "$imdb.rating"
  }
])
```

### 二、$bucket
根据边界分段分组，boundaries 表示分段边界数组，default 表示不在边界内显示字段，默认显示 count，使用 output 可以自定义输出字段。
```
db.companies.aggregate([
  {
    "$match": {"founded_year": {"$gt": 1980}}
  },
  {
    "$bucket": {
      "groupBy": "$number_of_employees",
      "boundaries": [ 0, 20, 50, 100, 500, 1000, Infinity  ],
      "default": "Other",
      "output": {
        "total": {"$sum":1},
        "average": {"$avg": "$number_of_employees" },
        "categories": {"$addToSet": "$category_code"}
      }
    }
  }
])
```

### 三、$bucketAuto
自动分段分组
```
db.movies.aggregate([
  {
    "$match": { "imdb.rating": { "$gte": 0 } }
  },
  {
    "$bucketAuto": {
      "groupBy": "$imdb.rating",
      "buckets": 4,
      "output": {
        "average_per_bucket": { "$avg": "$imdb.rating" },
        "count": { "$sum": 1 }
      }
    }
  }
])
```

### 四、$facet
将 $bucket、$bucketAuto、$sortByCount 等聚合到同一阶段，每个键都是输出字段，不能包含 $facet、$out、$geoNear、$indexStats 和 $collStats。
```
db.companies.aggregate( [
  { "$match": { "$text": {"$search": "Databases"} } },
  { "$facet": {
    "Categories": [{"$sortByCount": "$category_code"}],
    "Employees": [
      { "$match": {"founded_year": {"$gt": 1980}}},
      {"$bucket": {
        "groupBy": "$number_of_employees",
        "boundaries": [ 0, 20, 50, 100, 500, 1000, Infinity  ],
        "default": "Other"
      }}],
    "Founded": [
      { "$match": {"offices.city": "New York" }},
      {"$bucketAuto": {
        "groupBy": "$founded_year",
        "buckets": 5   }
      }
    ]
}}]).pretty()

<!-- 应用实例 -->
同时在 imdb.rating 和 metacritic 字段上排前十的有多少部电影？
<!-- 聚合查询 -->
db.movies.aggregate([
  {
    $match: {
      metacritic: { $gte: 0 },
      "imdb.rating": { $gte: 0 }
    }
  },
  {
    $project: {
      _id: 0,
      metacritic: 1,
      imdb: 1,
      title: 1
    }
  },
  {
    $facet: {
      top_metacritic: [
        {
          $sort: {
            metacritic: -1,
            title: 1
          }
        },
        {
          $limit: 10
        },
        {
          $project: {
            title: 1
          }
        }
      ],
      top_imdb: [
        {
          $sort: {
            "imdb.rating": -1,
            title: 1
          }
        },
        {
          $limit: 10
        },
        {
          $project: {
            title: 1
          }
        }
      ]
    }
  },
  {
    $project: {
      movies_in_both: {
        $setIntersection: ["$top_metacritic", "$top_imdb"]
      }
    }
  }
])
```