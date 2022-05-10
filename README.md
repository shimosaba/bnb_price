# ProbSpace 民泊コンペ 9th Place Solution

- [ProbSpace 民泊コンペ 9th Place Solution](#probspace-民泊コンペ-9th-place-solution)
  - [前処理](#前処理)
    - [外れ値除去](#外れ値除去)
    - [翻訳](#翻訳)
  - [特徴量エンジニアリング](#特徴量エンジニアリング)
    - [日付](#日付)
    - [集約](#集約)
    - [name](#name)
    - [位置情報](#位置情報)
    - [ラベルエンコーディング](#ラベルエンコーディング)
    - [ターゲットエンコーディング](#ターゲットエンコーディング)
    - [その他](#その他)
  - [モデル](#モデル)
  - [後処理](#後処理)
  - [上手くいかなかった手法](#上手くいかなかった手法)

ProbSpaceの[民泊コンペ](https://comp.probspace.com/competitions/bnb_price)でPrivate 9位になったのでここに簡単な解法を残しておきます。

`name`のNLP関連の特徴量エンジニアリングは[yuuuuki](https://comp.probspace.com/competitions/bnb_price/discussions/yuuuuki-Post8a46673032da75e94b76)さんを参考にしました  

## 前処理

### 外れ値除去

特に必要ないと思ったが、学習データのID`4360`のデータの形が特殊かなぁと思ったので削除

### 翻訳

`name`が体感英語:9, 日本語:0.7, 中国語:0.3くらいな気がしたので、一部日本語を英語に変換  

- 東京23区
- 男女、家族、カップル、友達
- 空港名、駅名、電車、徒歩、分、アクセス

## 特徴量エンジニアリング

### 日付

- year, month, day

### 集約

- station_nameごとのavailability_365の平均

### name

- 単語数、文字数
- fasttextの結果を次元圧縮、SVC(50)
- Bert, sentenceTransformer, pipeline, universal-sentence-encoder-multilingualでそれぞれ特徴抽出したものをPCAで20次元ずつに次元圧縮
- Universal Sentence Encoderの結果を次元圧縮、SVD(30)
- 出現率が高そうな単語の個別フラグ
'wi-fi', 'free', 'min ', 'airport', 'sale', '★', 'Uhome', 'roppongi', 'modern', 'suite', 'sale'
- 男女
- room_num
- people
- area
- %off
- floor

### 位置情報

- 最寄り駅の名称
- 最寄り駅までの距離
- 100m, 500m以内にある駅の数
- latitude, longitude, room_typeのGaussianMixture(n_components=30, covariance_type='full')
- 重心からの距離

### ラベルエンコーディング

- カテゴリ変数全般

### ターゲットエンコーディング

- gmm
- room_type + neighbourhood
- room_type + minimum_nights

### その他

- neighbourhoodごとにnumber_of_reviewsを標準化
- number_of_reviews / reviews_per_month

## モデル

- LightGBM2モデルのアンサンブル
- ※githubにある1層目のモデル2つでアンサンブルしました。コード自体はスタッキングになっているが、スコアは良くなかった。。。
- モデル1:StratifiedKFold(X, neighbourhood)
- モデル2:StratifiedGroupKFold(X, X['station_name'], host_id)

## 後処理

- nameに`Uhome`がある施設は、予測値と学習データの`Uhome`施設の平均価格で平均

## 上手くいかなかった手法

- スタッキング(あまり時間かけてない。。。)
- ゴールデンウィークの時期フラグ
- availability, number_of_reviewsが0フラグ
