NATインスタンスについて学んだ---2017-11-18 18:39:17

NATインスタンス(EC2)は Source/Destチェック を無効にしなければいけない理由について学んだ。

AWSインフラでNATインスタンスを構築する際は、NATインスタンス自身の Source/Destチェック を無効にする必要がある。

### NATインスタンスとは？

プライベートサブネットからインターネットに出るために使う。
プライベートサブネットのルートテーブルで、デフォルトルートをNATインスタンスに向ける。
NATインスタンス自体はパブリックサブネット自体にある。

### Source/Destチェック とは？

インスタンスが自分宛てに来たパケット以外を捨てるための設定である。

### 何故無効にするのか？

NATインスタンスは、その性質上自分宛てのパケットをインターネットに通す必要がある。
NATインスタンスがあるパブリックサブネットのルートテーブルのデフォルトルートはインターネットゲートウェイに向いており、
プライベートサブネットはインターネットへのトラフィックをNATインスタンスに送信することで、外向きの通信を実現する。
しかし、NATインスタンスとはいえただのインスタンスであり、
インスタンスはデフォルトで自分宛てでないパケットは捨ててしまうので、そこはパススルーするような設定が必要になる。

また、基本的にはNATゲートウェイを使うのが良い。
NATインスタンスは設定が面倒で、インスタンス使用料金も、スケールしない(HTTP Proxyを使えばスケールできる)ため、あまり採用するメリットはない。
