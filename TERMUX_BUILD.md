# RikkaNovel 手机编译指南（Termux 方案，不需要电脑）

## 原理
Minis 的沙箱禁止 JIT 内存（mprotect exec 被拦），JVM/Gradle 跑不起来。
但手机上的 **Termux**（独立 app，aarch64 原生环境）没有这个限制，可以完整编译。

## 第 1 步：安装 Termux
- 从 F-Droid 或 Termux GitHub Release 下载 APK 安装（不要用 Google Play 版，已停更）
- 打开 Termux，等它初始化完成

## 第 2 步：获取源码
1. 在 Minis 里点这个链接下载源码包：
   minis://shared/rikkahub-novel-lite.zip
2. 下载后用**质感文件**把它保存到手机 Download 目录（/sdcard/Download/）

## 第 3 步：Termux 里装环境（逐条粘贴执行）
```bash
pkg update -y
pkg install -y openjdk-21 git unzip
```

## 第 4 步：解压源码
```bash
cd ~
cp /sdcard/Download/rikkahub-novel-lite.zip .
unzip -q rikkahub-novel-lite.zip
cd rikkahub
```

## 第 5 步：安装 Android SDK
```bash
# 下载 cmdline-tools
mkdir -p ~/android-sdk/cmdline-tools
curl -L -o /tmp/cmdtools.zip https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip
unzip -q /tmp/cmdtools.zip -d ~/android-sdk/cmdline-tools
mv ~/android-sdk/cmdline-tools/cmdline-tools ~/android-sdk/cmdline-tools/latest

# 安装 SDK 组件（接受许可）
yes | ~/android-sdk/cmdline-tools/latest/bin/sdkmanager --sdk_root=$HOME/android-sdk \
  "platform-tools" "platforms;android-37" "build-tools;36.0.0"

# 告诉 Gradle SDK 在哪
echo "sdk.dir=$HOME/android-sdk" > local.properties
```

## 第 6 步：编译（耗时 30-60 分钟，手机别锁屏）
```bash
chmod +x gradlew
./gradlew :app:assembleDebug --no-daemon
```

## 第 7 步：拿到 APK
```bash
ls -la app/build/outputs/apk/debug/
# 安装：
# 用质感文件打开 app/build/outputs/apk/debug/app-debug.apk 安装即可
```

## 注意事项
- 编译时手机发热正常，建议插电、关后台、别锁屏（或调永不锁屏）
- 内存小的手机如果报 OOM：关掉其他 app 再试
- 报错就把最后 20 行错误发给我，我帮你修

## 备选方案：GitHub Actions（如果你有/愿注册 GitHub 账号）
源码包里已带 `.github/workflows/build-novel.yml`：
1. github.com 注册 → New repository 建空仓库
2. 把解压后的源码 git push 上去（Termux 里：git init → add → commit → remote add → push）
3. 仓库 Actions 标签页会自动编译，跑完在 Artifacts 下载 APK
