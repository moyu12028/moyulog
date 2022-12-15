---
title: ラズパイ4でクラスタを作りたい。Part1
tags: RaspberryPi クラスタ Ubuntu kubernetes
author: moyu12028
slide: false
---
# はじめに
* 最近流行りのDockerやKubernetesに触りたいという好奇心
* Raspberry Piでスーパーコンピュータを作る海外の動画を見て見た目がかっこいい(ロマン)

これらの理由がラズパイでクラスタを作りたいと思った経緯です。
Docker触るならWSLでもいいじゃんという反論がありますが無視します。実物あってのロマンです(？)

手探りで始めるので知識を共有というより日記程度の記事なので参考に至らない部分や見づらい部分があるかと思いますが、温かい目で見てもらえると幸いです。

# 目次

1. 使ったもの
   1. これらのものを使った理由
   1. 失敗談・改善点
   1. これらの製品を選ぶにあたって悩んだこと
1. 使用する環境
1. 環境構築
1. 参考書籍・参考リンク
1. 続き

# 使ったもの

2022年4月現在の構成のため今後ラズパイの台数を増やす予定です。
|分類名|商品名|値段(円)|リンク|
|-----|------|---|-----|
|ラズパイ|Raspberry Pi4 8GB|11440*2|[リンク](https://www.switch-science.com/catalog/6370/)|
|ケース|GeeekPi Raspberry Piクラスターケース|7599|[リンク]()|
|ストレージ|SanDisk 2.5インチ 240GB SSD|4568|[リンク](https://www.amazon.co.jp/gp/product/B01F9G43WU/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&th=1)|
|ストレージ|KIOXIA 2.5インチ 240GB SSD|3470|[リンク]()|
|電源ケーブル|Smraza Raspberry Pi 4 USB-C (Type C）電源 5V 3A|1080*2|[リンク](https://www.amazon.co.jp/gp/product/B07DN5V3VN/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1&language=ja_JP)|
|ヒートシンク|waves Raspberry Pi4 ヒートシンク 4点セット|498*2|[リンク](https://www.amazon.co.jp/gp/product/B083TMVKKW/ref=ppx_yo_dt_b_asin_title_o01_s00?ie=UTF8&th=1)|
|ハブ|TP-Link 8ポート スイッチングハブ|3412|[リンク](https://www.amazon.co.jp/gp/product/B00A121WN6/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1)|
|LANケーブル|エレコム LANケーブル CAT6A CAT6A|-------|[リンク](https://www.amazon.co.jp/gp/product/B088P7Z2KM/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&th=1)|
|SATA to USBケーブル|Groovy USB SATAUD-3101|996|[リンク](https://shop.tsukumo.co.jp/goods/4943508515354/)|
|その他|PC ダストフィルター|1198|[リンク](https://www.amazon.co.jp/gp/product/B088PCCJJ9/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)|

## これらの製品を使った理由 
* ラズパイ
Jetson Nanoと比べてラズパイの単価が安価だという事。
情報量多い事
値段の割に高性能だということ
消費電力が低い事
これらを理由にこちらを選びました
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1275928/27e7644e-aa43-8ee4-3080-f144b29c0efe.png)

* ケース
個人的に見た目がかっこいい
ファンがついてる(排熱的な問題)
ラズパイのほかにJetson Nanoや2.5インチSSD・HDDを載せることが出来る
合計12台のラズパイやJetson NanoやSDD・HDDを載せることが出来る。
これらを理由にこちらを選びました
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1275928/567aac0d-44d9-e538-1296-fa76a45b39ab.png)

* ストレージ
ストレージには元東芝メモリのキオクシアとWD傘下のSanDiskのSSDを使っています。
2種類ある理由は余ったSSDの流用とキオクシアが安かったという理由です。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1275928/bd22e4ad-8d09-4970-f864-7e59e6dfa959.png" width="40%">
<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1275928/268d259a-d1d9-dd47-c9a1-741e0d5a0a1c.png" width="40%">

* 電源ケーブル
ラズパイの電源供給に関しては、「これらの製品を選ぶにあたって悩んだこと」にて解説します。
![61Iaz4sqdaS._AC_SX569_.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1275928/a6617eca-5fa6-bab6-fb4e-f5a5da039dd6.jpeg)

* ヒートシンク
値段です！中華製なので効果あるのか知らんけど！(金属ではなく樹脂っぽい)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1275928/c6708704-3426-82da-8c7c-f4dd0b1824f1.png)

* スイッチングハブ
こちらもラズパイの電源供給に関わることなので「これらの製品を選ぶにあたって悩んだこと」にて電源ケーブルと一緒に解説します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1275928/ca9d4adb-0200-994b-dd87-c94e518a731f.png)

* LANケーブル
LANケーブルは家に70mほど余っていたので流用しました。100均でも買えると思うので価格は載せないでおきます。
<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1275928/fcea2e1d-62d8-cf50-ba48-a086b9ad0d9c.png" width="40%">


* SATA to USBケーブル
Amazonに中華製のSATA to USBケーブルを何個か買った中ほとんどがすべてUASPに対応してなかったり、1日で断線したりするので、日本の大手パソコンショップでも置いてあるGroovy
<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1275928/378c3a2c-bf27-8df9-a097-e9988e536889.png" width="30%">


## 失敗談・改善点
選択に失敗した製品を並べます。
* ケース
これの何がいけなかったのかと言いますとむき出しのため埃がたまりやすいです。
また、組み立て・解体がとても手間がかかります。(30分くらい)
おすすめできるかと思うと微妙です。

## これらの製品を選ぶにあたって悩んだこと
# 使用する環境
* OS
Ubuntu Server 22.04 LTS
(2022年4月現在)
# 参考書籍・参考リンク
以下の記事や書籍や動画などの資料が非常に参考になりました。
* 参考書籍

    * 自作PCクラスタ超入門:ゼロからはじめる並列計算環境の構築と運用
https://www.amazon.co.jp/gp/product/4627818211/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1
    * Raspberry Piでスーパーコンピュータをつくろう!
https://www.amazon.co.jp/gp/product/4320124375/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1
* 参考リンク
    * ラズパイ4で作るディスプレイ付きKubernetesクラスター
https://qiita.com/reireias/items/0d87de18f43f27a8ed9b
    * RaspberryPiクラスタ製作記 第1回「スパコンを作ろう」
http://www.cenav.org/raspi2/
    * Raspberry Pi 4Bで4台構成の自宅クラスター！ ラズパイ4B向けPoE HATを試す
https://internet.watch.impress.co.jp/docs/column/shimizu/1325054.html
    * i built a Raspberry Pi SUPER COMPUTER!! // ft. Kubernetes (k3s cluster w/ Rancher)
https://www.youtube.com/watch?v=X9fSMGkjtug&ab_channel=NetworkChuck
# 続き
続きは環境構築をしていきます。Ubuntuの設定やサーバー構築・Dockerの環境構築などをしていきたいと思います。
5月下旬までに投稿予定です。