---
title: MongoDB schema design
description: The post is about MongoDB schema design.
date: 2022-12-22
tags:
  - MongoDB
  - Tech
layout: layouts/post.njk
readtime: 14
---

我第一個完整設計 Schema 的專案是在 2021 年，當時我們使用的是 MongoDB，做的應用程式是關於測試資料的分析，我們的資料來源是從測試機器上的 STDF 檔案，經過 Parser，並將這些資料存到 MongoDB 裡面，以便我們之後可以進行分析。

由於是第一次設計反正規化的 Schema，所以我們在設計的時候，可以說是東踩一個雷西踩一個屎，但之後還是有摸索出適合的 Schema。

之後參與 MongoDB 年會，有一場次是關於 Schema Design 的，那次議程將常用的 Schema Design Pattern 進行了整理，並且提供了一些設計的原則，也包含了我們以前自行摸索出來的 Pattern，有點相見恨晚的感覺，所以我將這些資訊整理成一篇文章，方便之後查閱。

## MongoDB

MongoDB 是 JSON Document Store 的資料庫，屬於 Flexible Data Model，讓使用者可以更彈性的制定專屬於應用的資料模型，可以**不需要再一昧的要求正規化**，能夠調整到最適合開發者的效益 (簡易方便/效能最佳)。

## 三個誤區

1. Flexible Schema = No Schema，MongoDB 不需要 Schema design

2. 應該用一個超級大文檔來組織所有數據

3. MongoDB 不支持關聯或者事務

## 關係模式設計

![1](/img/remote/mongodb-schema-design/1.png)
ref: MongoDB

## MongoDB 架構設計

MongoDB schema design 與 SQL schema design 有很大不同。想要像以前一樣設計類似於 SQL 模式的模式將數據拆分成整齊的小表格是完全正常的，但並不是這麼做。

以下這是 MongoDB 模式設計：

- No formal process(design flow)
- No algorithms
- No rules

## 如此自由的話，那我們要從何下手？

在設計 MongoDB 模式設計時，唯一重要的是設計一個適合你的 app 的 schema。

如果 app 的使用方式不同，即使使用完全相同數據，也可能有兩個非常不同的架構。

在設計 schema 時，我們要考慮以下因素：

- Store the data
- Provide good query performance
- Require reasonable amount of hardware

## SQL ,MongoDB 模型設計方法比較

![2](/img/remote/mongodb-schema-design/2.png)
ref: 此張圖來自非常久前的筆記，若有侵權請告知，我會立即刪除。

## SQL ,MongoDB document 比較

### SQL

![1](/img/remote/mongodb-schema-design/1.png)

### MongoDB

```json
{
_id: ObjectId("507f191e810c19729de860ea"),
first_name:'Paul',
sirname:'Miller',
cell:447557505611,
cite:'London',
location_x:45.123,
location_y:47.232,
professions:['banking','finance','trader'],
cars:[
	{
		model:'Bentley',
		yead:1973
	},
	{
		model:'Rolls Royce',
		yead:1965
	}
]

```

由上面的範例我們可以知道，這筆文檔在實現關聯關係時，採用的是**嵌入(Embedding)** ，另一種方式則是**引用(Referencing)**。

## Embedding vs. Referencing

MongoDB 現行主要使用兩種方式做資料關聯，分別為  **Embedding**與  **Referencing**。

### Embedding

將關聯資料以子文檔(Subdocuments)的方式存在同一個資料表裡面，方便直接取用。

### Referencing

將關聯資料獨立出來成一個文檔，適合分別存取使用，需要時可以透過 MongoDB 的 Join 功能 ($lookup) 來做關聯查詢。

![3](/img/remote/mongodb-schema-design/3.png)

所以 MongoDB 在資料關聯的應用上是彈性的，而 Embedding 與 Referencing 的選擇，可依照使用行為的不同來決定，上表是關聯的使用優點與注意事項，至於有沒有懶人包呢？ 有的。

那就是

> **_『盡量使用 Embedding 除非有效能問題』_**

## 什麼時候該使用引用方式？

1. 內嵌文檔太大，數 MB 或者超過 16MB (MongoDB 文檔大小不能超過 16MB)
2. 內嵌文檔或數組元素會頻繁修改
3. 內嵌數組元素會持續增長並且可能沒有極限

## Recap

### RDBMS vs. MongoDB

|              | RDBMS                      | MongoDB           |
| ------------ | -------------------------- | ----------------- |
| 模型設計層次 | 概念模型 邏輯模型 物理模型 | 概念模型邏輯模型  |
| 模型實體     | 表                         | 集合              |
| 模型屬性     | 列                         | 字段              |
| 模型關係     | 關聯關係 主外鍵            | 內嵌數組 引用字段 |

### MongoDB 模型設計

1. 在大多數情況下，資料關聯建議使用 Embeding。
2. 除非資料有單獨存取的需求，再使用 Referencing。
3. 關聯資料存取盡量避免使用 Join ($lookup)`，可以使用兩次查詢取代。
4. 如果有使用 Array，盡量避免儲存超過一百個。
5. 視情況參考進階的 Design Pattern 或是 Anti-Pattern。
