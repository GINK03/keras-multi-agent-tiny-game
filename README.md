# Multi Agent Deep Q Network for Keras

## Kerasでマルチエージェント DQN
マルチエージェントラーニングは、相互に影響を与え合うモデルが強調ないし、敵対して、目的となる報酬を最大化するシチュエーションのディープラーニングです[1][2]

強化学習の特殊系と捉えることができそです　　

Deep Mind社が提案したモデルの一部では非常に面白く、報酬の設定しだいでは各エージェントが協力したり敵対したりします。　　

Kerasで敵対的な簡単なマルチエージェントラーニングを21言っちゃダメゲームで構築しました  

(これはQiitaのはむこさんの記事を参考にさせていただきました、ありがとうございます[3]) 

(調べながらやったこともあり、理論的な間違いを見つけたら、ツイッターで指摘していただけると助かります)  

## 強化学習の理論
強化学習は、人間が特に正しい悪いなどを指定せずとも、なんらかの報酬系から報酬を得ることで報酬を最大化します　　

このとき、ある系列の状態をSとし、その時の行動をrとし、この組み合わせで得られる報酬関数をQとすると、報酬はのようになります。  
<p align="center">
  <img width="200px" src="https://user-images.githubusercontent.com/4949982/28245373-e7ef3be6-6a3f-11e7-8440-7307f7814321.png">
</p>

また報酬の割引率というものがあるらしく、行動が連続する系では、最終的に得られる得点が各選択に対して何かしら値が振られるのですが、通常どの程度影響を及ぼしているのかわかりません  

（わからないので、今回は具体的に割引率を与えたり、求めるということをしていません）  

今回の例では、Q関数を具体的にDeepLearningによる関数としています  

### ϵ-greedy
全く意識していなかったのですが、どうやら行動の選択は初期値依存性がある程度あり、運が悪いと局所解に嵌ったなままなかなか更新してくれなくなります　　

適度にランダムに行動を選択することを入れないとダメっぽいです　　

## ルール（問題設定）
先手、後手に別れて0から最小１、最大の３つ増やした数字を言い合います　　

数字を累積していって、21以上のになるように言った時点で負けです。相手に21以上を踏ませれば勝ちです  

## 報酬の設定
一個一個の行動に報酬を設定するのは、困難なので、一連の行動の系の結果として勝ったか、負けたかを見ていきます　　

以下の式が本当に最小化すべきことであるのです
<p align="center">
  <img width="300px" src="https://user-images.githubusercontent.com/4949982/28245532-fb6944b0-6a43-11e7-9ec5-76e267fe3a41.png">
</p>

これをもう少し雑に変更します（Nはゲーム終了までにかかった手数）　　
<p align="center">
  <img width="420px" src="https://user-images.githubusercontent.com/4949982/28259485-707732a8-6b11-11e7-834a-c0086963f58e.png">
</p>

勝った時の報酬を+1, 負けた時の報酬を-1のReward関数とすると、以上の式を最小化すれば、勝った時はその選択をより強化して学習し、負けた時は選択を誤ったとして、別の可能性を探索する可能性が強くなります　 

## Q関数
Q関数は状態と次にする行動を入力することで、値を得ます　　

今の状態といくつかの
Q関数は状態と次にする行動を入力することで、予想する報酬を計算します  

報酬がもっとも多いと期待できる選択を選ぶことで、（この問題の場合）最短の手数で、勝ちに行くことができます

この出力がもっとも高い行動をゲーム中保存しておき、一連のゲームの行動とします  
<p align="center">
  <img width="450px" src="https://user-images.githubusercontent.com/4949982/28259591-f631b918-6b11-11e7-8e65-53450eb09ac4.png">
</p>

## マルチエージェント
同じような報酬系をもつモデルを２つ以上用意して、対決させました  

先に21以上を言った方が負けというルールで二つのモデルに対決させました  


## コード＆実行
[github](https://github.com/GINK03/keras-multi-agent-tiny-game)にて管理しています　　

マルチエージェント学習
```console
$ python3 21-icchadame-pure.py --reinforce
```

実際に対戦してみる  
```console
$ python3 21-icchadame-pure.py --play
```

## 強さについて
この21言っちゃダメゲームは4の倍数を取りに行けば勝てることがわかっている問題なのですが、途中から4の倍数にはめ込もうとしようとしていることがわかります  

例えば、次のような結果になります  

なお、最適解は、初手で１を打って次に4を取らせることですが、初期値依存性があり、この状態に素早く収束させるのは結構難しいです  

下記の例は、20000回のゲームをさせた例

例1. 
```console
コンピュータは3を選択しました
now position 3
数字（１−３）を入力してください
2
コンピュータは3を選択しました
now position 8
数字（１−３）を入力してください
1
コンピュータは3を選択しました
now position 12
数字（１−３）を入力してください
2
コンピュータは2を選択しました
now position 16
数字（１−３）を入力してください
1
コンピュータは3を選択しました
now position 20
数字（１−３）を入力してください
1
結果 あなたの負け
```

例2. 
```console
コンピュータは3を選択しました
now position 3
数字（１−３）を入力してください
3
コンピュータは2を選択しました
now position 8
数字（１−３）を入力してください
3
コンピュータは1を選択しました
now position 12
数字（１−３）を入力してください
3
コンピュータは1を選択しました
now position 16
数字（１−３）を入力してください
3
コンピュータは1を選択しました
now position 20
数字（１−３）を入力してください
3
結果 あなたの負け
```
## 実際やってみて
Reinforce Learning関しては教師あり学習、教師なし学習についで、最後にやろうと思っていたこともあり、あまり深く手をつけていませんでした  

最初、理論をあまり勉強せず、とりあえず適当にコードを書いてみたのですが、収束はめっちゃ早いけど、すぐオーバーフィットしてしまうモデルになってしまいました。基礎理論を再度確認して、コードに落として行くという気持ちでやると、サクサクできます（反省）  

マルチエージェントにすることで、コードがやばいことになるのかなと思ったのですが、意外とシンプルに構築することができました  

今回は敵対的なゲームでしたが、時には協力し、時には裏切るモデルなど面白そうであります。学習させることも容易なので、様々な応用が利きそうで面白そうでした  


## 参考文献
[1] [Understanding Agent Cooperation](https://deepmind.com/blog/understanding-agent-cooperation/)  
[2] [Multi-agent Reinforcement Learning in Sequential Social Dilemmas](https://storage.googleapis.com/deepmind-media/papers/multi-agent-rl-in-ssd.pdf)  
[3] [深層強化学習：「20言っちゃダメゲーム」の最適解を30分程度で自動的に編み出す（chainerRL）](http://qiita.com/hamko/items/119750780dc430760d78#_reference-4664ea066f5790a8570e)  