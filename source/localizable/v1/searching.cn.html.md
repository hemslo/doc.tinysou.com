---
layout: api.cn
section: searching
toc: article
title: 搜索 API
---

## 总述

搜索 API，均需要提供 `auth_token` 以通过权限验证。验证方式请参考[权限验证][auth]一节。

搜索 API 的每个 endpoint 均支持 `GET` 和 `POST` 两种 HTTP 请求方法。其中 `GET` 接受 `url 编码`的参数， `POST` 接受 `json 编码`的参数。
由于`url 编码`存在一定的局限性或者不确定性，当需要使用复杂的参数时，我们推荐使用 `POST` 的请求方式。

## Endpoint

### 1.搜索单个 `collection`

```
GET /engines/:engine_name/collections/:collection_name/search
```

```
POST /engines/:engine_name/collections/:collection_name/search
```

### 2.搜索多个 `collection`

```
GET /engines/:engine_name/search
```

```
POST /engines/:engine_name/search
```

## 参数

| 名称    | 类型    | 说明 |
| ------ | ------ | ------------------------------------------------------ |
| q   | string | 待搜索的内容。**必需**。 |
| c   | string | 在 [Endpoint 2][endpoint2]中，指定需要搜索的多个 `collection`。在 [Endpoint 2][endpoint2] 中**必需**，在 [Endpoint 1][endpoint1] 中**无效**。 |
| page   | number | 分页参数，指定返回结果的起始页数，默认从第 0 页开始。**可选** |
| per_page   | number | 分页参数，指定每页显示条目的数据量，默认每页20条。**可选** |
| search_fields   | array(of string) | 需要被搜索的 `field`。**默认值**：所有`string`和`text`类型的`field`。**可选** |
| fetch_fields   | array(of string) | 搜索结果中，`document` 需要包含的 `field`。 **默认值**：所有`field`。**可选** |
| sort   | hash | 排序方式。 **可选** |

### `q`

待搜索的内容。例如 "自定义样式 css"。

### `c`

以','分隔多个`collection`的`name`。例如：`"post,comment"`表示：搜索"posts","comments"两个`collection`。

### `search_fields`

默认情况下，会对 `document` 的所有`string` 和 `text` 类型的 `field` 进行搜索。如果想只搜索特定的 `field`，可以利用 `search_fields` 进行指定。例如：
`["title", "body"]` 表示只搜索每个 `document` 的 'title', 'body' 两个 `field`。

可以被搜索的 `field` 类型包括：`string`，`text`，`enum`，`integer`，`float`。

### `fetch_fields`

默认情况下，搜索结果中的每个`document`，会包含所有`field`。如果只需要每个返回的`document`包含特定的 `field` (例如是出于带宽的考虑)，可以利用 `fetch_fields` 参数来进行指定。例如 `["title", "url"]` 表示返回结果中，每个 `document` 只包含 'title', 'url' 两个 `field`。

### `sort`

默认情况下，根据搜索结果中每个`document`的[score][score]，对搜索结果进行排序。如果你需要按照特定 `field` 进行排序，可通过 `sort` 参数实现。例如：

```
{
  "field": "price",
  "order": "asc",
  "mode": "avg"
}
```

表示按照 'price' 升序(从小到大)排序，当 'price' 是个 `array` 时，取平均值作为排序依据。

`order` 用来指定排序方式；`order` 的可选项包括：`asc` 和 `desc`。

`mode` 用来指定，当需要用来排序的 `field` 的值是 `array` 时，如何处理。`mode` 的可选项包括：`min`，`max`，`sum` 和 `avg`。

可以用来排序的 `field` 类型包括：`enum`，`integer`，`float`，`date`。

## 结果

搜索结果的结构如下：

| 名称    | 类型    | 说明 |
| ------ | ------ | ------------------------------------------------------ |
| info   | hash | 本次搜索相关信息，包括搜索内容，结果数等等。 |
| records   | array | 搜索结果(`document`)的列表。 |
| errors   | hash | 相关错误信息。 |

### info

`info` 部分由如下内容组成：

| 名称    | 类型    | 说明 |
| ------ | ------ | ------------------------------------------------------ |
| query   | string | 本次搜索的搜索内容。例如：`"自定义样式 css"`。 |
| page   | number | 本次搜索的 `page` 参数值。例如：`1`。 |
| per_page | number | 本次搜索的 `per_page` 参数值。例如：`10`。 |
| total | number | 搜索结果的总数。例如：`19`。 |
| max_score | number | 搜索结果中，`score` 的最大值。例如：`1.3830765`。 |

### records

`records` 的每个元素是一个`document`，由如下内容组成：

| 名称    | 类型    | 说明 |
| ------ | ------ | ------------------------------------------------------ |
| collection   | string | 该`document`所属`collection`。 |
| score   | number | 该`document`的"score"。 |
| highlight | hash | 以<em></em>标记出该`document`与搜索内容符合的部分。 |
| document | hash | 该`document`所包含的`field`。 |

例如：

```json
{
  "collection": "post",
  "score": 0.048643537,
  "highlight": {
      "body": [
          "我们很高兴地在这里宣布，今天开始，<em>微搜索</em>支持<em>拼音</em><em>搜索</em>了。<em>微</em><em>搜索</em>的<em>拼音</em><em>搜索</em>包括：<em>拼音</em><em>补全</em>，首字母<em>搜索</em>及<em>补全</em>，例如：如果你的文本中包含 程序员，输入 cxy, chen, chengxu均可匹配到程序员效果如下图所示："
      ],
      "title": [
          "支持<em>拼音</em><em>搜索</em>"
      ]
  },
  "document": {
      "id": "53c028da70726f7e2d010000",
      "title": "支持拼音搜索",
      "tags": [
          "公告",
          "特性"
      ],
      "author": "Michael Ding",
      "date": "2014-08-16T00:00:00.000Z",
      "body": "我们很高兴地在这里宣布，今天开始，微搜索支持拼音搜索了。微搜索的拼音搜索包括：拼音补全，首字母搜索及补全，例如：如果你的文本中包含 程序员，输入 cxy, chen, chengxu均可匹配到程序员效果如下图所示："
  }
}
```

### errors

显示相关错误信息，例如 `search_fields` 指定了不存在的 `field`等。

例如：

```json
{
  "search_fields": [
    "collection page dosen't contain text fields: [\"noexist\"]"
  ]
}
```

## 示例

```
curl -XPOST 'http://api.tinysou.com/v1/engines/blog/collections/post/search' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: token YOUR_AUTH_TOKEN' \
  -d '{
        "q": "自定义样式",
        "fetch_fields": ["title", "sections", "url"],
        "sort": {
          "field": "update_at",
          "order": "desc",
          "mode": "min"
        },
        "per_page": 100
      }' | json_reformat
```

返回

```json
{
    "info": {
        "query": "自定义样式",
        "page": 0,
        "per_page": 100,
        "total": 17,
        "max_score": 1.3830765
    },
    "records": [
        {
            "collection": "page",
            "score": 1.3830765,
            "highlight": {
                "body": [
                    "基本概念    <em>自定义</em>搜索<em>样式</em>    <em>自定义</em>搜索行为    API 快速上手    添加 opensearch 支持              指南   指南概述    基本概念    <em>自定义</em>搜索<em>样式</em>    自定义搜索行为",
                    "<em>自定义</em>搜索行为    API 快速上手    添加 opensearch 支持          <em>自定义</em>搜索<em>样式</em>   你可以通过添加 CSS 代码来覆盖默认的<em>样式</em>。  注意：为了定制微搜索<em>样式</em>，你需要对HTML和CSS相关知识有一些了解。",
                    "和CSS相关知识有一些了解。  搜索框<em>样式</em>  安装微搜索搜索框时，可以为 input 元素添加 ts-search-input CSS class来覆盖默认的<em>样式</em>。  <form>\n  <input type=\"text\"",
                    "</form>  定制搜索表单的<em>样式</em>，需要覆盖 form input.ts-search-input <em>样式</em>。如果去掉 ts-search-input class，表单将使用你网站的默认表单<em>样式</em>。  你可以随意定制搜索框的样式。例如：",
                    "你可以随意定制搜索框的<em>样式</em>。例如：  body form input.ts-search-input {\n  color: green;\n  font-weight: bold;\n}  自动补全下拉框<em>样式</em>  定制自动补全下拉框的样式，需覆盖"
                ],
                "sections": [
                    "<em>自定义</em>搜索<em>样式</em>",
                    "搜索框<em>样式</em>",
                    "自动补全下拉框<em>样式</em>",
                    "搜索结果<em>样式</em>"
                ],
                "title": [
                    "<em>自定义</em>搜索<em>样式</em>"
                ]
            },
            "document": {
                "id": "1eaff92c20998b096190c45ac92fb444",
                "title": "自定义搜索样式",
                "sections": [
                    "微搜索 文档中心",
                    "自定义搜索样式",
                    "文档中心",
                    "搜索框样式",
                    "自动补全下拉框样式",
                    "搜索结果样式",
                    "搜索结果链接颜色",
                    "搜索结果高亮",
                    "搜索结果字体"
                ],
                "url": "http://doc.tinysou.com/guides/custom-styles.html"
            }
        },
        ...
        {
            "collection": "page",
            "score": 0.0012169777,
            "highlight": {
                "body": [
                    "为大家提供一个易于安装又特性丰富的站内搜索引擎。本次的内测版本，跨站点全文搜索，并提供搜索框的下拉<em>式</em>自动补全。而整个过程只需要简单的三步：创建引擎，添加站点地址，复制代码到 html。内测阶段需要注册",
                    " 参数   创建 document 的参数遵循 collection 的 field_types <em>定义</em>。  参数名为 field_types 中的 key , 参数值类型与 field_types 中的"
                ]
            },
            "document": {
                "id": "d51ec79799c23f6b06fe83e0d735417d",
                "title": "索引 API",
                "sections": [
                    "微搜索 文档中心",
                    "索引 API",
                    "文档中心",
                    "总述",
                    "Engine",
                    "Collection",
                    "Document",
                    "罗列 Engines",
                    "创建一个 Engine",
                    "获取一个 Engine",
                    "更新一个 Engine",
                    "删除一个 Engine",
                    "罗列 Collections",
                    "创建一个 Collection",
                    "获取一个 Collection",
                    "删除一个 Collection",
                    "罗列 Documents",
                    "创建一个 Document(自动生成 id 方式)",
                    "创建一个 Document(指定 id 方式)",
                    "获取一个 Document",
                    "创建或更新一个 Document",
                    "删除一个 Document"
                ],
                "url": "http://doc.tinysou.com/v1/indexing.html"
            }
        }
    ],
    "errors": {}
}
```

[endpoint1]:/v1/searching.html#2-1-1.搜索单个-collection
[endpoint2]:/v1/searching.html#2-2-2.搜索多个-collection
[auth]:/v1/overview.html#6-权限验证
[score]:/v1/overview.html#4-score
