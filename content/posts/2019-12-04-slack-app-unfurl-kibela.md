---
title: "Kibela のリンクを Slack に貼ったときに展開されるようにした"
slug: slack-app-unfurl-kibela
date: 2019-12-04T23:30:00Z
categories: 
- "Tech"
tags: 
- "kibela"
- "slack"
- "golang"
---

この記事は [BBSakura Networks Advent Calendar 2019](https://adventar.org/calendars/4517) の4日目の記事です。

[BBSakura Networks](https://www.bbsakura.net/)では[Kibela](https://kibe.la/ja)を使っていて、Slackでは日々Kibelaのリンクが飛び交っているのですが、リンクを貼っても展開されないため、何のドキュメントなのかわからず不便でした。
そこで、KibelaのリンクをSlackに貼ったときに展開されるようにするSlack Appを作ったので、使い方を書いておきます。
ソースコードは [https://github.com/higebu/slack-app-unfurl-kibela](https://github.com/higebu/slack-app-unfurl-kibela) に置いてあります。
<!-- PELICAN_END_SUMMARY -->

以下、手順です。

# Kibelaのアクセストークンを取得する

以下のURLにアクセスし、 `アクセストークンの作成` ボタンをクリックすると作成できます。
権限は `read` のみで大丈夫です。

[https://my.kibe.la/settings/access_tokens](https://my.kibe.la/settings/access_tokens)

# Slack Appを作成する

以下の手順で作成します。

1. [https://api.slack.com/apps](https://api.slack.com/apps) を開く
2. `Create New App` ボタンをクリック
3. 名前を入れて、 `Create App`
4. `Event Subscriptions` をクリック
5. `Enable Events` を `On` にする
6. `OAuth & Permissions` を開き、 `Scopes` で `link:write` を追加する
7. `App unfurl domains` を展開し、 `Add Domain` で、 `{TEAM_NAME}.kibe.la` を入力し、 `Save Changes`
8. 左メニューから `Install App` を開き、 `Install App to Workspace` -> `Allow`
9. OAuth Access Token が表示されるのでメモしておく

※後で戻ってくるので、Slack Appの管理画面は開いたままにしておきます。

# slack-app-unfurl-kibelaをデプロイ

## Herokuにデプロイ

下記のボタンからデプロイできます。

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy/?template=https://github.com/higebu/slack-app-unfurl-kibela)

`App name` に入れた名前がURLになります。具体的には `https://{app name}.herokuapp.com/` のようになります。

`KIBELA_TEAM` にはKibelaのチーム名、 `KIBELA_TOKEN` にKibelaのアクセストークン、 `SLACK_TOKEN` にSlackのOAuth Access Tokenを入力し、 `Deploy App` ボタンを入力してください。

## GCPのCloud Runにデプロイ

```shell
KIBELA_TEAM=xxxxxx
KIBELA_TOKEN=xxxxxx
SLACK_TOKEN=xxxxxx
PROJECT_ID=xxxxxx
REPO=docker-repo
IMAGE=slack-app-unfurl-kibela

git clone https://github.com/higebu/slack-app-unfurl-kibela.git
cd slack-app-unfurl-kibela

gcloud projects create $PROJECT_ID
gcloud config set project $PROJECT_ID
echo "$KIBELA_TEAM" | gcloud secrets create KIBELA_TEAM --data-file -
echo "$KIBELA_TOKEN" | gcloud secrets create KIBELA_TOKEN --data-file -
echo "$SLACK_TOKEN" | gcloud secrets create SLACK_TOKEN --data-file -

gcloud artifacts repositories create docker-repo \
    --repository-format=docker \
    --location asia-northeast1

gcloud builds submit \
    --config=cloudbuild.yaml \
    --substitutions=_REPOSITORY="$REPO",_IMAGE="$IMAGE" .
```

リポジトリの更新時に自動でデプロイするためにトリガーの設定をします。
※フォークしたリポジトリを使う必要があります。

```shell
OWNER=xxxxxx
gcloud beta builds triggers create github \
    --name=trigger-slack-unfurl-kibela \
    --repo-name=slack-app-unfurl-kibela \
    --repo-owner=$OWNER \
    --branch-pattern='^master$' \
    --build-config=cloudbuild.yaml \
    --substitutions=_REPOSITORY="$REPO",_IMAGE="$IMAGE"
```

# Slack AppにURLを登録

1. `Event Subscriptions` を開き、 `Request URL` に `https://{app name}.herokuapp.com/` を入力する
2. `Verified` と表示さたら `Enable Events` を `On` にして `Save Changes`

# 動作確認

SlackでKibelaのリンクを含んだメッセージを投稿してみてください。
下記のように展開されるはずです。

![slack-unfurl-kibela](/images/20191204-slack-unfurl-kibela.png)