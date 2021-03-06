マイクロサービスを結合させて動作確認する---2018-12-22 15:41:59

この記事は[Microservices Advent Calendar](https://qiita.com/advent-calendar/2018/microservices)23日目の記事です。

---

マイクロサービスは関心ごとのスコープを小さくできる。
大きなシステムを小さなマイクロサービスに分割することで、
各チームは小さなマイクロサービスだけに集中することができる。
これこそマイクロサービスのメリットだ。

と同時に、自分の開発したマイクロサービスが、全体の中で正しく振る舞うのかを
テストするのは難しいことである。
筆者の所属するメルカリのように、サービス間通信をgRPCで行っている場合、インタフェースは予めprotoとして定義されている。
そのため、レスポンスとしてありえるケースをすべてモックしてテストコードを書くことはもちろんできる。
しかし、 **実際に他のサービスと結合してテストを行いたい** という場合がある。
関心のスコープを小さくするという意味では、考えすぎなのかもしれないが、
簡単に他のサービスと繋げる方法を持っておくことは悪くはない。

この記事では、どのようにマイクロサービスを簡単に他のマイクロサービスと結合するかを考える。
なお、自分が開発するサービス以外のサービスは、クラウド上の開発環境ですでに稼働していることを前提とする。

## 1. すべてローカルで動かす

![](/images/ms-connect/1.png) 

(画像は[Development Environments for Microservices](https://dzone.com/articles/development-environments-for-microservices)より。以下全て同様。)

minikubeなどを使ってすべてローカルで動かすことができれば、手軽だ。
メリットは構築/削除がしやすいことである。
デメリットはいくつかある。

* ローカルでのクラスタ構築までの手順が大変
* Pub/SubやSpannerなど、クラウドマネージドなサービスのエミュレートは困難である
* プロダクションとローカルで様々な設定を揃える必要がある。例えば、サービス呼び出しの権限など。
* 要するに、 **結合という意味では不十分である**

## 2. すべてクラウド上で動かす

![](/images/ms-connect/2.png)

次に考えられるのは、ローカルで開発したサービスをクラウド上の開発環境にデプロイする方法である。

メリットは、より現実に近いことだ。
インフラがterraformなどでIaCとして構築されていれば、なお安心感がある。
デメリットはいくつかある。

* 確認までに時間がかかる
* デバッグが困難
* ログの取得がめんどくさい

## 3. ローカルとクラウドの組み合わせ① マネージドサービスだけはクラウド

![](/images/ms-connect/3.png)

例えば、ビジネスロジック自体はローカルで動かし、データベースなどはクラウドで動かすという選択肢がある。
このやり方は悪くないが、結局ローカルにクラスタを構築する手間は発生してしまう。
結論としては、次の4のやり方がいいと思っている。

## 4. ローカルとクラウドの組み合わせ② 自分の開発したサービス以外はすべてクラウド上で動かす

![](/images/ms-connect/4.png)

このやり方は、動作確認までの時間、ログ確認やデバッグのやりやすさ、構築の容易性などがメリットである。
すでにクラウドで動いているマイクロサービスに手を入れる必要もない。

ローカルのサービスからの接続をどのように行うかは、VPNやプロキシを立てる方法がひとつある。
ただし、Kubernetesを使っている場合は `kubectl port-forward` や、[telepresenceio/telepresence](https://github.com/telepresenceio/telepresence/)などを使っていると、容易につなぎこみが可能である。

## まとめ

結論としては、4のやり方がおすすめである。
Kubernetesを使っていれば、デメリットらしいデメリットもない。
