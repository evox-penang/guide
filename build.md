# ビルドの手順メモ

基本的には下記の要領でビルドできる・・・はずです（動かなかった場合はご連絡下さい）


## ビルド設定（必要に応じて変更して下さい）

```bash
ANDROID_VERSION=14
DEVICE=penang
```

## local_manifestsをクローンして、Evolution Xのソース取得

```bash
git clone https://github.com/evox-penang/local_manifests ./repo/local_manifests
repo init -u https://github.com/Evolution-X/manifest -b udc
repo sync -c -j$(nproc --all) --force-sync --no-clone-bundle --no-tags
```

## moto-commonリポジトリ群セットアップの方法（Electimon氏作成の全自動スクリプトを使用）と、追加コード取得

```bash
curl https://raw.githubusercontent.com/moto-common/android_device_motorola_targets/master/scripts/manifest_creator.py > ./manifest_creator.py
curl https://raw.githubusercontent.com/moto-common/local_manifests/$ANDROID_VERSION/moto-common.xml > ./.repo/local_manifests/moto-common.xml
python3 manifest_creator.py .repo/manifests/default.xml .repo/manifests/*.xml .repo/local_manifests/moto-common.xml .repo/local_manifests/a-remove.xml
repo sync -j$(nproc --all) --force-sync
```

## ビルドエラー回避

```bash
# device/qcom/sepolicyの内容にパッチ（ビルドエラー回避）
./device/motorola/targets/scripts/replace_camera_sepolicy.sh

# カーネルコンフィグを作っておく（ダミー・ビルドエラー回避）
echo "# DUMMY" > kernel/motorola/msm-5.4/arch/arm64/configs/vendor/$DEVICE_defconfig
```

## いざ、ビルド開始！

```bash
. build/envsetup.sh
lunch evolution_$DEVICE-userdebug
m evolution
```

# ビルド終了後

`out/*/*/*/IMAGES/`上にboot.imgとvendor_boot.imgがあるので、これをfastbootからフラッシュして下さい。

リカバリーに再起動し、データフォーマット後、`out/target/product/$DEVICE/`上に出来上がった`evolution_$DEVICE-ota-*.zip`形式のzipファイルを`adb sideload`でフラッシュして下さい。

これで、おそらく起動するはずです。
