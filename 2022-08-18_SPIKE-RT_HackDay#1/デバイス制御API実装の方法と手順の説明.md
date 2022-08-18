# デバイス制御API実装の方法と手順の説明
2022-08-18  
発表者：
名古屋大学大学院情報学研究科　組込みリアルタイムシステム研究室
朱　義文（@envzhu）

## SIKE-RTのビルドとテスト
### コードの入手
以下により，ソースコードをクローンする．
```bash
git clone git@github.com:spike-rt/spike-rt.git
cd spike-rt
git submodule update --init ./external/
```
以下，特に断りの無い限りトップディレクトリが`spike-rt` のディレクトリであるとする．

### 開発環境の構築
[docs/ja/Env.md](https://github.com/spike-rt/spike-rt/blob/main/docs/ja/Env.md)を参照．

※M1 Macの場合は注意！

Docker for Macの場合  
```bash
docker buildx build tools/ -t spike-rt-builder --platform linux/amd64
```

### テスト
[docs/ja/Test.md](https://github.com/spike-rt/spike-rt/blob/main/docs/ja/Test.md)を参照．


### 作業タイム
環境構築&テストの作業タイム（とりあえず10分間）  
躓いている点・質問などあれば，チャットまたは音声で対応します．

## 今回のハッカソンでやること
- 以下のデバイス制御APIの実装．
  - Pybricksと同じ（または一部の）APIの実装．
  - APIは適宜変更しても構わない．また，冗長と思われる場合は全てを実装する必要はない．

### デバイスリスト
- HUB Device
  - Display : [Issue #1](https://github.com/spike-rt/spike-rt/issues/1)
  - Button : [Issue #2](https://github.com/spike-rt/spike-rt/issues/2)
  - IMU(加速度センサ) : [Issue #2](https://github.com/spike-rt/spike-rt/issues/2)
  - Speaker : [Issue #4](https://github.com/spike-rt/spike-rt/issues/4)
- PUP Device
  - Motor : [Issue #5](https://github.com/spike-rt/spike-rt/issues/5)

### 難易度順
低いものが上．難易度差は大きくない．
- Speaker 
- Motor
- Display
- Button
- IMU


## API開発手順例
あくまでも一例
1. 実装する関数の決定．
2. 公開用ヘッダファイルを実装する．
3. テストコードを実装する．
4. Makefileにソースコードと（必要であれば）インクルードパスを登録．
5. コンパイルが通るように空の関数を実装する．
6. テストが落ちることを確認する．
7. テストが，全て通るまで実装する．

## 実装する上での参考
- [Pybricks Documentation — pybricks v3.2.0a1 documentation](https://docs.pybricks.com/en/latest/index.html)
- [Pybricks Modules and Examples - docs-pybricks-com-en-latest.pdf](https://docs.pybricks.com/_/downloads/en/latest/pdf/)
  - Pybricks 公式のドキュメント．各APIの説明・動作例が載っている．
  - SPIKE-RTが使用しているバージョンのPybricksとは変更されている可能性があることに注意する．

- [test/](https://github.com/spike-rt/spike-rt/blob/main/test/)
  - 各テストコード
- external/Unity/README.md
  - テストコードの実装に
- asp3/target/primehub\_gcc/drivers/cbricks/\*
  - 各ドライバの実装
- external/libpybricks/pybricks/pupdevices/
  - Pybricksの各 PUP デバイスの micropython 向け API の実装
  - これをTOPPERS/ASP3向けに移植するイメージ

## PybricksのMicroPython向けAPIをSPIKE-RTに移植する上での注意点
MicroPythonランタイムに依存のコードを修正する．(以下1~3)

### 1. MicroPython向けの引数などのデータ表現を修正する．
（例で説明する？）

### 2. MicroPython向け例外処理を消去する．
`nlr_push()/nlr_pop()`によるエラー処理をif文とreturn文によるエラー処理に置き換える．  
`setjmp()/longjmp()`には置き換えない．

### 3. MicroPythonランタイム依存の関数呼び出しを置き換える．
delay[ms]待つ処理：`mp_hal_delay_ms(delay)` -> `dly_tsk(delay*1000)`


## コーディングをする上での情報
[docs/ja/CONTRIBUTING.md](https://github.com/spike-rt/spike-rt/blob/main/docs/ja/CONTRIBUTING.md)を参照．

## その他のお願い
- コーディングスタイル
  - 後で，clang-formatにより整形するので，特に気にしなくても良い
- 著作権表示
  - 各ソースファイルは以下のようなフォーマットで著作権表示するのが望ましい．
  - **著作権をメンテナーのERTLに移譲して頂くことが前提．**
    - 貢献者を`CHANGELOG.md`に明記．
    - 移譲できない事情がある場合は応相談.
  - 参考にしたソースコードとその著作権も明記する．

```c
// SPDX-License-Identifier: MIT
/*
 * API for ultrasonic sensors
 *
 * Copyright (c) 2022 Embedded and Real-Time Systems Laboratory,
 *             Graduate School of Information Science, Nagoya Univ., JAPAN
 */
```

- コードの整形
  - こちら，これから導入予定なので，とりあえず無視で良い．
  - `script/clang-format.sh`を実行し，コードを整形する．`clang-format`の前後で実行結果が変わってしまうことが稀にあるので，実行前にコミットし，実行後にテストが全て通ることを確認する．


## デバッグ方法
- printfデバッグとUnityによるユニットテストによるデバッグを想定．
- GDBによるデバッグには未対応．
- USB シリアルは，Unityから利用可能
```c
TEST_PRINTF("color reflected : h : %u  s : %u v : %u\n", hsv.h, hsv.s, hsv.v);

```
- Port Fは専用コネクタを接続すれば，syslogから出力可能．

## Hub/Lightの例
### ヘッダファイル
- [asp3/target/primehub_gcc/drivers/cbricks/hub/light.h](https://github.com/spike-rt/spike-rt/blob/main/asp3/target/primehub_gcc/drivers/cbricks/hub/light.h)
- Doxygenについてのコメントを適宜記述するのが望ましい．（任意）

### テストコード
- [test/hub/test_light.c](https://github.com/spike-rt/spike-rt/blob/main/test/hub/test_light.c)
- テストの呼び出し : [test/test.c#L19](https://github.com/spike-rt/spike-rt/blob/main/test/test.c#L19)
- 対応するMakefile : [test/Makefile.inc#L6](https://github.com/spike-rt/spike-rt/blob/main/test/Makefile.inc#L6)

### API実装
- [asp3/target/primehub_gcc/drivers/cbricks/hub/light.c](https://github.com/spike-rt/spike-rt/blob/main/asp3/target/primehub_gcc/drivers/cbricks/hub/light.c)
- 対応するMakefile : [asp3/target/primehub_gcc/drivers/Makefile#L22](https://github.com/spike-rt/spike-rt/blob/main/asp3/target/primehub_gcc/drivers/Makefile#L22)

## PUP/UltrasonicSensorの例




# Issueの割当ての調整

以上