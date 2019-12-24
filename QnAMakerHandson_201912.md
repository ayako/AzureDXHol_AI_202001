# Microsoft Cognitive Services & Azure Bot Framework を利用した FAQ チャットボット 開発 (201912 版)

FAQ のWebページや リストから Q&A 返答エンジンが作成できる [Cognitive Services QnA Maker](https://azure.microsoft.com/ja-jp/services/cognitive-services/qna-maker/) および
ノンコーディングでチャットボットを作成、ホストできる [Azure Bot Service](https://azure.microsoft.com/ja-jp/services/bot-service/) を用いて、FAQ チャットボットを作成する方法を紹介します。

## 目次

0. 準備
1. Cognitive Services QnAMaker で Q&A 回答エンジンを作成する
2. QnAMaker を利用する Azure Web App Bot を作成する


## 0. 準備

### Microsoft アカウント & Azure サブスクリプション

- Microsoft アカウント
    - Azure サブスクリプション申し込みに必要です。
    - [Microsoft アカウント登録手続き](https://www.microsoft.com/ja-jp/msaccount/signup/default.aspx)
- Azure サブスクリプション
    - 無料試用版で充分です。上記↑で取得した Microsoft アカウントで申し込みを行います。(無料のプランがあるので、従量課金プランでも大丈夫ですが念のため。)
    - [Azure 無料アカウントを今すぐ作成](https://azure.microsoft.com/ja-jp/free/)
- Web API ツール
    - HTTP リクエストを送信できるツールがあると、Web APIの挙動やレスポンスを確認できて便利です
    - 詳しくは [補足](https://qiita.com/annie/items/7b63e366deeaeae63f94#%E8%A3%9C%E8%B6%B3-web-api-%E3%83%84%E3%83%BC%E3%83%AB%E3%81%AE%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95) をご覧ください


## 手順

## 1. Cognitive Services QnAMaker で Q&A 回答エンジンを作成する

### 1.1. QnAMaker のサブスクリプション作成

ブラウザーで [Azure Portal](https://portal.azure.com) (https://portal.azure.com) を開き、Azure サブスクリプションが紐づいている Microsoft アカウントでサインインします。
Azure Portal から左端ナビゲーションバーから **[+リソースの作成]** をクリックします。
*新規* ペインの検索欄に **QnAMaker** と入力し、表示される **QnAMaker** をクリックします。*QnAMaker* ペインで **[作成]** をクリックして作成に進みます。
![20190314_01.PNG](https://qiita-image-store.s3.amazonaws.com/0/117583/5e5eb44f-2c38-a79d-d18c-54dfe5c2f5ff.png)


*QnAMaker* ペインで必要事項を設定します。

- Name : 識別できるものを (ここでは QnAMaker としています)
- サブスクリプション : (自動入力)
- Pricing Tier : **F0** の無償版でOKです
- 場所 : 米国西部
- Resource Group : 見分けのつくお好みのところを。東日本(JapanEast) or 西日本(JapanWest) など
- Search Pricing Tier :  **F** の無償版でOKです
- Search location : 東日本
- App name : 基本的には Name に入力したものが自動で入力されます。azurewebsites.net 内でユニークになる必要があるため、必要に応じて修整します
- Website location : 東日本
- App Insights : デフォルトで On になっていますが、Offにしても構いません

*QnAMaker* ペイン の一番下にある **[作成]** をクリックするとサブスクリプションが作成されます。
サブスクリプションの作成(デプロイ)が成功した旨のメッセージが表示されたら、QnAMaker のサブスクリプション作成は完了です。
![20190314_02.PNG](https://qiita-image-store.s3.amazonaws.com/0/117583/00cc32d3-d0f1-9c52-3aa4-bf109d25cfd9.png)


### 1.2. QnAMaker で Q&A 回答エンジン作成

#### ナレッジベース (KB: knowledgebase) の作成

QnAMaker のポータルで Q&A 回答エンジンを作成していきます。

ブラウザーから [QnAMaker ポータル](https://qnamaker.ai/)(https://qnamaker.ai/) にアクセスします。ページ右上の **Sign In** をクリックして、1.1 の手順で QnAMaker サブスクリプションを作成したマイクロソフトアカウントでサインインします。

![20190314_03.PNG](https://qiita-image-store.s3.amazonaws.com/0/117583/714d7bf5-aa4e-ec18-42be-844064b0a586.png)

サインインできたら、上部メニューバーにある **Create a knowledge base** をクリックして、ナレッジベースを作成します。

![20190314_04.PNG](https://qiita-image-store.s3.amazonaws.com/0/117583/2a0ce10b-238a-07b1-5169-20eb68664268.png)

*Step 1* はスルーして (1.1 で作成済み)、*Step 2* で Azure サブスクリプションと作成済みの QnAMaker サブスクリプションを選択します。
*Step 2* で Azure サブスクリプションと作成済みの QnAMaker サブスクリプションを選択します。

![20190314_05.PNG](https://qiita-image-store.s3.amazonaws.com/0/117583/d613532a-b515-6ea0-77fc-6ebe1ec89a48.png)

*Step 3* 以降ででナレッジベースの情報を登録していきます。

- Name : 識別できるものを (ここでは AzureSupportFAQBot としています)

*Step 4* でナレッジベースの元となる Q&A データを登録します。
今回はこちらのデータソース(テキストファイル、タブ区切り＆UTF-8)を使います。(ローカルにダウンロードしてお使いください。)

[データソース(テキストファイル)](https://raw.githubusercontent.com/a-n-n-i-e/Azure-LineBot/master/WhatCatsCanEatBot/WhatCatsCanEat.txt)  

> ※ こちらは以下の [ペトこと 様 Web サイト](https://petokoto.com/) で公開されている情報を編集したものです。今回のハンズオン手順を確認する範囲のみでご利用ください。(**このデータソース内容による、いかなるトラブル・損失・損害に対して責任を負いません。**)
        - [猫が食べて良いものまとめ　野菜・果物・穀物など主な栄養素や期待される効果](https://petokoto.com/1400)
        - [猫が食べてはいけないものまとめ！　特に危険な食べ物・植物を解説](https://petokoto.com/412)

> QnAMaker では FAQページ (html), PDFファイル, テキスト (タブ区切り) などをデータソースとして利用できます。

**[Create KB]** をクリックして、ナレッジベースを作成します。
![20190314_06.PNG](https://qiita-image-store.s3.amazonaws.com/0/117583/06db90e3-22e8-1712-14a3-5aa0b527eb22.png)

登録したデータソースからナレッジベース (Q&Aの一覧) が作成されていれば OK です。
![20190314_07.PNG](https://qiita-image-store.s3.amazonaws.com/0/117583/808adc21-030d-2efd-aa38-a80192ec5356.png)

#### ナレッジベース の Q&A 追加、学習、テスト

ナレッジベースを確認して、不要な改行コードなどのゴミがあれば修正、削除します。
ナレッジペースに直接 Q&A の組を追加することも可能です。「こんにちは」と入力すると説明を返答するようにします。

**+ Add Q&A pair** をクリックして、Q&A の組を追加します。
*Question* の列にユーザーからの入力(今回は「こんにちは」)、*Answer* の列に返答したい内容(今回はBotの説明) をそれぞれ入力します。
入力が完了したら、上部メニューバー(2段目)の **[Save and train]** をクリックして、Q&A 回答エンジンに学習させます。
![20190314_08.PNG](https://qiita-image-store.s3.amazonaws.com/0/117583/580d217e-fc7c-18f3-dc76-1c0cecb81ea0.png)

学習が終了したら、上部メニューバー(2段目)の **[Test]** をクリックして、QnA 回答エンジンをテストします。
Question に含まれている内容を入力して、対応する Answer が回答されるのを確認してください。

> Q&A の追加、修整を行ったら、[Save and train] をクリックして学習を行ってください。(学習するまで内容が反映されません)

![20190314_09.PNG](https://qiita-image-store.s3.amazonaws.com/0/117583/e84a6423-fc62-1f5a-2ddf-c781a922d860.png)

もう一度 **[Test]** をクリックすると、Knowledge Base の画面に戻ります。


#### ナレッジベース の発行

上部メニューバー(2段目)の **[Publish]** をクリックして、このナレッジベースに Web API 経由でアクセスできるように発行します。

*Publish* ページで **[Publish]** をクリックして発行します。
![20190314_10.PNG](https://qiita-image-store.s3.amazonaws.com/0/117583/1b452000-71fe-18fd-6c79-e0461fc16ed4.png)

Web API でアクセスするための情報が表示されれば、発行は完了です。(このページは閉じないでおいてください。2. のステップで利用します)
![20190419_01.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/117583/8f464c49-d608-ffd4-6dd5-c06c37aeceb7.png)

```
POST /knowledgebases/YOUR_KNOWLEDGEBASE_ID/generateAnswer
Host: https://YOUR_SERVER.azurewebsites.net/qnamaker
Authorization: EndpointKey YOUR_ENDPOINT_KEY
Content-Type: application/json
{"question":"<Your question>"}
```

このような情報が表示されるので、以下の部分をコピーしてローカルに保存しておきます。

- Knowledge Base ID (KbId): **YOUR_KNOWLEDGEBASE_ID**
- (QnAMaker) Endpoint: **https://YOUR_SERVER.azurewebsites.net/qnamaker**
- (QnAMaker) Endpoint Key: **YOUR_ENDPOINT_KEY**

Postman などの Web API ツールを使って QnAMaker API の動作を確認します。

> Web API ツールの使い方 は [補足: Web API ツールの使用方法](https://qiita.com/annie/items/7b63e366deeaeae63f94#%E8%A3%9C%E8%B6%B3-web-api-%E3%83%84%E3%83%BC%E3%83%AB%E3%81%AE%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95) をご覧ください。

- Method: POST
- URL: https://YOUR_SERVER.azurewebsites.net/qnamaker/knowledgebases/YOUR_KNOWLEDGEBASE_ID/generateAnswer
- Parameter:
    - Header:
        - Authorization = EndpointKey YOUR_ENDPOINT_KEY
        - "**EndpointKey**" をつけるのを忘れずに
    - Content-Type=application/json
- Body: `{"question":"こんにちは"}` というフォーマット(Json)に構成します

Web API リクエストを送信して、Response 200、Body から回答が取得できれば OK です。
![20190314_12.PNG](https://qiita-image-store.s3.amazonaws.com/0/117583/c49f3fc9-f39c-a263-b2f4-6a437a24cd42.png)


## 2. QnAMaker を利用する Azure Web App Bot を作成する

ナレッジベース 発行完了画面で、**[Create Bot]** をクリックします。
![20190419_01.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/117583/8f464c49-d608-ffd4-6dd5-c06c37aeceb7.png)

Azure Portal が開き、Web App Bot の作成画面になります。
*Web アプリ ボット* ペインで必要事項を設定します。
![20190419_02.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/117583/56d726c4-4c1d-8424-4208-84918e22c820.png)

- ボット名
    - お好みの名前を入力します。こちら Azure Bot Service 全体で一意となる名前になるような名前を設定してください。一意となる名前を入力すると ✓ (チェックマーク) が表示されます
- サブスクリプション (※デフォルトのまま)
- リソースグループ
    - お好みのグループを新規作成 or 既存のリソースグループを選択します 
- 場所
    - Bot アプリを配置するデータセンターロケーションを選択します。ここでは **Japan East** (東日本) を選択しています
- 価格レベル
    - 開発や検証では F0 (無償版) で充分です
- アプリ名 (※自動入力)
    - 基本的にボット名と同じ名前が自動入力されますが、こちらも azurewebsites.net で一意となる必要があるため、適時修整してください
- SDK 言語
    - C# or Node.js (お好みの方を選択)
- QnA Auth Key (※デフォルトのまま)
- App Service プラン
    - これまでに Azure App Service を作成している場合はそちらが自動設定されている場合があります。そのまま利用する or 新規作成を行ってください
- Application Insights
    - アプリの稼働ログを取得するサービスです。お好みですが、ここではデフォルトのまま **オン** になっています。
- Application Insights の場所
    - Application Insights をオンにした場合に表示されます。ここでは **Japan East** (東日本) を選択しています。
- Microsoft アプリ ID とパスワード
    - デフォルトのまま **アプリIDとパスワードの自動作成** で OK です

*Web アプリボット* ペイン の一番下にある **[作成]** をクリックするとアプリが作成されます。

アプリの作成(デプロイ)が成功した旨のメッセージが表示されたら、[リソースに移動] をクリックして作成した Web アプリボット を表示します。
![20190308_04.PNG](https://qiita-image-store.s3.amazonaws.com/0/117583/50531c13-95e8-ba60-ab3d-847575bb8de7.png)

> App Service プランを新規作成した場合、プランが S1 の有償プランになっています。[すべての App Service 設定] ＞ [スケールアップ] をクリックして 無償プラン(F1) に変更しておくと良いでしょう。
![20190308_06.PNG](https://qiita-image-store.s3.amazonaws.com/0/117583/d0934686-a8ab-873f-cf51-db3720c6fd2c.png)

作成した *Web アプリ ボット* ペインのメニューから **Web チャットでテスト** をクリックします。チャット画面が表示されるので、メッセージを入力すると、QnA Maker のナレッジベースからメッセージが返答されれば Web アプリボットは正常に作成されています。
![20190419_03.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/117583/88e63d36-b2b7-744b-367d-f72bfade6140.png)
