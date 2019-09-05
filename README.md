# エンジニアブログ運用方法

## 社内エンジニアブログとは

下記URLで運用している社内エンジニアによる技術情報を主としたコンテンツです。

<https://dev.to/seattleconsulting>

目的は、

* 対社内
  * 誰がどんな技術に興味を持っているのかを表明・認識する場
  * 継続的なスキルアップと相互サポートができる場
* 対社外
  * 会社の中にこんなことができる人がいるというアピールの場
  * 会社としてこんなことをやっているというアピールの場

という場所として発展していけたら良いなと思っています。

## dev.toのOrganizationに参加する

以下の手順を実施してください。

1. GitHubアカウント作成
2. [ブログ運用用のGitHubリポジトリ](https://github.com/seattleconsulting/blog-contents)をclone
3. dev.toアカウント作成（GitHubアカウントと紐付け）
4. dev.toで自身のsettingsページのOrganizationで、メールで送ったシークレットを入力してOrganizationに参加（もらっていない人は誰かに転送してもらってください）

## 運用手順

ただ記事を書くだけであれば投稿するだけで良いのですが、相互サポートという意味でも、履歴管理とレビューのフローを入れたいと思います。
以下の流れです。

1. 記事を書く
2. 記事のレビュー依頼
3. レビュー指摘を修正・再レビュー
4. 記事を公開

## 記事を書く

まずはGitHubで記事を書きます。

1. 記事のURL＝ファイル名を英語で決めます。なんでも良いですが、出来るだけ内容が想像しやすいものが好ましいです。例：`hoge.md`
2. 公開したい対象年月を決めます。例：2019-09
3. cloneしたプロジェクトのcontentsフォルダ配下の、公開したい対象年月フォルダ配下に`hoge.md`を作ります。フォルダがなければ作成してください
4. gitでブランチを作ります。ブランチ命名規則は`feature/<記事ファイル名>` 作成コマンド例：`git checkout -b feature/hoge`
5. 記事をMarkdownで書きます。VSCodeなどのエディタでプレビューを見ながら書くのがオススメです。lintなどで構文エラーも出来るだけ無くしましょう
6. 記事をコミットします

## 記事のレビュー依頼

次にレビュー依頼＝Pull Request（以下PR）を出しましょう。

1. 作成したブランチ上で`git push origin <ブランチ名>`を実行
2. [GitHubリポジトリページ](https://github.com/seattleconsulting/blog-contents)を開いてPullRequestを作成（Slackの#blogチャネルにも通知が来る）。このとき、レビューして欲しいポイントを書くと良い
3. レビューを待つ

ただ待っているだけだと誰もレビューしてくれない可能性があるので、Slackでも促してみましょう。

## レビュー指摘を修正・再レビュー

レビューで指摘があれば修正しましょう。
修正が完了したら、再度レビューを依頼します。このとき同じPRを使っても良いですし、別PRを作っても良いです。

## 記事を公開

無事レビューが通ったら、dev.toで記事を公開しましょう。
書き方や、ヘッダ部分の説明は[この記事](https://dev.to/programmingmonky/devtomarkdown-4a2l)が詳しいので参考にしてください。

1. `WRITE A POST` を押す
2. Organizationを選択するプルダウンが表示され流ので`Seattle Consulting, Inc.`を選択
3. PRが通った記事Markdownをコピペ
4. ヘッダを `published: true` に変更
5. すでにQiitaや自分のブログなどで公開済みの記事の場合で、かつそちらが元ネタであると主張したい場合はヘッダに`canonical_url: <元記事URL>`を追加
6. `SAVE CHANGES`を押して公開！