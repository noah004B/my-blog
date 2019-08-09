---
title: "GithubのnpmパッケージのVlunerablility Alertを解決する"
date: 2019-08-09
published: true
tags: ['vuejs', 'yarn', 'npm']
series: false
canonical_url: false
description: "GithubからnpmパッケージのVulnerability Alertが来ました。解決するためにyarnコマンドを使ってnpmパッケージのアップグレードをする方法を残しておきます。"
---

Githubから以下のVlunerablility Alertが来ました。これをGithubの機能でサクッと解決する方法と、yarnコマンド使って手動で解決する方法を残しておきます。

## GithubのAutomated Security Fixの機能を使う

Vunnerablityの画面の「Create automated security fix」ボタンをクリックします。

![1565320795377](images/1565320795377.png)

少しすると自動的にこの問題を解決するPull requestが作成されます。

![1565321012635](images/1565321012635.png)

ちょっとビックリしたのが、Netlifyと連携する設定をしているせいか、自動でテストが行われるところ。

![1565321099472](images/1565321099472.png)

Merge pull requestをクリックするだけ。これなら手作業でやらないで任せてしまってもよさそう。すごい。

## 手動でパッケージをアップグレードするなら

```
$ yarn upgrade lodash@^4.17.13
```

https://yarnpkg.com/lang/ja/docs/cli/upgrade/