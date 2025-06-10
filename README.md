webrtc_ros [![Build Status](https://api.travis-ci.org/RobotWebTools/webrtc_ros.png)](https://travis-ci.org/RobotWebTools/webrtc_ros)
================

#### Streaming of ROS Image Topics using WebRTC
This node provides a WebRTC peer that can be configured to stream a ROS image topic and recieve a stream that is published to a ROS image topic.
The node hosts a webserver that serves a simple test page and offers a websocket server that can be used to create and configure a WebRTC peer.

For full documentation, see [the ROS wiki](http://wiki.ros.org/webrtc_ros).

This project is released as part of the [Robot Web Tools](http://robotwebtools.org/) effort.

### License
webrtc_ros is released with a BSD license. For full terms and conditions, see the [LICENSE](LICENSE) file.

### Authors
See the [AUTHORS](AUTHORS.md) file for a full list of contributors.

## 以下 PiezoSonic 追記
2025/6/10 PiezoSonic 羽部
### 本リポジトリについて
本リポジトリは、[オリジナルのwebrtc_ros](https://github.com/RobotWebTools/webrtc_ros)に対し以下の処理を行ったものである。

1. オリジナルをfork
1. [Jazzy対応PR](https://github.com/RobotWebTools/webrtc_ros/pull/72)を適用
1. webrtcについて[Unified Planへの移行](https://github.com/PiezoSonic/webrtc_ros/pull/2)
1. [Debianパッケージ化スクリプトを追加](https://github.com/PiezoSonic/webrtc_ros/pull/3)

ベースとしているブランチは[develop-ros2](https://github.com/RobotWebTools/webrtc_ros/tree/develop-ros2)で、PiezoSonicとしてのメインブランチは[develop-piezo](https://github.com/PiezoSonic/webrtc_ros/tree/develop-piezo)。

### ビルド方法
[Jazzy対応PR](https://github.com/RobotWebTools/webrtc_ros/pull/72)の[コメント](https://github.com/RobotWebTools/webrtc_ros/pull/72#issuecomment-2640532650)に従うが、記載のソースコード修正は[Jazzy対応PR](https://github.com/RobotWebTools/webrtc_ros/pull/72)[他](https://github.com/PiezoSonic/webrtc_ros/pull/1)で適用されているので、これを前提に改めて以下に示す。ブランチは```develop-piezo```を使用する。  

1. ツールのインストール
   webrtcはビルドツールとして```ninja```、pythonコマンドとして```python```（3系だが```python3```コマンドではない）を使用するため、```ninja-build```と```python-is-python3```をインストールする
    ```bash
    sudo apt install ninja-build python-is-python3
    ```
2. クローンしたディレクトリ（webrtc_ros）で以下コマンドを実行する
   ```bash
   colcon build --cmake-args -DCMAKE_CXX_FLAGS=-fpermissive --packages-select webrtc
   ```
   ※初回実行時はソースコードのダウンロード（23GB程）が行われるため、時間がかかる
3. エラーが発生しビルド失敗で終了したら、ソースコードに対し以下の修正を行う
   - 修正対象 
     - webrtc_ros/webrtc/build/webrtc/src/call/rtp_demuxer.h
     - webrtc_ros/webrtc/build/webrtc/src/rtc_base/third_party/base64/base64.h
   - 修正内容
     - それぞれのファイルの```include```文部分に以下を追加する
       ```cpp
       #include <cstdint>
       ```
4. 再度以下コマンドを実行する（今度は全パッケージビルドする）
   ```bash
   colcon build --cmake-args -DCMAKE_CXX_FLAGS=-fpermissive
   ```
### Debianパッケージの作成
各パッケージ毎にDebianパッケージを作成する。  
パッケージ作成前に[ビルド方法](#ビルド方法)に従って一度ビルドし、ビルドが正常終了する状態にしておくこと。

以降各項において、作業開始時のカレントディレクトリはクローンしたディレクトリ（```webrtc_ros```）とする。

#### 全パッケージ共通
- 以下コマンドを実行する
   ```bash
   rm -rf build/ install/ log/
   colcon build --cmake-args -DCMAKE_CXX_FLAGS=-fpermissive --symlink-install
   ```

#### webrtcパッケージ
1. 以下コマンドを実行する
   ```bash
   cd webrtc
   fakeroot debian/rules binary
   ```
1. クローンしたディレクトリ（```webrtc_ros```）にパッケージが出力される
    - ros-jazzy-webrtc_59.0.4-0noble_amd64.deb
    - ros-jazzy-webrtc-dbgsym_59.0.4-0noble_amd64.ddeb

#### webrtc_ros_msgsパッケージ
1. 以下コマンドを実行する
   ```bash
   cd webrtc_msgs
   fakeroot debian/rules binary
   ```
1. クローンしたディレクトリ（```webrtc_ros```）にパッケージが出力される
    - ros-jazzy-webrtc-ros-msgs_59.0.4-0noble_amd64.deb
    - ros-jazzy-webrtc-ros-msgs-dbgsym_59.0.4-0noble_amd64.ddeb

#### webrtc_rosパッケージ
1. ```webrtc_ros/package.xml```を修正する
    - 修正内容
      - ```<depend version_gte="59.0.0">webrtc</depend>```と```<depend>webrtc_ros_msgs</depend>```をコメントアウトする
1. 以下コマンドを実行する
   ```bash
   cd webrtc_ros
   fakeroot debian/rules binary
   ```
1. クローンしたディレクトリ（```webrtc_ros```）にパッケージが出力される
    - ros-jazzy-webrtc-ros_59.0.4-0noble_amd64.deb
    - ros-jazzy-webrtc-ros-dbgsym_59.0.4-0noble_amd64.ddeb
1. 1.で行った修正をもとに戻す

### 作成したDebianパッケージの環境への適用
[Debianパッケージの作成](#Debianパッケージの作成)で作成したパッケージを適用環境に持ち込み、```apt```コマンドでインストールする。

1. 作成したパッケージ（3つ）を適用環境に持ち込む
   ```ros-jazzy-webrtc_59.0.4-0noble_amd64.deb```、```ros-jazzy-webrtc-ros-msgs_59.0.4-0noble_amd64.deb```、```ros-jazzy-webrtc-ros_59.0.4-0noble_amd64.deb```の3ファイルをインストールするPC等にUSBメモリや```scp```コマンドなどでコピーする

以下の手順は、インストール先環境で実行する。

2. 1.で持ち込んだパッケージを```/tmp```ディレクトリにコピーする
   ```bash
   cp *.deb /tmp
    ```
3. ```apt```コマンドでインストールする
   ```
   sudo apt install /tmp/ros-jazzy-webrtc_59.0.4-0noble_amd64.deb /tmp/ros-jazzy-webrtc-ros-msgs_59.0.4-0noble_amd64.deb /tmp/ros-jazzy-webrtc-ros_59.0.4-0noble_amd64.deb
   ```
