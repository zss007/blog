---
title: mongodb 聚合查询综合实例
categories:
- database
---
mongodb aggregation 综合实例
<!--more--> 

### 一、性能优化
- 查询结果需少于 16M
 - 使用 $limit & $project
- 每个管道阶段内存占用少于 100M
 - 使用索引
 - 使用 allowDiskUse，减慢查询速度，可规避 100M 限制，不过不适用于 $graphLookup，因为 $graphLookup 不支持

```
db.movies.aggregate([
  {
    $match: {
      title: /^[aeiou]/i
    }
  },
  {
    $project: {
      title_size: { $size: { $split: ["$title", " "] } }
    }
  },
  {
    $group: {
      _id: "$title_size",
      count: { $sum: 1 }
    }
  },
  {
    $sort: { count: -1 }
  }
])
<!-- 可优化为 -->
db.movies.aggregate([
  {
    $match: {
      title: /^[aeiou]/i
    }
  },
  {
    $sortByCount: {
      $size: { $split: ["$title", " "] }
    }
  }
])

db.stocks.aggregate([
  {
    $unwind: "$trades"
  },
  {
    $group: {
      _id: {
        time: "$id",
        action: "$trades.action"
      },
      trades: { $sum: 1 }
    }
  },
  {
    $group: {
      _id: "$_id.time",
      actions: {
        $push: {
          type: "$_id.action",
          count: "$trades"
        }
      },
      total_trades: { $sum: "$trades" }
    }
  },
  {
    $sort: { total_trades: -1 }
  }
])
<!-- 可优化为 -->
db.stocks.aggregate([
  {
    $project: {
      buy_actions: {
        $size: {
          $filter: {
            input: "$trades",
            cond: { $eq: ["$$this.action", "buy"] }
          }
        }
      },
      sell_actions: {
        $size: {
          $filter: {
            input: "$trades",
            cond: { $eq: ["$$this.action", "sell"] }
          }
        }
      },
      total_trades: { $size: "$trades" }
    }
  },
  {
    $sort: { total_trades: -1 }
  }
])
```

### 二、综合实例
使用 air_alliances 和 air_routes 集合，找出哪个 alliance 在 JFK 和 LHR 机场之间双向运作有最多的航空公司
```
db.air_routes.aggregate([
  {
    $match: {
      src_airport: { $in: ["LHR", "JFK"] },
      dst_airport: { $in: ["LHR", "JFK"] }
    }
  },
  {
    $lookup: {
      from: "air_alliances",
      foreignField: "airlines",
      localField: "airline.name",
      as: "alliance"
    }
  },
  {
    $match: { alliance: { $ne: [] } }
  },
  {
    $addFields: {
      alliance: { $arrayElemAt: ["$alliance.name", 0] }
    }
  },
  {
    $group: {
      _id: "$airline.id",
      alliance: { $first: "$alliance" }
    }
  },
  {
    $sortByCount: "$alliance"
  }
])
```