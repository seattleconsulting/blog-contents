---
title: Rails & Nuxt.jsのアプリケーションにGraphQLを導入する
published: false
description: 
tags: ruby, rails, nuxt.js, Graphql
canonical_url: https://qiita.com/kohbis/items/1b8d63e46d3d339a7df4
---

## 前提

[Rails & Nuxt.jsのDocker環境をalpineイメージで構築](https://dev.to/seattleconsulting/rails-nuxt-js-docker-alpine-1nf0)  
こちらのポストの環境をもとに進めるので、Dockerのサービス名等は適宜読み替えていただくようにお願いします。

ディレクトリ構成  
後述のコマンドでは、Railsは `backend`、Nuxtは `frontend` がDockerのサービス名になっています。

```text
.
├── backend <- Ruby on Rails
│   ├── Dockerfile
│   ├── Gemfile
│   ├── Gemfile.lock
│   (中略)
│   
├── frontend <- Nuxt.js
│   ├── Dockerfile
│   ├── README.md
│   ├── nuxt.config.js
│   ├── package-lock.json
│   ├── package.json
│  (中略)
│
├── docker-compose.yml
└── .env
```

## ライブラリ追加

### Rails

* [graphql-ruby](https://github.com/rmosolgo/graphql-ruby) ... RubyでGraphQLを導入するならコレ
* [graphiql-rails](https://github.com/rmosolgo/graphiql-rails) ... GraphQL IDE のRails版。

`./backend/Gemfile` を修正し、 `bundle install` します。

```ruby
## (中略) ##

gem 'graphql' #added

group :development do
  gem 'listen', '>= 3.0.5', '< 3.2'
  # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
  gem 'spring'
  gem 'spring-watcher-listen', '~> 2.0.0'

  gem 'graphiql-rails' #added
end

## (中略) ##
```

```bash
$ docker-compose exec backend bundle install
```

### Nuxt

* [apollo-module](https://github.com/nuxt-community/apollo-module)

こちらのライブラリをインストールします。  
本記事では [graphql-tag](https://github.com/apollographql/graphql-tag) は使いません。

```bash
$ docker-compose exec frontend yarn add @nuxtjs/apollo
```

## 実装

### Rails

[generator](https://github.com/rmosolgo/graphql-ruby/blob/master/guides/schema/generators.md) で雛形を作成します。

```bash
$ docker-compose exec backend rails g graphql:install
```

graphiql-railsの [Readme](https://github.com/rmosolgo/graphiql-rails#usage) にしたがって、Railsのconfigファイルを修正します。

#### `./backend/config/routes.rb`

GraphiQLエンジンをマウントし、ブラウザからアクセスできるようにします。

```ruby
Rails.application.routes.draw do
  post "/graphql", to: "graphql#execute" #generatorでinsertされる
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html

  #added
  if Rails.env.development?
    mount GraphiQL::Rails::Engine, at: '/graphiql', graphql_path: '/graphql'
  end
end
```

#### `./backend/config/application.rb`

APIモードの場合に必要な修正です。

```diff
## (中略) ##

- # require "sprockets/railtie"
+ require "sprockets/railtie"

## (中略) ##
```

#### GraphiQLで動作確認

dockerコンテナを再起動後、ブラウザで http://localhost:3000/graphiql にアクセスし、GraphiQLを開きます。  
`./backend/app/graphql/types/query_type.rb` のサンプルを利用して、下記のようにqueryの結果が返ってくればOKです。

```bash
$ docker-compose restart backend
```

<img width="569" alt="graphiql.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/123634/4555e512-5992-0f0e-baf6-202e7d00efa8.png">


### Nuxt

Nuxtアプリのルートに、下記のディレクトリ、ファイルを追加、編集します。  
今回はmutationは使いませんが、あわせて作成しておきます。

```
.
└── frontend 
    ├── nuxt.config.js
    │
    ├── pages
    │   └── index.vue
    │
    └── apollo
        ├── client-configs
        │   └── default.js
        └── gqls
            ├── mutations
            └── queries
                └── testField.gql
```

#### `./frontend/nuxt.config.js`

Nuxtでapolloクライアントを使用するための設定を追加します。  
`default.js` を読み込まず、nuxt.config.jsに直書きしてもOKです。

```js
export default {

  /* (中略) */

  modules: [
    '@nuxtjs/apollo', //added
  ],

  /* (中略) */

  apollo: {
    clientConfigs: {
      default: '~/apollo/client-configs/default.js'
    }
  }
}
```

#### `./frontend/apollo/client-configs/default.js`

apolloには様々なオプションありますがが、今回はqueryの実行が確認できればよいので最低限です。
uriのホスト名は、DockerのRailsアプリのサービス名 `backend` になります。

```js
import { HttpLink } from 'apollo-link-http'

export default () => {
  const httpLink = new HttpLink({ uri: 'http://backend:3000/graphql' })
  return {
    link: httpLink
  }
}
```

#### `./frontend/apollo/gqls/queries/testField.gql`

GraphiQLで実行したものです。

```
query {
  testField
}
```

#### `./frontend/pages/index.vue`

queryを実行した結果を表示します。

```html
<template>
  <p>{{ testField }}</p>
</template>

<script>
import testField from '~/apollo/gqls/queries/testField';

export default {
  data() {
    return {
      testField: {}
    }
  },
  apollo: {
    testField: {
      query: testField
    }
  }
}
</script>
```

<img width="500" alt="nuxt.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/123634/19b693c7-e9d7-b33b-bcf3-477f3456c923.png">

お疲れさまでした。  
(簡略化しすぎた感)
