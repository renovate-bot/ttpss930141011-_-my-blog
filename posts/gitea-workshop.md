---
title: 一個強大開源 Codebase 管理平台 — Gitea 的 Workshop
description: 這篇文會介紹如何使用 Gitea 這個開源的 Codebase 管理平台，並且開啟 Gitea Actions 做 CI/CD 流程。
date: 2023-08-03
scheduled: 2023-08-03
tags:
  - Tech
  - Infrastructure
layout: layouts/post.njk
readtime: 14
---


我在 Cloud summit 2023 參與了這個 [Workshop](https://cloudsummit.ithome.com.tw/2023/lab-page/2225) ，Gitea 是講者 Appleboy 與其他四位國外開發者在業餘時間所開發，建造一個開源的類 Github，誓言要取代且變成下一個它。

在這次 Workshop 上他介紹到，講者所任職的工作是一個很注重安全的職場環境，有些地方甚至連 Internet 都沒有，這個時候自架 Git 服務變成一個很麻煩的事情。若使用 Gitlab 又較耗資源，於是就誕生了 Gitea 這個專案，Gitea 使用 Golang 撰寫，編譯後的檔案為執行檔，可以跨平台執行，所需資源又十分小，只要一台沒人用的 VM 上開啟 Gitea service 就可以為全公司開啟 Git 服務，十分厲害。

這個專案整合了所有做完 Workshop 的材料，從一開始 Gitea 服務的建設，Runner 的建設，到最後上傳一個測試 Repo 並做好 Gitea actions。

## Prerequisite

- Docker
- Docker-compose


## **Installation**

### Clone repo

```docker
git clone https://github.com/ttpss930141011/Gitea-workshop.git
```

### 啟動 Gitea 服務

```docker
cd gitea
docker-compose up -d
```

### 訪問 localhost:3000

會看到初始畫面，可根據自己需求設定。

![1.png](/img/remote/gitea-workshop/1.png)

### 設定管理員帳號

![2.png](/img/remote/gitea-workshop/2.png)

## 安裝 Gitea Actions

在 Gitea 1.19 版本後，Gitea 就內建了 Actions 功能，可以在 Repo 內使用，但是預設是關閉的，所以要先開啟。

```
// Gitea-workshop\gitea\config\app.ini

[actions]
ENABLED = true

```

Docker restart 之後就可以在**設定**看到 Actions 功能了。

![3.jpg](/img/remote/gitea-workshop/3.jpg)

## 安裝 Runner

### 複製 Registration Token

![8.jpg](/img/remote/gitea-workshop/8.png)

### 設定 .env

```makefile
// Gitea-workshop\gitea-runner-1\.env
GITEA_INSTANCE_URL=<gitea service instance url>
GITEA_RUNNER_REGISTRATION_TOKEN=<runners registration token>
```

這邊要注意的是 `GITEA_INSTANCE_URL` 請用靜態 IP 或是 domain name，localhost 會無法使用。

### 啟動 Runner

```docker
docker-compose up -d
```

啟動後就可以在 Runners Management 內看到剛建立的 Runner

![4.jpg](/img/remote/gitea-workshop/4.jpg)
ref: [https://docs.gitea.com/zh-cn/usage/actions/quickstart](https://docs.gitea.com/zh-cn/usage/actions/quickstart)

## 啟用 Repo 的 Actions 功能

### 建立一個 Repo

可以用此 repo 內的 test-repo 來測試。介面與我們平常使用的工具十分相似，甚至也有可以 migrate 其他 codebase 平台的 repo。

![5.jpg](/img/remote/gitea-workshop/5.jpg)

![9.jpg](/img/remote/gitea-workshop/9.png)

### 開啟 Repo 內的 Actions 功能

![10.jpg](/img/remote/gitea-workshop/10.png)

![11.jpg](/img/remote/gitea-workshop/11.png)

## 測試

可以使用此 repo 內的 test-repo 推上去測試是否 CI 會進行。

![6.jpg](/img/remote/gitea-workshop/6.jpg)

## Notes

一開始是推 Python 專案且使用 pytest 做測試，但遇到 Version 3.11.4 was not found in the local cache 錯誤如圖。

![7.jpg](/img/remote/gitea-workshop/7.jpg)

安裝的 Python version 與 runner os 都符合 [versions-manifest.json](https://raw.githubusercontent.com/actions/python-versions/main/versions-manifest.json) 內的規定，但都會在 Set up Python 出錯，之後爬文到近日有人也提出類似的 [Bug](https://github.com/actions/setup-python/issues/585)，此 issue 也尚未被解決，所以改為使用 TypeScript 並使用 jest 做測試。
