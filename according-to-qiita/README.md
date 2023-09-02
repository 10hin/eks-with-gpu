# これは何

GPUを利用するEKSクラスタの使い方の勉強。

まずは日立製作所の大崎さんがQiita.comで公開している[記事][qiita-osaki]を参考に基本的に同じ内容を動かしてみる

[qiita-osaki]: https://qiita.com/Hiroyuki_OSAKI/items/fa465fe48cfa191ded5a

## 記事との差分

- eksctlのサブコマンドでNGを追加するのではなくマニフェストを書いてクラスタ作成・NodeGroup作成まで行った
- `gpupod.yaml`の`nvidia/cuda:latest`イメージは公開されなくなっていたのでマニフェストにある通り`nvidia/cuda:12.2.0-base-ubuntu22.04`を使用した
- nvidia-device-pluginの更新
    - nvidia-device-pluginが[EKSの導入手順][use-gpu-ami]で導入される最新バージョン([0.14.1][k8s-device-plugin-release])より古かったので更新することにした
    - daemonsetはダウンロードしてみたらそのまま上書きできそうだったのでapplyしたが、`nvidia-smi`コマンドで変化は確認できなかった
    - nodeGroupの入れ替えでNodeを入れ替えてみる
    - 新しいNodeで実行しても(少なくとも`nvidia-smi`の出力は)変化がなかった

[use-gpu-ami]: https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/eks-optimized-ami.html#gpu-ami
[k8s-device-plugin-release]: https://github.com/NVIDIA/k8s-device-plugin/releases/tag/v0.14.1

クラスタ構築で同時にできたNodeで`gpupod.yaml`を実行したときのログ:

```text
Sat Sep  2 03:07:47 2023
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.182.03   Driver Version: 470.182.03   CUDA Version: 12.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla K80           On   | 00000000:00:1E.0 Off |                    0 |
| N/A   31C    P8    27W / 149W |      0MiB / 11441MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

ドライバ更新・ノード入れ替え後のNodeで`gpupod.yaml`を実行したときのログ:

```text
Sat Sep  2 03:42:20 2023
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.182.03   Driver Version: 470.182.03   CUDA Version: 12.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla K80           On   | 00000000:00:1E.0 Off |                    0 |
| N/A   33C    P8    28W / 149W |      0MiB / 11441MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

