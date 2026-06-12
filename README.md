# BoardVision Android

BoardVision Android 是一个面向五子棋棋谱图片的本地识别应用。应用可以从相册选择棋谱图片，手动框选棋盘并拉正，识别棋盘上的黑白棋子和棋子序号，最后输出可复制的棋谱文本或 SGF 格式棋谱。

当前 Android 版本复用了桌面脚本训练出的棋子序号识别模型，并在手机端用 Kotlin 本地推理。图片读取、棋盘拉正、棋子定位、序号识别和排序均在本机完成。

## 主要功能

- 选择棋谱图片并手动框选棋盘。
- 拉正棋盘并识别黑白棋子位置。
- 使用内置模型识别棋子序号并自动排序。
- 支持手动排序、调整序号、交换两手。
- 支持复制普通棋谱或 SGF 棋谱。
- 支持长按查看原图。
- 支持保存并导出识别样本。
- 支持开发者模式和悬浮日志。

## 项目结构

```text
BoardVision/
├─ app/
│  ├─ src/main/java/com/boardvision/   Android 主代码
│  ├─ src/main/assets/
│  │  └─ stone_digit_mlp.bin           棋子序号识别模型，应用启动后由 StoneDigitModel 读取
│  ├─ src/main/res/                    布局、图片、样式资源
│  ├─ build.gradle.kts                 App 构建配置
│  └─ proguard-rules.pro               R8/ProGuard 规则
├─ build.gradle.kts                    根项目 Gradle 配置
├─ settings.gradle.kts                 Gradle 模块配置
├─ gradle.properties                   Gradle 属性
└─ local.properties                    本机配置，禁止提交
```

`stone_digit_mlp.bin` 是当前 Android 应用使用的序号识别模型文件。如果你训练了自己的模型，需要导出为兼容格式并替换这个文件。

## 环境要求

- Android Studio。
- JDK 17，推荐使用 Android Studio 自带 JBR。
- Android Gradle Plugin 8.x。
- Kotlin Android Plugin。
- Android SDK：
  - `compileSdk = 34`
  - `targetSdk = 34`
  - `minSdk = 24`

建议直接使用 Android Studio 构建。

## 运行调试

在 Android Studio 中：

1. 打开 `BoardVision` 项目目录。
2. 等待 Gradle Sync 完成。
3. 选择运行配置 `app`。
4. 连接手机或启动模拟器。
5. 点击 Run。

如果使用命令行，并且本机安装了 Gradle：

```powershell
gradle :app:assembleDebug
```

调试包通常输出到：

```text
app/build/outputs/apk/debug/
```

## 识别样本保存

每次点击“拉正并识别”完成识别后，应用会自动保存原图和一张拉正后的棋盘图片。

默认保存到应用专属目录：

```text
Android/data/<applicationId>/files/recognition_samples/original/
Android/data/<applicationId>/files/recognition_samples/warped/
Android/data/<applicationId>/files/recognition_exports/
```

- `original/`：用户选择的原图。
- `warped/`：手动框图后拉正的棋盘图。
- `recognition_exports/`：应用导出的样本压缩包。

应用内右上角 `i` 信息页会显示当前设备上的实际保存路径，也可以点击“导出样本”将 `original` 和 `warped` 两个目录打包为 zip。

## 开发者模式

快速点击左上角 `BoardVision` 标题 7 次可开启开发者模式。第 5 次会提示继续点击。

开发者模式可显示悬浮日志，用于查看：

- 图片加载信息。
- 棋盘拉正和识别信息。
- 棋子检测数量。
- 模型对每颗棋子的序号判断结果。
- 样本保存和导出路径。

设置页中可以调整日志显示、背景透明度、文字颜色和字体大小。

## 棋子序号识别模型

Android 端的序号识别模型位于：

```text
app/src/main/assets/stone_digit_mlp.bin
```

模型由 `StoneDigitModel` 读取，由 `StoneNumberRecognizer` 调用。它的输入不是整张棋盘，而是按棋盘格点裁切出的单颗棋子图。模型会对棋子上的数字进行识别，随后应用结合以下信息生成整局顺序：

- 单颗棋子的数字概率。
- 黑白棋对应的奇偶手数。
- 红色最后一手标记。
- 整局棋子数量。
- 全局序号分配结果。

也就是说，当前排序不是简单地逐颗读取数字，而是“单颗模型识别 + 整局约束分配”。

### 模型开发过程简介

当前模型的大致开发流程是：

1. 先用桌面脚本验证棋盘识别、棋子定位和裁切方式。
2. 从真实棋谱中裁切单颗棋子，建立带标签的真实样本。
3. 生成大量合成棋子样本，覆盖不同数字、黑白棋、字体、清晰度和轻度模糊。
4. 加入 hard samples，重点修复容易混淆的数字，例如 3/9、4/8、41/43 等。
5. 使用轻量 MLP 训练三位数字分类模型。
6. 在真实样本和整局棋谱上评估。
7. 导出为 Android 可直接读取的 `stone_digit_mlp.bin`。
8. 在 Android 端通过 `StoneNumberRecognizer` 做整局序号分配。

当前模型追求的是本地、轻量、易嵌入。它不依赖服务器，也不依赖大型深度学习运行库。

### 如何训练自己的模型

如果你有一批固定风格的棋谱图片，可以训练自己的模型来适配该风格。

推荐流程：

1. **收集样本**
  在 Android 应用中完成识别后，点击“导出样本”。导出的压缩包中会包含：
   其中 `warped/` 是拉正后的棋盘图，更适合继续裁切训练。
2. **准备训练数据**
  使用训练脚本或桌面脚本从整张棋盘中裁切单颗棋子，并生成 manifest。manifest 至少需要记录：
  - 裁切图路径。
  - 正确序号标签。
  - 样本状态，例如 `accepted`。
   标签必须尽量准确。错误标签会直接降低模型准确率。
3. **生成合成样本**
  真实样本通常不够多，建议混合合成样本。合成样本应覆盖：
  - 黑棋和白棋。
  - 1 位、2 位、3 位数字。
  - 常见字体和字号。
  - 轻度模糊、压缩、锐化。
  - 最后一手红色标记。
  - 与真实裁切一致的棋子大小和留边。
4. **训练 MLP 模型**
  如果仓库中包含 `stone-number-recognizer/` 训练项目，可以参考以下流程：
   参数可以根据样本规模调整。真实样本较少时，不要只追求训练集准确率，应额外保留一部分真实图片做验证集。
5. **评估模型**
  至少做两类评估：
   第一类评估看单颗棋子的直接识别准确率；第二类评估看整局分配后的准确率。实际应用中，整局分配准确率更接近用户体验。
6. **导出 Android 模型**
  训练得到 `.npz` 后，需要导出成 Android 读取的二进制格式：
   导出脚本会写入 `BVMLP1` 格式的模型文件。Android 端当前按这个格式读取。
7. **替换并测试**
  替换：
   然后重新构建 Android 应用，在真机上测试：
  - 单颗数字是否更准。
  - 整局排序是否连续。
  - 红色最后一手是否仍能正确处理。
  - 3/9、4/8、41/43 等易混数字是否改善。

### 模型替换注意事项

- 如果只替换 `.bin`，模型结构必须与当前 Android 端兼容。
- 如果修改了输入尺寸、隐藏层结构、输出类别或二进制格式，需要同步修改 `StoneDigitModel`。
- 如果修改了裁切策略，需要同步修改 `StoneNumberRecognizer` 中的裁切和预处理逻辑。
- 不建议只用合成样本训练。真实样本越多，模型越稳定。
- 建议保留一批没有参与训练的真实棋谱作为最终测试集。

## 权限和隐私

当前 Manifest 中声明：

```xml
<uses-permission android:name="android.permission.CAMERA" />
```

应用主要处理用户选择的棋谱图片。当前设计目标是本地识别，不依赖服务器上传图片。

识别样本默认保存在应用专属目录中。用户可以手动导出样本压缩包，用于反馈问题或继续训练模型。

## 样本贡献

模型准确率需要持续依赖真实样本改进。如果你遇到识别错误，欢迎把样本发给开发者。

推荐提供：

- 原始棋谱图片。
- 应用识别后的截图。
- 错误说明，例如哪个数字识别错、哪颗棋子漏识别。
- 应用导出的样本压缩包。
- 如果方便，也可以附上正确棋谱文本。

样本越丰富，模型越容易适配不同棋盘、字体、压缩质量和最后一手标记。提交样本前，请确认图片中不包含你不希望公开的信息。
