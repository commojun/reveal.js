## 7年間運用したソーシャルゲームを<br>Amazon EC2構成からAmazon ECS構成へと<br>乗り換えた話

---

## 自己紹介

<img src="./img/marunouchi_ol_trans.png" width="10%"><br>

- 大澤 純 (@commojun)
- 2016~ サーバサイドエンジニア@KAYAC

---

アホみたいなことをつぶやいたり

<img src="./img/perlnigeruna.png" class="r-stretch">

---

バカゲーを作ってリリースしちゃったりしています

<img src="./img/kusonazo.png" class="r-stretch">

<div class="inyou">
https://twitter.com/kayac_inc/status/1434804351336255491
</div>

---

## 今日のお話

<br>

- ぼくらの甲子園!ポケットという長期運用ゲームをコンテナベースのシステムへ移行した
- 長期運用特有・もしくはこのゲーム特有の事情に合わせた工夫点だったと思う箇所
- 専門的な内容でも理解を得ながら仕事を進めることの重要性

---

## 目次

- ぼくらの甲子園!ポケットについて
- EC2時代のサーバ構成
- なぜECSに乗り換えるか
- ホコリかぶったPerlとモジュールのバージョンアップ
- 時代に乗ったデプロイ方法にする
- ゲームの心臓なのにSPOFになっていたバッチサーバの冗長化作戦
- 非エンジニアに仕事をわかってもらう

---

## 目次

- <span class="mokuji">ぼくらの甲子園!ポケットについて</span>
- EC2時代のサーバ構成
- なぜECSに乗り換えるか
- ホコリかぶったPerlとモジュールのバージョンアップ
- 時代に乗ったデプロイ方法にする
- ゲームの心臓なのにSPOFになっていたバッチサーバの冗長化作戦
- 非エンジニアに仕事をわかってもらう

---

ぼくらの甲子園!ポケットは2014年から運用を続けているソーシャルゲームタイトルです。

おかげさまで2021年10月で7周年を迎えることができました。

<img src="./img/kuwatasite.png" class="r-stretch">

<div class="inyou">
https://koshien-pocket.kayac.com/
</div>

---

### ぼくらの甲子園!ポケットの特徴

- ユーザは監督ではなく、選手である
- ユーザが9人集まらないと試合すらできない
- チームメイトの一人ひとりがユーザ
- ユーザは選手になって甲子園を目指すため、青春を追体験できる

<img src="./img/sousa.jpg" class="r-stretch">

---

### ぼくらの甲子園!ポケットの特徴

- 2週間に1回の甲子園大会に出場するためにリーグ戦を繰り返す
- 甲子園大会はトーナメント戦で1日かけて行う
- 優勝すると、優勝チームの要望に応えた新聞を運営が作成し、ゲーム内に掲載する

<img src="./img/ingame.png" class="r-stretch">

---

### ぼくらの甲子園!ポケットの特徴

<img src="./img/tokucho.jpg" class="r-stretch">

- リーグ戦の試合は毎日定刻に開始
- 参加するには決まった時間にログインしなければならない
- 作戦会議中にスキルを発動したり、仲間にエールを送ったりする
- 作戦会議が終了すると完全オートで試合が進行する
- 複雑なアクションは無く、戦況に応じてどのスキルを使うかが重要
- 打席・試合の結果は全てサーバのバッチ実行によって決定する

---

## 目次

- ぼくらの甲子園!ポケットについて
- <span class="mokuji">EC2時代のサーバ構成</span>
- なぜECSに乗り換えるか
- ホコリかぶったPerlとモジュールのバージョンアップ
- 時代に乗ったデプロイ方法にする
- ゲームの心臓なのにSPOFになっていたバッチサーバの冗長化作戦
- 非エンジニアに仕事をわかってもらう


---

ぼくらの甲子園!ポケットは2014年9月12日リリース

当然開発はそれよりも前に開始していた。その頃の時流に乗ったサーバ構成

---

<img src="./img/ec2kousei.jpg" class="r-stretch">

- 役割に応じたEC2インスタンスがあり、必要に応じてオートスケールをしたりする
- Aurora MySQLや、Elasticache（Redis）などマネジメントされた物を利用
- batchサーバ上でcrondが常駐して、定刻の試合開始を行っている

---

## 目次

- ぼくらの甲子園!ポケットについて
- EC2時代のサーバ構成
- <span class="mokuji">なぜECSに乗り換えるか</span>
- ホコリかぶったPerlとモジュールのバージョンアップ
- 時代に乗ったデプロイ方法にする
- ゲームの心臓なのにSPOFになっていたバッチサーバの冗長化作戦
- 非エンジニアに仕事をわかってもらう

---

### 1, Amazon Linuxのサポート終了

<img src="./img/amazonlinux.jpg" class="r-stretch">

EC2上で利用しているOSのサポートが2020年末に終了するため<br>何かしら手を打たなければならなかった

後継OS Amazon Linux2 への乗り換えという選択肢もあるが…

<div class="inyou">
https://aws.amazon.com/jp/blogs/news/update-on-amazon-linux-ami-end-of-life/
</div>

---

### 2, コンテナを利用したサーバ構築が当たり前になりつつある

<br>

他のゲームタイトルやサービスで、Amazon ECSを使った<br>サーバ構成について社内に知見が貯まっていた

---

### 3, このゲームの運用を10年続けたい！

<br>

長期運用タイトルにする意思がチームの総意としてあった

リリース時から時代が変わっているので、将来性のある構成に乗り換えよう！

---

## 目次

- ぼくらの甲子園!ポケットについて
- EC2時代のサーバ構成
- なぜECSに乗り換えるか
- <span class="mokuji">ホコリかぶったPerlとモジュールのバージョンアップ</span>
- 時代に乗ったデプロイ方法にする
- ゲームの心臓なのにSPOFになっていたバッチサーバの冗長化作戦
- 非エンジニアに仕事をわかってもらう

---

<img src="./img/kanshin.jpg" class="r-stretch">

運用が続くとゲーム体験に関係のある部分に関心が向きがち

---

<img src="./img/kanshin2.jpg" class="r-stretch">

言語のバージョンとモジュールのバージョンも長らくメンテされていなかった

---

### Perl 5.16 👉 Perl 5.30

エイヤで言語バージョンを上げてみる

---

@INC問題

---

Perl 5.26での変更点

<blockquote class="inyou">
perl バイナリは @INC にパスのデフォルト集合を含んでいます。歴史的に、汚染モード (perl -T) が有効でない限り、最終的なエントリとして カレントディレクトリ (".") も含んでいました。 これは便利ですが、セキュリティ上の問題がありました: 例えば、 カレントディレクトリが(/tmp のように)信頼できない場合、 スクリプトが追加のモジュールを読み込もうとすると、そのディレクトリの下から コードを読み込んで実行する可能性があります。

v5.26 から、"." は汚染モードの場合だけではなく、 **常にデフォルトで 除去されるようになりました。** これはモジュールのインストールとスクリプトの実行に大きな影響を与えます。
</blockquote>

特にテストコードが影響を受けた

```perl　[4]
use strict;
use warnings;
use utf8;
use t::Util;
use Test::More;
...
```

<div class="inyou">
https://perldoc.jp/docs/perl/5.26.0/perl5260delta.pod
</div>

---

<blockquote class="inyou">
@inc からドットが除去されたことによるテストの問題を修正するとき、 @inc にドットを再挿入するのは慎重に行うべきです、 これも実行時コードの実際の問題を抑制するかもしれないからです。 可能な限り、明示的な絶対/相対パスを使うという前述した手法を適用するか、 必要なファイルをサブディレクトリに再配置して、代わりに そのサブディレクトリを @inc に追加することを勧めます。
</blockquote>

@incのドットに頼ったモジュールはそう多くなかったので、引っ越して対応

##### Before
```bash
/repository_root
|--t
   |--Util.pm
   |--testa.t
   |--testb.t
```

##### After
```bash
/repository_root
|--t
   |--lib
   |  |--t
   |     |--Util.pm
   |--testa.t
   |--testb.t
```

```
perl -Ilib -It/lib
```



<div class="inyou">
https://perldoc.jp/docs/perl/5.26.0/perl5260delta.pod
</div>

---

コンパイルエラーしまくる

---

Perl 5.24での変更点

<blockquote class="inyou">
(autoderef 機能は取り除かれました)

実験的な autoderef 機能 (push, pop, shift, unshift, splice, keys, values, each をスカラ引数で呼び出せるようにする) は失敗と 判断されました。 これは削除されました; この機能を use しようとすると (または以前は 引き起こされていた experimental::autoderef 警告を無効にしようとすると) 例外が発生するようになりました。
</blockquote>

こういうのがダメ

```
my $arrayref = [1,2,3];
while (my $num = shift $arrayref) { # NG!
    ...
}
```

<div class="inyou">
https://perldoc.jp/docs/perl/5.24.0/perl5240delta.pod
</div>

---

@ をつけて回る作業
```perl
my $item_dic = {
    one   => "hoge",
    two   => "fuga",
    three => "piyo",
};
for my $item (keys $item_dic) { # NG!
    ...
}
```
↓
```perl [6]
my $item_dic = {
    one   => "hoge",
    two   => "fuga",
    three => "piyo",
};
for my $item (keys @$item_dic) { # OK
    ...
}
```

---

長いときはpostderef機能に積極的に頼った

<blockquote class="inyou">
接尾辞デリファレンスは実験的ではなくなりました

postderef 機能と postderef_qq 機能を使っても警告が発生しなくなりました。 以前使われていた、experimental::postderef 警告カテゴリを無効にしている 既存のコードはそのまま動作します。 postderef 機能は何の効果もありません; **全ての Perl コードは、スコープ内でどの機能が宣言されているかに関わらず、接尾辞デリファレンスを使えます。** 5.24 機能バンドルは postderef_qq 機能を含むようになりました。
</blockquote>

```perl
my $arrayref_by_name = {
    one => [2,3,4],
    two => [5,6,7],
};
while (my $num = shift $arrayref_by_name->{one}){ # NG!
    ...
}
```
↓

```perl [5]
my $arrayref_by_name = {
    one => [2,3,4],
    two => [5,6,7],
};
while (my $num = shift $arrayref_by_name->{one}->@*){ # postderefに頼る
    ...
}
```

<div class="inyou">
https://perldoc.jp/docs/perl/5.24.0/perl5240delta.pod
</div>

---

テストが運で落ちたり通ったりする

---

Perl 5.18 ハッシュのランダム化のあおりを受ける

<blockquote class="inyou">
Perl のハッシュ関数が使う種はランダムになりました。 これは、keys(), values(), each() のような関数が返すキー/値の 順序は実行毎に異なるということです。
</blockquote>


```perl
my %tests = (
    testa => sub { return "aaa" },
    testb => sub { return "bbb" },
);
my @expected = ["aaa", "bbb"];

for my $test (values %tests) {
    is, $test(), shift @expected; # testa から実行されるとは限らない！
}
```

keys や valuesを使って取り出した配列を使ってfor文を回すような箇所で、<br>配列の順番に依存したテストが落ちていた


<div class="inyou">
https://perldoc.jp/docs/perl/5.18.0/perl5180delta.pod#Hash32randomization
</div>

---

### モジュールのバージョンを上げてみる

---

### 蓋を開けてみると見えた問題

- そもそも cpanfile.snapshot でのバージョン管理がなされていなかった
- 後方互換の無いモジュールバージョンアップがうっかり混入してあわてて固定する運用
- モンキーパッチを当てたせいでバージョンを上げられない
- 日本語を含んだJSONのリクエストがどうしても上手くさばけない

---

2つのOSSにPRを出させていただきました

<img src="./img/oss.png" class="r-stretch">

- ネストされたJSONリクエストの中に日本語があると正しくエンコードされない問題
- ネストされたJSONリクエストが正しくデコードされない問題

<div class="inyou">
https://github.com/kazeburo/HTTP-Entity-Parser/pull/13<br>
https://github.com/moznion/Plack-Request-WithEncoding/pull/3
</div>

---

## 目次

- ぼくらの甲子園!ポケットについて
- EC2時代のサーバ構成
- なぜECSに乗り換えるか
- ホコリかぶったPerlとモジュールのバージョンアップ
- <span class="mokuji">時代に乗ったデプロイ方法にする</span>
- ゲームの心臓なのにSPOFになっていたバッチサーバの冗長化作戦
- 非エンジニアに仕事をわかってもらう

---

EC2構成からECS構成へ

コンテナベースのシステムへの転換

---

最も恩恵があるのがデプロイまわり

最も変化があるのがデプロイまわり

---

### EC2構成のデプロイ方法

strecherというOSSによるpull型のデプロイツール

<img src="./img/ec2deploy.jpg" class="r-stretch">

- デプロイサーバがリポジトリから配布物を取得する
- デプロイサーバが配布物を全て一つのアーカイブにまとめてAmazon S3に保存する
- 各ホストがAmazon S3から配布物を取得する

<div class="inyou">
https://techblog.kayac.com/10_stretcher.html
</div>

---

### ECS構成のデプロイ方法

<img src="./img/ecs-deploy.png" class="r-stretch">

- とにかくコンテナイメージをビルドしてリポジトリに収める
- デプロイ = コンテナインスタンスの入れ替え

---

### EC2構成でのデプロイ方法の利点

<img src="./img/ec2deploy2.jpg" class="r-stretch">

Github上でデプロイ用のブランチが準備できれば、その先のデプロイが迅速

そのスピード感に頼ってデプロイの頻度が多い運用となっていた

---

### ビルドが長い

<img src="./img/ecsdeploy2.jpg" class="r-stretch">

ブランチの準備ができてから、ビルドする時間がかかる

---

### circleciにビルドさせる

- ブランチにコミットさえすればビルドされる
- ECRへのPUSHまでのパイプライン

---

### 2階建て作戦

baseイメージとappイメージに分ける

- 夜中に依存パッケージ、モジュール、ツール類をbaseイメージでインストール
- リリース用ブランチへのコミットがある度、アプリケーションコードをコンテナにコピーする

##### Dockerfile.base
```Dockerfile
FROM perl:5.30.0-buster

RUN apt-get install 必要なソフトウェア
...
```

```bash
$ docker build -t app:base .
```

##### Dockerfile
```Dockerfile
FROM app:base

COPY ./ /home/user/repo/
...
```

```bash
$ docker build -t app .
```

運用上必要な箇所のみにコンテナイメージビルドの時間をかける

---

## 目次

- ぼくらの甲子園!ポケットについて
- EC2時代のサーバ構成
- なぜECSに乗り換えるか
- ホコリかぶったPerlとモジュールのバージョンアップ
- 時代に乗ったデプロイ方法にする
- <span class="mokuji">ゲームの心臓なのにSPOFになっていたバッチサーバの冗長化作戦</span>
- 非エンジニアに仕事をわかってもらう

---

### とにかくバッチサーバが命

<img src="./img/shiaiinochi.jpg" class="r-stretch">

試合進行というゲームの最重要要素が1台のバッチサーバ(crontab)にかかっていた

---

### batchサーバが突然死したときのバックアップ策が存在しなかった（！）

<img src="./img/batch1dai.jpg" class="r-stretch">

ECS移行を機に冗長化できないか？

---

### 他プロジェクトでの冗長化事例

CloudWatch Event + SQS + sqsjkr作戦

---

CloudWatch Event(今はEventBridge) + SQS + sqsjkr作戦

<img src="./img/sqsjkr.jpg" class="r-stretch">

- CloudWatch Event: 定刻のイベント発火をマネジメントサービス化
- SQS: 発火したジョブをキューイング
- sqsjkr: SQSからジョブを取得し実行する(排他制御機能があり、冗長化可能)

<div class="inyou">
https://techblog.kayac.com/2017/04/10/090000<br>
https://github.com/kayac/sqsjkr
</div>

---

### ぼくらの甲子園!ポケットでは…

##### crontab.txt
```txt
10 00 * * * perl script/batch_aaa.pl
12 00 * * * perl script/batch_bbb.pl
16 00 * * * perl script/batch_ccc.pl
...
```

- cron書式のテキストファイルで管理している
- テキストを解析して日時や内容の整合性をチェックするテストが作り込まれている
- 100以上のエントリがあるのでCloudWatch Eventの上限に引っかかる
- 運用上、頻繁に書き換える必要がある

**このフォーマットは維持したい…**

---

### sqsjfr + SQS + sqsjkr作戦

---

sqsjfr + SQS + sqsjkr作戦

<img src="./img/sqsjfr.png" class="r-stretch">

- sqsjfr: crondと同様にスケジューラの役割を果たし、実行すべきジョブをSQSに送り込む
- SQS FIFO キュー: 同一メッセージを削除しながらジョブをキューイングする
- sqsjfr: SQSからジョブを取得し実行する

**SQS FIFOキューの重複削除機能によって、sqsjfrの冗長化ができる**

**sqsjfrがcron書式を解釈できるためcron書式がそのまま使える！**

<div class="inyou">
https://github.com/kayac/sqsjfr
</div>

---

## 目次

- ぼくらの甲子園!ポケットについて
- EC2時代のサーバ構成
- なぜECSに乗り換えるか
- ホコリかぶったPerlとモジュールのバージョンアップ
- 時代に乗ったデプロイ方法にする
- ゲームの心臓なのにSPOFになっていたバッチサーバの冗長化作戦
- <span class="mokuji">非エンジニアに仕事をわかってもらう</span>

---

ECS構成への移行作業の意義を非エンジニアの人にわかってもらうのは難しい

---

「ECS作業はユーザにとってどのようなメリットになりますか？」

---

「ええと…Amazon Linuxのサポートが切れるからやらないとダメなんですよ…」

---

これでわかってもらえる職場は果たして存在するだろうか？（反語）

---

ユーザに直接的なメリットははっきり言って無い

でも長く運用するなら必要なこと

### 説明してわかってもらう必要がある！

---

エンジニアとしての背景知識が必要な専門用語を使っていたら理解してもらえない

---

受け手と伝え手が共に持っている**共通言語**を使った**例え**をうまく使う

---

### Amazon Linuxのサポート切れ

**Amazon Linux -> Windows** に言い換えるととたんに実感がわく

<img src="./img/windows.png" class="r-stretch">

---

### なぜECS構成に移行するのか？

- ECS（コンテナ技術）とはなにか？
- なぜわざわざAmazon Linux2ではなくECSを選ぶのか？

---

<img src="./img/background.jpg" class="r-stretch">

それぞれのバックグラウンドを持つが

---

<img src="./img/gamer.jpg" class="r-stretch">

みんなゲーマーという点は共通だった

---

### ゲーム機の時代変遷で例えて説明してみる

<img src="./img/famicon.jpg" class="r-stretch">

**ゲームカセット ≒ コンテナ**

コンテナの便利さと、いかに当たり前になったかを感じてもらえた

<div class="inyou">
https://ja.wikipedia.org/wiki/ホーム・ポン <br>
https://ja.wikipedia.org/wiki/ファミリーコンピュータ
</div>

---

### コンテナ技術が当たり前だと訴える

<img src="./img/support.png" class="r-stretch">

---

<img src="./img/atarimae.png" class="r-stretch">

---

## ECS移行を理解してもらうことの意義

準備作業はほぼサーバサイドエンジニアだけでやる仕事

それを運用中のサービスに適用するとなると、やはり運用メンバー全員の協力が必要

移行メンテナンスも一筋縄ではいかず、不具合もいくつか起こしてしまった

それでも運用メンバー全員がその難易度と重要性を理解していたので

足並みを揃えて問題に対処し、移行作業を完了できた

---

## エンジニアの説明責任？

諦めず工夫して説明すれば、高度に背景知識が必要なことも理解してもらえる

<br>

専門的すぎるからと非エンジニアの人に仕事の意義を理解してもらうことを怠ると

最終的に新しい技術への新陳代謝ができない組織になり、成長の可能性を閉ざしてしまう

<br>

一線級のエンジニアが現役のまま経営も兼ねている組織でもない限りは

このような説明責任も担えるエンジニアが重宝されるのではないだろうか？

---

## さいごに

---

## 泥臭い作業

この発表で紹介したことは、いわゆる **作戦がきれいにハマった** 箇所にすぎない

<br>

- 環境変数の使われかたを正す
- configの場当たり的な書き方を正す
- 修正が必要そうな箇所の洗い出し（勘と経験と気合で）

<br>

結局、ホコリかぶっていた負債を全部見て回って全部直すパワープレイが必要

<br>

### **飽きずに長く運用やって全体把握できる人間じゃないと<br>たぶん無理**

---

ありがとうございました

<img src="./img/marunouchi_ol_trans.png" width="10%"><br>
