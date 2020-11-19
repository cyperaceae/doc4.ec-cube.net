---
title: 動的スキャンの仕方
permalink: /penetration-testing/testing/active_scan
---
## 動的スキャンの基本

基本は、以下の流れで動的スキャンを実施します。

1. 履歴または、サイトタブのURL一覧から、スキャン対象の URL を選択する
1. **攻撃→動的スキャン** を選択し、実行する

![動的スキャンの基本操作](/images/penetration-testing/testing_active_scan.png)

**Note:** POSTパラメータに設定された CSRF トークンが、 Local Proxes 経由でアクセスしている Firefox のものと一致していることを確認しましょう。
![CSRFトークンの一致を確認](/images/penetration-testing/testing_active_scan_csrftoken.png)
CSRFトークンが一致してない場合は、動的スキャンがエラーになってしまい、十分にテストができません。
{: .notice--warning}

## GET でアクセス可能な URL

GET でアクセス可能な URL は、 CSRF トークンによる影響を受けませんので、サイトタブの URL 一覧から、ディレクトリを選択し、下位階層をまとめて動的スキャンすることが可能です。

![下位階層をまとめて動的スキャン可能](/images/penetration-testing/testing_active_scan_get.png)

## 特殊な遷移パターンのテスト

**Note:** まだ書きかけです
{: .notice}

### 複数画面遷移のテスト

- 日本語入力必須の場合は、1画面ずつ **手動探索→動的スキャン** を繰り返すのが確実
- 日本語入力が絡まなければ、 sequence アドオンが使用できる可能性
  - うまくいかない場合もあるため、確実にスキャンできているかよく確認すること

### コンテンツが削除されてしまう場合

- 画像削除、カートから削除などが該当する
- 削除のロジックをコメントアウトしたり、DBをロールバックしたりしてテスト可能
- ただし、削除のロジックに SQL インジェクションが潜んでいた場合など、検出できなくなる可能性がある

### POST パラメータで遷移制御している場合

- サイトタブのURL一覧には、 POSTパラメータの値が異なるURLは登録されない
- 履歴から該当のリクエストを探すか、一旦サイトタブの URL 一覧から削除して、1画面ずつ動的スキャンする必要がある

**Note:** お問い合わせ画面、会員登録画面などが該当します。
{: .notice--info}

### セッションでデータの引き継ぎをしている場合

- 確認画面→完了画面ではセッションでデータの引き継ぎをしている場合はテストできない
  - ZAP Script を組むことでテストできる可能性あり

**Note:** 入力画面で入力した情報を、完了画面で表示するカスタマイズや、プラグインを導入している場合は、そこに脆弱性が潜んでいる可能性に注意してください
{: .notice--warning}

**Note:** 商品購入完了画面が該当します。
{: .notice--info}

### ファイルアップロードのテスト

**手動探索でファイルアップロード→動的スキャン** をすることで、ファイルアップロードのテストが可能です。
大きなサイズのファイルをアップロードすると、多大な時間がかかりますので、できるだけ小さなサイズのファイルでテストすることをおすすめします。

### 削除系機能のテスト

注文や会員を削除するテストは、繰り返し削除のリクエストを送信することができないため、テストが困難です。

トランザクションをロールバックさせたり、 `EntityManager::flush()` をコメントアウトしたりすることで、削除機能を無効化して、ある程度のテストは可能です。

削除機能を無効化すると、SQLインジェクションのテストが難しいことには変わりありませんので、削除時に何らかのパラメータを送信している場合は、パラメータの入力に脆弱性が潜んでないか注意しましょう。