# Swagger Docs

SP-API提供のOpenAPIスペックをレンダーするためのツール


## セットアップ

```sh
git clone https://bitbucket.com/xxxxx swagger-ui

cd swagger-ui

docker compose up docs
```

# アクセス

`http://localhost:8080`


# スペック追加

1. OpenAPIスペックファイルを `./specs` に追加
2. コンポーズファイルの環境変数にスペックのリンクを追加
3. コンテナーを再起動

```yml
# compose.yaml
services:
  docs:
    environment:
      PORT: 8080
      URLS: |
        [
          { url: "/specs/orders_2026-01-01.json", name: "Orders API" },
          { url: "/specs/ファイル名.json", name: "ページタイトル" },
          ...
        ]
...
```

# 参照
[SP-API 全モデル](https://github.com/amzn/selling-partner-api-models)
