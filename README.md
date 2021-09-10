# 4th-tellus-satellite-challenge
[SIGNATEコンペ The 4th Tellus Satellite Challenge：海岸線の抽出](https://signate.jp/competitions/284)  
前処理、学習、推論のコードです。  
チーム「DSH-Beginners」として参加し、最終評価の順位は33位 / 125位でした。  

Pre-processing, train and predict code.  
Participated as a team "DSH-Beginners", the final evaluation ranking was 33rd / 125th.


# アプローチ概要
過去コンペ上位入賞者と同じくディープラーニングの[セマンティックセグメンテーション](https://speakerdeck.com/motokimura/semantic-segmentation-zhen-rifan-ri?slide=14)で取り組みました。

- 学習データに対して海と陸で塗り分けたマスク画像を用意し、海、陸、欠損の３クラスのセマンティックセグメンテーションとして、モデルの実装を行いました。

- データオーギュメンテーションでは、各学習用データを２０倍に水増しし、[Albumentations](https://github.com/albumentations-team/albumentations)というサードパーティのライブラリを使用しました。使用したデータ拡張は以下の通りです。
    -  水平方向に反転 HorizontalFlip(p=0.5)
    -  上下方向に反転 VerticalFlip(p=0.5),
    -  -90°〜90°の範囲で画像を回転させる。Rotate(limit=[-90, 90], interpolation=0, p=0.5),
    - 3x3のグリッドにしてシャッフル。RandomGridShuffle(grid=(3, 3), p=0.5)
    - ランダムに明るさとコントラストを変える　RandomBrightnessContrast(brightness_limit=0.2, contrast_limit=0.2, p=0.5)
    - 弾性変形（歪ませる）　ElasticTransform(alpha=1, sigma=50, alpha_affine=50, interpolation=1, border_mode=4, p=0.5)

- アーキテクチャは[FPN](http://presentations.cocodataset.org/COCO17-Stuff-FAIR.pdf), エンコーダには[EfficientNet-b5](https://hampen2929.hatenablog.com/entry/2019/07/06/024347)を用い、クラス数は3クラス、入力チャネルを１として、[ファインチューニング](https://www.kikagaku.ai/tutorial/basic_of_computer_vision/learn/tensorflow_finetuning)を行いました。
    - learning_rate : 0.001
    - weight decay: 0.0003
    - encoder_learning_rate : 0.0005
    - ミニバッチ数 : 5
    - epoch数: 8

- 損失関数(Loss)には、Dice, IoU, CEを使用し、以下の式で合計した損失関数を使用しました。
    - `IoU + Dice + 0.8*CE`

- 最適化関数(Optimizer)には[RAdam](https://arxiv.org/abs/1908.03265)を使用しました。

- 学習はGoogle ColabのGPUを使用しました。

- 交差検証には8グループ(1グループ=1シーン)の[Leave One Group Out(LOGO)](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.LeaveOneGroupOut.html)を採用しました。

- 推論では、LOGOで学習されたweightのベストパラメータをアンサンブルさせました。

- public boardのスコアが26.7187758、private boardでのスコアが22.7821334となり、最終スコアが上がりました。過学習が抑えられたモデルが構築できていたと考えられます。

## 参考文献
### ディープラーニング手法
セマンティックセグメンテーション https://speakerdeck.com/motokimura/semantic-segmentation-zhen-rifan-ri?slide=14  
### 深層学習フレームワーク(PyTorch)
PyTorch https://pytorch.org/  
Catalyst(PyTorchのラッパーライブラリ) https://catalyst-team.github.io/catalyst/  
### データ前処理
Albumentations https://github.com/albumentations-team/albumentations  
Albumentations サンプル集 https://qiita.com/kurilab/items/b69e1be8d0224ae139ad  
### アーキテクチャ
モデルのアーキテクチャ segmantation_models.pytorch https://github.com/qubvel/segmentation_models.pytorch  
FPN解説 http://presentations.cocodataset.org/COCO17-Stuff-FAIR.pdf  
FPN論文 https://arxiv.org/pdf/1612.03144.pdf  
EfficientNet https://hampen2929.hatenablog.com/entry/2019/07/06/024347  
### 転移学習、ファインチューニング
転移学習 https://jp.mathworks.com/discovery/transfer-learning.html  
ファインチューニング https://www.kikagaku.ai/tutorial/basic_of_computer_vision/learn/tensorflow_finetuning  
### 最適化手法（Optimizer）
RAdam https://qiita.com/omiita/items/d24568a835da6911b01e  
### Label Smoothing
Label Smoothing解説 https://deecode.net/?p=1229  
### 交差検証
Leave One Group Out(LOGO) https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.LeaveOneGroupOut.html  
### Tensorboard
Tensorboardの使い方 https://qiita.com/tetrar124/items/aa27b4d616c859860d02  
### 学習コンテンツ
衛星画像の分類 https://quest.signate.jp/quests/10019  
Tellus e-learning https://tellusxdp.github.io/tellus-trainer/index.html  
画像100本ノック https://github.com/yoyoyo-yo/Gasyori100knock  
ゼロから作るディープラーニング-Pythonで学ぶディープラーニングの理論と実装 https://www.amazon.co.jp/dp/4873117585/  
Udemy講座 「Hands Onで学ぶ PyTorchによる深層学習入門」 https://www.udemy.com/course/hands-on-pytorch/  
「つくりながら学ぶ! PyTorchによる発展ディープラーニング」 https://www.amazon.co.jp/dp/4839970254  
Yann LeCun教授によるディープラーニングの授業 https://atcold.github.io/pytorch-Deep-Learning/  
