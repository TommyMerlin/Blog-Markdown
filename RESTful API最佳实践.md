---
title: RESTful API最佳实践
date: 2020-05-20 10:40:04
tags:
- RESTful
- 微服务
- API
categories:
- 微服务
- API设计
---

### RESTful API CRUD 操作与 HTTP 请求对应关系

- `GET /tickets` - Retrieves a list of tickets
- `GET /tickets/12` - Retrieves a specific ticket
- `POST /tickets` - Creates a ne w ticket
- `PUT /tickets/12` - Updates ticket #12
- `PATCH /tickets/12` - Partially updates ticket #12
- `DELETE /tickets/12` - Deletes ticket #12

<!-- more -->

利用现有 HTTP 请求方法在单个 `/tickets` 端点上实现丰富功能，没有要遵循的方法命名约定，URL 结构清晰明了。

### 端点名称为单数 or 复数？

保持最简单原则，全部都采用**复数**，保持 URL 一致性且无需考虑单词复数形式（person/people,goose/geese)。

### 如何处理关系？

如果某一关系只能存在于其他资源内，举例如下：

- `GET /tickets/12/messages` - Retrieves list of messages for ticket #12
- `GET /tickets/12/messages/5` - Retrieves message #5 for ticket #12
- `POST /tickets/12/messages` - Creates a new message in ticket #12
- `PUT /tickets/12/messages/5` - Updates message #5 for ticket #12
- `PATCH /tickets/12/messages/5` - Partially updates message #5 for ticket #12
- `DELETE /tickets/12/messages/5` - Deletes message #5 for ticket #12

### 如果某一行为和 HTTP 请求方式无法很好对应如何处理？

- 重组操作以使其看起来像资源的字段。如果操作不使用参数，则此方法有效。例如，*激活* 动作可以映射到布尔`activated`字段，并通过 PATCH 更新到资源。
- 使用 RESTful 原则将其视为子资源。 例如，GitHub 的 API 允许您通过 `PUT /gists/:id/star` 来 [star a gist](http://developer.github.com/v3/gists/#star-a-gist) 和通过 `DELETE /gists/:id/star` 来 [unstar](http://developer.github.com/v3/gists/#unstar-a-gist) .
- 有时候，确实无法将操作映射到合理的 RESTful 结构。 例如，将多资源搜索应用于特定资源的端点并没有任何意义。 在这种情况下，`/search` 即使不是资源，也将是最有意义的。 只需从 API 使用者的角度进行正确的操作，并确保已对其进行了清晰记录，以免造成混淆即可。

### 结果过滤排序和搜索

**过滤**

通过指定唯一的请求参数，例如过滤得到状态为 open 的 tickets，则请求为：`GET /tickets?state=open`。

**排序**

与过滤类似，可以使用参数 sort 来描述排序规则。通过允许 sort 参数接受逗号分隔的字段列表来适应复杂的排序要求。

- `GET /tickets?sort=-priority` - Retrieves a list of tickets in descending order of priority
- `GET /tickets?sort=-priority,created_at` - Retrieves a list of tickets in descending order of priority. Within a specific priority, older tickets are ordered first

**搜索**

有时基本的过滤器是不够的，你需要全文搜索。也许你已经在使用 ElasticSearch 或者其他基于 Lucene 的搜索技术。当使用全文搜索作为检索特定类型资源的资源实例的机制时，可以将其作为资源端点上的查询参数在 API 上公开。比如 `q`。搜索查询应该直接传递到搜索引擎，API输出应该与普通列表结果的格式相同。

- `GET /tickets?sort=-updated_at` - Retrieve recently updated tickets
- `GET /tickets?state=closed&sort=-updated_at` - Retrieve recently closed tickets
- `GET /tickets?q=return&state=open&sort=-priority,created_at` - Retrieve the highest priority open tickets mentioning the word 'return'

### 更新 & 创建操作应该返回资源展示

PUT、POST 或 PATCH 调用可能会对不属于提供参数的底层资源的字段进行修改（例如：created_at 或 updated_at 时间戳）。为了防止 API 使用者需要再次请求 API 来获取更新后的结果，将更新的（或创建的）结果作为响应的一部分。

[参考链接](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)