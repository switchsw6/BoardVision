# BoardVision

BoardVision 是一个面向五子棋棋谱图片的本地识别工具，包含桌面端 Python 脚本和 Android 应用。项目目标是从棋谱截图或照片中识别棋盘、棋子位置和棋子序号，并输出可复制的棋谱文本或 SGF 格式棋谱。

BoardVision is a local recognition tool for Gomoku record images. It includes a desktop Python script and an Android app, and aims to recognize the board, stones, stone numbers, and export copyable move text or SGF.

## 目录 / Contents

| 中文目录 | English Contents |
| --- | --- |
| [中文说明](#中文说明) | [English README](#english-readme) |
| [主要功能](#主要功能) | [Features](#features) |
| [基本使用流程](#基本使用流程) | [Basic Workflow](#basic-workflow) |
| [识别流程](#识别流程) | [Recognition Pipeline](#recognition-pipeline) |
| [模型与样本](#模型与样本) | [Model and Samples](#model-and-samples) |
| [样本贡献](#样本贡献) | [Sample Contribution](#sample-contribution) |
| [Android 开发者模式](#android-开发者模式) | [Android Developer Mode](#android-developer-mode) |
| [项目结构](#项目结构) | [Project Structure](#project-structure) |
| [Android 构建](#android-构建) | [Android Build](#android-build) |
| [发布与签名](#发布与签名) | [Release and Signing](#release-and-signing) |
| [开发流程](#开发流程) | [Development Process](#development-process) |
| [隐私说明](#隐私说明) | [Privacy](#privacy) |
| [已知限制](#已知限制) | [Known Limitations](#known-limitations) |
| [反馈与联系](#反馈与联系) | [Feedback and Contact](#feedback-and-contact) |
| [许可证](#许可证) | [License](#license) |

## 中文说明

项目仍在持续迭代中。识别效果会受图片清晰度、棋盘样式、裁切范围、序号字体、压缩质量和最后一手标记影响。如果你遇到识别错误，欢迎提交样本帮助改进模型。

## 主要功能

桌面端 Python 脚本：

- 选择并识别五子棋棋谱图片。
- 手动裁切棋盘区域。
- 检测棋盘边界、格线和棋子位置。
- 识别棋子上的手数编号。
- 自动排序或手动排序棋子序号。
- 调整单颗棋子的序号，或点击两子交换序号。
- 自动补全剩余序号。
- 输出普通坐标棋谱或 SGF 棋谱。
- 保存标注图和调试结果，便于排查识别问题。

Android 应用：

- 从相册选择棋谱图片。
- 手动框选四角并拉正棋盘。
- 检测黑白棋子位置。
- 使用内置模型识别棋子序号。
- 支持自动排序、手动排序、调整序号、交换两手。
- 支持长按查看原图，松手返回渲染结果。
- 支持复制普通棋谱或 SGF 棋谱。
- 支持自动补全剩余序号。
- 支持开发者模式和悬浮日志。
- 每次识别后自动保存原图和拉正后的图片。
- 支持导出样本压缩包，用于后续训练和排查。

## 基本使用流程

Android 端：

1. 点击“选图”，从相册选择一张五子棋棋谱图片。
2. 点击“手动框图”，拖动四个角点框住棋盘。
3. 点击“拉正并识别”。
4. 检查棋子位置和识别结果。
5. 如需处理序号，可使用“自动排序”“手动排序”“调整序号”或“交换两手”。
6. 点击“复制棋谱”复制结果。
7. 如果识别有误，可在“说明”或“设置”中导出样本并反馈。

桌面端：

1. 运行 Python 脚本。
2. 选择棋谱图片。
3. 必要时手动裁切棋盘。
4. 使用自动识别或手动排序。
5. 检查识别结果。
6. 导出棋谱文本或 SGF。

## 识别流程

BoardVision 的识别大致分为以下步骤：

1. 读取图片。
2. 手动框图或自动定位棋盘区域。
3. 透视拉正棋盘。
4. 识别棋盘网格和边界。
5. 检测黑白棋子位置。
6. 以棋子中心裁切单颗棋子。
7. 使用棋子序号识别模型判断每颗棋子的编号。
8. 汇总棋子位置和编号，生成完整棋谱。
9. 用户可手动修正排序、序号或交换两手。
10. 输出普通文本棋谱或 SGF 棋谱。

棋盘定位、棋子检测和序号识别是相对独立的步骤。棋子位置正确并不代表序号一定正确；序号识别会受到字体、颜色、压缩、模糊、红色最后一手标记等因素影响。

## 模型与样本

项目使用专门训练的棋子序号识别模型，对单颗棋子裁切图进行分类识别。

训练样本主要来自：

- 真实棋谱截图裁切。
- 自动生成的棋子序号样本。
- 不同棋盘材质、字体、颜色、清晰度和缩放比例的合成样本。
- 带最后一手标记的特殊样本。
- 用户反馈的识别失败样本。

Android 应用每次完成识别后，会保存两类图片：

```text
Android/data/<applicationId>/files/recognition_samples/original/
Android/data/<applicationId>/files/recognition_samples/warped/
Android/data/<applicationId>/files/recognition_exports/
```

- `original/`：用户选择的原图。
- `warped/`：手动框图后拉正的棋盘图。
- `recognition_exports/`：导出的样本压缩包。

这些样本可用于后续继续训练模型、排查棋盘定位问题和改进自动框图能力。

## 样本贡献

BoardVision 的识别效果高度依赖真实样本。欢迎大家上传或反馈识别失败、不稳定、风格特殊的棋谱图片，帮助改进模型。

推荐提交内容：

- 原始棋谱图片。
- 应用或脚本识别后的截图。
- 错误说明，例如哪个序号识别错、哪颗棋子漏识别、棋盘边界是否偏移。
- Android 应用导出的样本压缩包。
- 如果方便，也可以提供正确棋谱文本。

提交样本前，请确认图片中不包含你不希望公开的信息。你可以通过 GitHub Issue 或联系作者提交样本。样本越丰富，模型对不同棋盘样式、字体、压缩质量和最后一手标记的适应能力就越好。

## Android 开发者模式

Android 应用支持开发者模式：

1. 快速点击左上角 `BoardVision` 标题 7 次。
2. 第 5 次点击时会提示继续点击。
3. 开启后，设置页会出现“显示日志”选项。

开发者模式可用于查看图片加载、棋盘识别、棋子识别数量、模型判断结果、样本保存路径和其他调试日志。

## 项目结构

```text
BoardVision/
├─ Python_Script/                  # 桌面端 Python 脚本
├─ Android_App/
│  └─ BoardVision/                 # Android 应用工程
│     ├─ app/
│     │  ├─ src/main/java/         # Android Kotlin 源码
│     │  ├─ src/main/assets/       # 模型文件
│     │  └─ src/main/res/          # 布局、样式、资源
│     ├─ build.gradle.kts
│     ├─ settings.gradle.kts
│     └─ README.md                 # Android 子项目说明
├─ LICENSE
└─ README.md                       # 项目总说明
```

部分训练样本、测试图片或中间产物可能不会全部提交到 GitHub。模型训练和样本生成会随着项目继续迭代。

## Android 构建

建议环境：

- Android Studio。
- JDK 17，推荐使用 Android Studio 自带 JBR。
- Android Gradle Plugin 8.x。
- Kotlin Android Plugin。
- `minSdk = 24`。
- `targetSdk = 34`。

使用 Android Studio 打开：

```text
Android_App/BoardVision/
```

然后执行：

```text
Sync Project with Gradle Files
Build APK
```

如果命令行环境配置完整，也可以运行：

```powershell
gradle :app:assembleDebug
```

注意：如果本机默认 Java 版本过高，例如 Java 25，可能导致 Gradle/Kotlin 无法解析版本。建议使用 Android Studio 自带 JBR 或 JDK 17。

## 发布与签名

正式发布前建议：

1. 确认 `applicationId`。
2. 递增 `versionCode`。
3. 更新 `versionName`。
4. 使用同一套签名密钥。
5. 构建 release APK 或 AAB。
6. 在真机完整测试主要识别、排序、复制、导出和开发者日志功能。

签名文件和密码不要提交到 GitHub。同一个 `applicationId` 的后续版本必须使用同一套签名，否则无法覆盖升级。

## 开发流程

本项目大致按以下流程迭代：

1. 在桌面端 Python 脚本中验证棋盘识别、棋子检测和序号识别思路。
2. 收集真实棋谱样本。
3. 自动生成合成训练样本。
4. 训练棋子序号识别模型。
5. 在桌面端验证模型准确率。
6. 将稳定模型接入 Android 应用。
7. 对齐桌面端和 Android 端的棋盘识别逻辑。
8. 增加手动修正功能，降低模型偶发错误的影响。
9. 保存真实识别样本，用于下一轮训练。
10. 继续迭代模型、棋盘识别和 UI。

## 隐私说明

BoardVision 当前设计为本地识别工具：

- 不需要服务器识别。
- 不主动上传用户图片。
- 识别样本保存在应用专属目录中。
- 用户可自行导出样本压缩包，用于训练或排查。

如果后续接入云端服务、统计 SDK 或崩溃收集 SDK，应同步更新隐私说明。

## 已知限制

- 极度模糊、压缩严重或被遮挡的序号可能识别错误。
- 红色最后一手标记可能干扰序号识别。
- 特殊棋盘样式、强反光、透视畸变较大的图片可能需要手动调整。
- 自动排序结果仍建议人工检查。
- Android 不同系统对 `Android/data` 目录访问限制不同，导出样本功能用于降低取样难度。

## 反馈与联系

如果你遇到识别错误、棋盘无法定位、Android 构建失败或有功能建议，欢迎联系作者。

- QQ：3269938079
- 项目仓库：https://github.com/switchsw6/BoardVision

反馈识别问题时，建议提供原始图片、识别截图、错误说明和导出的样本压缩包。

## 许可证

本项目采用自定义非商业署名许可。除商业用途外，你可以使用、复制、修改和分发本项目，但必须保留版权声明、许可证文本，并明确声明原作者版权。

未经版权持有人书面许可，不得将本项目代码、模型文件、训练样本或衍生作品用于商业用途。完整条款见 [LICENSE](LICENSE)。

## English README

BoardVision is still under active development. Recognition accuracy can be affected by image clarity, board style, crop range, number font, compression quality, and last-move markers. If you encounter recognition errors, sample contributions are welcome and very helpful.

## Features

Desktop Python script:

- Select and recognize Gomoku record images.
- Manually crop the board area.
- Detect board boundaries, grid lines, and stone positions.
- Recognize move numbers on stones.
- Sort move numbers automatically or manually.
- Edit a single move number, or swap two moves.
- Fill missing remaining numbers automatically.
- Export plain coordinate text or SGF.
- Save annotated images and debug output for troubleshooting.

Android app:

- Select a record image from the gallery.
- Manually mark the four board corners and rectify the board.
- Detect black and white stone positions.
- Recognize stone numbers with the built-in model.
- Support automatic sorting, manual sorting, number editing, and move swapping.
- Press and hold to view the original image, then release to return to the rendered result.
- Copy plain record text or SGF.
- Fill missing remaining numbers automatically.
- Support developer mode and floating logs.
- Save the original and rectified board image after each recognition.
- Export sample archives for future training and debugging.

## Basic Workflow

Android:

1. Tap "Select Image" and choose a Gomoku record image from the gallery.
2. Tap "Manual Crop" and drag the four corner points around the board.
3. Tap "Rectify and Recognize".
4. Check stone positions and recognition results.
5. Use automatic sorting, manual sorting, number editing, or move swapping if needed.
6. Tap "Copy Record" to copy the result.
7. If recognition is wrong, export samples from the info or settings panel and send feedback.

Desktop:

1. Run the Python script.
2. Select a record image.
3. Manually crop the board if needed.
4. Use automatic recognition or manual sorting.
5. Check the result.
6. Export plain record text or SGF.

## Recognition Pipeline

BoardVision roughly follows these steps:

1. Load the image.
2. Manually mark or automatically locate the board area.
3. Rectify the board with perspective transform.
4. Detect the board grid and boundaries.
5. Detect black and white stone positions.
6. Crop each stone around its grid center.
7. Use the stone number model to recognize the number on each stone.
8. Combine positions and numbers into a full record.
9. Allow manual correction, number editing, or move swapping.
10. Export plain text or SGF.

Board localization, stone detection, and number recognition are separate steps. Correct stone positions do not guarantee correct numbers. Number recognition may be affected by font, color, compression, blur, and red last-move markers.

## Model and Samples

The project uses a dedicated stone number recognition model for cropped single-stone images.

Training samples mainly come from:

- Crops from real record screenshots.
- Automatically generated stone-number samples.
- Synthetic samples with different board textures, fonts, colors, clarity levels, and scales.
- Special samples with last-move markers.
- User-submitted failure cases.

After each recognition, the Android app saves:

```text
Android/data/<applicationId>/files/recognition_samples/original/
Android/data/<applicationId>/files/recognition_samples/warped/
Android/data/<applicationId>/files/recognition_exports/
```

- `original/`: the image selected by the user.
- `warped/`: the rectified board image.
- `recognition_exports/`: exported sample archives.

These samples can be used to continue model training, debug board localization, and improve automatic board framing.

## Sample Contribution

BoardVision depends heavily on real samples. If you find recognition errors, unstable results, or unusual board styles, please consider sending samples to help improve the model.

Recommended materials:

- Original record image.
- Screenshot of the app or script result.
- Error description, such as wrong number, missed stone, or shifted board boundary.
- Sample archive exported from the Android app.
- Correct record text if available.

Please make sure the images do not contain anything you do not want to make public. Samples can be submitted through GitHub Issues or by contacting the author.

## Android Developer Mode

The Android app supports developer mode:

1. Quickly tap the `BoardVision` title in the top-left corner 7 times.
2. A hint appears after the 5th tap.
3. After enabling it, the settings panel shows a log display option.

Developer mode can show image loading information, board recognition details, stone count, model predictions, sample save paths, and other debug logs.

## Project Structure

```text
BoardVision/
├─ Python_Script/                  # Desktop Python script
├─ Android_App/
│  └─ BoardVision/                 # Android app project
│     ├─ app/
│     │  ├─ src/main/java/         # Android Kotlin source code
│     │  ├─ src/main/assets/       # Model files
│     │  └─ src/main/res/          # Layouts, styles, resources
│     ├─ build.gradle.kts
│     ├─ settings.gradle.kts
│     └─ README.md                 # Android subproject README
├─ LICENSE
└─ README.md                       # Project README
```

Some training samples, test images, and intermediate artifacts may not be committed to GitHub. Model training and sample generation will continue to evolve.

## Android Build

Recommended environment:

- Android Studio.
- JDK 17, preferably the JBR bundled with Android Studio.
- Android Gradle Plugin 8.x.
- Kotlin Android Plugin.
- `minSdk = 24`.
- `targetSdk = 34`.

Open this directory in Android Studio:

```text
Android_App/BoardVision/
```

Then run:

```text
Sync Project with Gradle Files
Build APK
```

If your command-line environment is configured, you can also run:

```powershell
gradle :app:assembleDebug
```

If the local default Java version is too new, such as Java 25, Gradle or Kotlin may fail to parse it. Use Android Studio JBR or JDK 17.

## Release and Signing

Before release:

1. Confirm `applicationId`.
2. Increase `versionCode`.
3. Update `versionName`.
4. Use the same signing key for later updates.
5. Build a release APK or AAB.
6. Test recognition, sorting, copying, exporting, and developer logs on a real device.

Do not commit signing files or passwords to GitHub. Future versions with the same `applicationId` must use the same signing key, otherwise they cannot be installed as updates.

## Development Process

The project has roughly evolved through this process:

1. Verify board recognition, stone detection, and number recognition in the desktop Python script.
2. Collect real record samples.
3. Generate synthetic training samples.
4. Train the stone number recognition model.
5. Validate model accuracy on desktop.
6. Integrate the stable model into the Android app.
7. Align Android board recognition logic with the desktop script.
8. Add manual correction tools to handle occasional model errors.
9. Save real recognition samples for the next training round.
10. Continue improving the model, board recognition, and UI.

## Privacy

BoardVision is designed as a local recognition tool:

- It does not require server-side recognition.
- It does not upload user images automatically.
- Recognition samples are saved in the app-specific directory.
- Users can export sample archives for training or debugging.

If cloud services, analytics SDKs, or crash reporting SDKs are added later, this privacy section should be updated accordingly.

## Known Limitations

- Extremely blurred, heavily compressed, or occluded numbers may be recognized incorrectly.
- Red last-move markers may interfere with number recognition.
- Special board styles, strong reflections, or large perspective distortion may require manual adjustment.
- Automatic sorting results should still be checked manually.
- Access to `Android/data` differs across Android versions, so sample export is provided to make sample collection easier.

## Feedback and Contact

If you encounter recognition errors, board localization failures, Android build issues, or have feature suggestions, feel free to contact the author.

- QQ: 3269938079
- Repository: https://github.com/switchsw6/BoardVision

When reporting recognition issues, please provide the original image, recognition screenshot, error description, and exported sample archive if possible.

## License

This project is available under a custom non-commercial attribution license. You may use, copy, modify, and distribute it for non-commercial purposes, but you must keep the copyright notice, include the license text, and give clear attribution to the original author.

Commercial use of the source code, model files, training samples, or derivative works is not allowed without prior written permission from the copyright holder. See [LICENSE](LICENSE) for the full terms.
