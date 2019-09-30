+++
title = "Mongodb小记事本"
date = "2019-07-25T23:04:46+08:00"
description = "自己的mongodb记事本"
tags = ["数据库"]
+++

## 更新多层嵌套的数组数据

```javascript
db.updateOne(
  { _id: new ObjectID(xxx) },
  { $set: { 'objData.$[objective].learnVideos.$[data].status': info.status } },
  {
    upsert: true,
    arrayFilters: [
      {
        'objective.objectiveId': new ObjectID(objectiveId),
      },
      {
        'data.url': info.url,
      },
    ],
  }
)
```
