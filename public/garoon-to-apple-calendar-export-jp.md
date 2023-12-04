---
title: ガルーン イベントを Apple カレンダーにエクスポートするブックマークレット
tags:
  - JavaScript
  - Apple
  - bookmarklet
  - garoon
  - icalendar
private: true
updated_at: '2023-12-04T16:44:13+09:00'
id: 18308ff3a6daad9029a4
organization_url_name: null
slide: false
ignorePublish: false
---

ガルーン（Garoon）からAppleカレンダーにイベントの詳細をコピペするのは面倒ですよね。  
このブックマークレットは、開いている任意のガルーンイベントからiCalファイルを簡単に生成できます。

* 🇬🇧 English version: [Import a Garoon Event to Apple Calendar Bookmarklet - Qiita](https://qiita.com/ahandsel/items/0e8d71aef698a9804944)


## 目次 <!-- omit in toc -->

* [使い方](#使い方)
  * [初期設定](#初期設定)
  * [ガルーンイベントをAppleカレンダーにエクスポート](#ガルーンイベントをappleカレンダーにエクスポート)
* [動作しない場合 🤔](#動作しない場合-)
* [garoon-to-apple-bookmarklet.js](#garoon-to-apple-bookmarkletjs)
* [ブックマークレットとは？](#ブックマークレットとは)
* [コードの解説](#コードの解説)
  * [コードをIIFEで囲む](#コードをiifeで囲む)
  * [ガルーンイベントオブジェクトを取得する](#ガルーンイベントオブジェクトを取得する)
  * [入力を確認する](#入力を確認する)
  * [メイン関数 - addCalendar](#メイン関数---addcalendar)
  * [開始時刻と終了時刻の書式設定](#開始時刻と終了時刻の書式設定)
  * [オリジンURLの変更](#オリジンurlの変更)
  * [イベントURLの作成](#イベントurlの作成)
  * [URLパラメーターの初期化](#urlパラメーターの初期化)
  * [イベントメモの書式設定](#イベントメモの書式設定)
  * [URLパラメーターの設定](#urlパラメーターの設定)
  * [終日イベントの処理](#終日イベントの処理)
  * [カレンダーイベントURLの作成と開く](#カレンダーイベントurlの作成と開く)
  * [デバッグのためのログ記録](#デバッグのためのログ記録)
* [ブックマークレットは役に立ちましたか?](#ブックマークレットは役に立ちましたか)
* [参考資料](#参考資料)


## 使い方


### 初期設定

1. 下の`garoon-to-apple-bookmarklet.js`コードブロックをコピーします。
1. Chromeのアドレスバーに `@bookmarks` と入力します。
1. 右上のその他アイコン `⋮` をクリック
1. `ブックマークを追加`をクリックし、URL フィールドにコードを貼り付けます。


### ガルーンイベントをAppleカレンダーにエクスポート

1. ガルーンイベントのページに移動します。
1. ブックマークレットをクリックします。
1. 生成されたiCalファイルをダウンロードします。
1. ファイルを開き、イベントがAppleカレンダーアプリに追加されていることを確認します。


## 動作しない場合 🤔

1. ブラウザのコンソールを開きます。
    * Mac: `Command+Option+C`
    * Windows、Linux、Chrome OS: `Control+Shift+C`
2. エラーメッセージが表示されているか確認します。


## garoon-to-apple-bookmarklet.js

```javascript
javascript: (() => {
  const formatTimestamp = (dateString) =>
    new Date(dateString).toISOString().replaceAll(/[-:]|\.\d+/g, '');
  const bodyFormat = (inputText) => inputText.replace(/\n/g, '\r');
  const addCalendar = (event) => {
    console.log({ event });
    const start = formatTimestamp(event.start.dateTime);
    const end = formatTimestamp(event.end.dateTime);
    const origin = location.origin.replace(".s.", ".");
    const url = `${origin}${location.pathname}?event=${event.id}`;
    const params = new URLSearchParams({ service: "apple" });
    const body = bodyFormat(event.notes);
    params.set("start", start);
    params.set("end", end);
    params.set("title", event.subject);
    params.set("description", body);
    params.set("location", url);
    params.set("timezone", event.start.timeZone);
    params.set("calname", `${start}-${event.id}`);
    if (event.isAllDay) { params.set("all_day", "true"); }
    console.log(params.toString());
    open(`https://calndr.link/d/event/?${params.toString()}`);
  };

  const event = window.garoon?.schedule?.event?.get();

  if (event === undefined) {
    alert(
      `エラー: ガルーンのスケジュールではありません。\n特定のガルーンイベントを開いてください。`
    );
    return;
  }

  addCalendar(event);
})();
```


## ブックマークレットとは？

ブックマークレット (Bookmarklet) は、ウェブブラウザのブックマークに保存することができる小さなJavaScriptプログラムの一種です。

ブックマークと同様に、ブラウザのブックマークバーまたはメニューに保存され、クリックすることで特定のアクションを実行します。


## コードの解説

コードがどのように動作するのかを詳しく見ていきましょう。


### コードをIIFEで囲む

まず、コードの言語として `JavaScript` を指定する必要があります。

次に、ブックマークレットはグローバルスコープで実行されるため、 [即時実行関数式 (IIFE)](https://developer.mozilla.org/ja/docs/Glossary/IIFE)でコードを囲むことで、グローバルスコープと同様に変数へアクセスすることを防ぎます。

```javascript
javascript: (() => {
  // ... (コードスニペット)
  addCalendar(event);
})();
```


### ガルーンイベントオブジェクトを取得する

[`garoon.schedule.event.get()`](https://cybozu.dev/ja/garoon/docs/js-api/schedule/get-schedule-event/) JavaScript APIを使用して、開いているガルーンイベントのイベントオブジェクトを取得します。

[`window`ウェブAPI](https://developer.mozilla.org/ja/docs/Web/API/Window)により、グローバルスコープから`garoon`オブジェクトにアクセスすることが可能です。

```javascript
const event = window.garoon?.schedule?.event?.get();
```


### 入力を確認する

コードを進める前に、`event`オブジェクトが`undefined`でない（開いているページがガルーンのイベントページである）ことを確認しておきましょう。

```javascript
const event = window.garoon?.schedule?.event?.get();

if (event === undefined) {
  alert(`エラー: ガルーンのスケジュールではありません。\n特定のガルーンイベントを開いてください。`);
  return;
}
```


### メイン関数 - addCalendar

関数は `event` オブジェクトを引数として受け取り、URL を準備するためのいくつかの操作を実行します。 この URL が開くと、アップルカレンダー の新しいイベントのフィールドに自動的に入力されます。


### 開始時刻と終了時刻の書式設定

イベントの開始日時文字列と終了日時文字列を必要な形式に変換するには、`formatTimestamp`ヘルパー関数を使用します。

```javascript
const start = formatTimestamp(event.start.dateTime);
const end = formatTimestamp(event.end.dateTime);
```

`formatTimestamp`ヘルパー関数は日付文字列を受け取り、それを ISO 8601 にフォーマットして、ハイフンとコロンを取り除きます。

```javascript
const formatTimestamp = (dateString) =>
  new Date(dateString).toISOString().replaceAll(/[-:]|\.\d+/g, '');
```


### オリジンURLの変更

`location.origin`を使って現在のページの[Origin (オリジン)](https://developer.mozilla.org/ja/docs/Glossary/Origin) URLを取得します。

[cybozu.comのセキュアアクセス](https://jp.cybozu.help/general/ja/id/02047.html)機能は、サブドメインとドメインの間に `.s` を追加してURLを変更するため、エクスポートする前にこれを取り除きましょう。

```javascript
const origin = location.origin.replace(".s.", ".");
```


### イベントURLの作成

オリジンURLとイベントIDを組み合わせて短いイベントのURLを生成します。

```javascript
const url = `${origin}${location.pathname}?event=${event.id}`;
```


### URLパラメーターの初期化

`URLSearchParams` オブジェクトを初期化し、サービスとして "apple" を設定します。

```javascript
const params = new URLSearchParams({ service: "apple" });
```


### イベントメモの書式設定

`bodyFormat` ヘルパー関数を呼び出し、イベントメモを特定のフォーマットに変換します。

```javascript
const body = bodyFormat(event.notes);
```

この関数は文字列を取り、すべての改行をキャリッジリターンに置き換えます。

```javascript
const bodyFormat = (inputText) => inputText.replace(/\n/g, '\r');
```


### URLパラメーターの設定

Appleカレンダーイベントに必要なパラメーターを設定します。

```javascript
params.set("start", start);
params.set("end", end);
params.set("title", event.subject);
params.set("description", body);
params.set("location", url);
params.set("timezone", event.start.timeZone);
params.set("calname", `${start}-${event.id}`);
```


### 終日イベントの処理

イベントが終日の場合は、`all_day` パラメーターに `true` の値を追加します。

```javascript
if (event.isAllDay) { params.set("all_day", "true"); }
```


### カレンダーイベントURLの作成と開く

[calndr.link](https://calndr.link/) は、[atymic](https://atymic.dev/) によって作成された無料のカレンダーリンクジェネレーターです！

パラメーターをCalndrの[ダイナミックAPI](https://calndr.link/api-docs#dynamic)に渡し、iCalファイルをダウンロードします。

```javascript
open(`https://calndr.link/d/event/?${params.toString()}`);
```


### デバッグのためのログ記録

デバッグに役立てるために、スクリプトの入力と出力がユーザーに見えるように、`event` オブジェクトと `params` オブジェクトをコンソールにログ記録します。

```javascript
console.log({ event });
// ...
console.log(params.toString());
```

---

以上です。簡単なスクリプトですよね〜


## ブックマークレットは役に立ちましたか?

**calndr.link**を開発した[atymic](https://atymic.dev/)さんに[コーヒーを買って上げませんか？](https://ko-fi.com/slashdev)


## 参考資料

* [CalndrのAPIドキュメント](https://calndr.link/api-docs#dynamic)
* [スケジュールオブジェクト - cybozu developer network](https://cybozu.dev/ja/ガルーン/docs/overview/schedule-object/)
* [Garoon_to_Apple_Bookmarklet_v0.js - GitHub](https://github.com/ahandsel/articles/blob/master/Garoon_to_Apple/Garoon_to_Apple_Bookmarklet_v0.js)
* [iCalのRFC 5545仕様](https://datatracker.ietf.org/doc/html/rfc5545)
