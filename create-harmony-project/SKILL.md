---
name: create-harmony-project
description: 在非 DevEco Studio 的 IDE（如 VS Code、Cursor 等）中创建兼容 DevEco Studio 的 ArkTS HarmonyOS Next 工程。自动检测用户本地 DevEco Studio 安装的 SDK 版本，生成完整可编译的工程结构和配置文件。
---

## 触发条件
当用户需要在非 DevEco Studio 环境创建 HarmonyOS 工程时触发。

## 执行步骤

### 步骤 1: 获取用户输入
询问用户以下信息（如未提供）：
- 工程名称（默认：MyApplication）
- 包名（默认：com.example.myapplication）
- 目标目录（默认：当前目录）

### 步骤 2: 检测环境
```bash
# 检测 DevEco Studio 版本
cat /Applications/DevEco-Studio.app/Contents/resources/product-info.json | grep "version"

# 检测 SDK 版本
cat ~/Library/Huawei/Sdk/system-image/*/phone_all_arm/sdk-pkg.json
```

从输出中提取：
- SDK 版本（如 6.1.0）
- API 版本（如 23）

### 步骤 3: 创建目录结构
在目标目录下创建以下结构：
```
{project_name}/
├── AppScope/
│   ├── app.json5
│   └── resources/base/
│       ├── element/string.json
│       └── media/
├── entry/
│   ├── build-profile.json5
│   ├── hvigorfile.ts
│   ├── oh-package.json5
│   └── src/
│       ├── main/
│       │   ├── ets/
│       │   │   ├── entryability/EntryAbility.ets
│       │   │   ├── entrybackupability/EntryBackupAbility.ets
│       │   │   └── pages/Index.ets
│       │   ├── module.json5
│       │   └── resources/base/
│       │       ├── element/
│       │       ├── media/
│       │       └── profile/
│       ├── mock/mock-config.json5
│       └── ohosTest/module.json5
├── hvigor/hvigor-config.json5
├── build-profile.json5
├── hvigorfile.ts
└── oh-package.json5
```

### 步骤 4: 生成配置文件

使用检测到的版本信息，替换模板中的占位符：
- `{SDK_VERSION}` → 如 "6.1.0"
- `{API_VERSION}` → 如 "23"
- `{BUNDLE_NAME}` → 用户指定的包名
- `{APP_NAME}` → 用户指定的工程名称

### 步骤 5: 输出完成信息
告知用户：
- 工程创建位置
- 如何在 DevEco Studio 中打开
- 后续可能需要的配置（如签名）

## 配置文件模板

### build-profile.json5（根目录）
```json5
{
  "app": {
    "signingConfigs": [],
    "products": [{
      "name": "default",
      "signingConfig": "default",
      "targetSdkVersion": "{SDK_VERSION}({API_VERSION})",
      "compatibleSdkVersion": "{SDK_VERSION}({API_VERSION})",
      "runtimeOS": "HarmonyOS"
    }],
    "buildModeSet": [{ "name": "debug" }, { "name": "release" }]
  },
  "modules": [{
    "name": "entry",
    "srcPath": "./entry",
    "targets": [{ "name": "default", "applyToProducts": ["default"] }]
  }]
}
```

### hvigorfile.ts（根目录）
```typescript
import { appTasks } from '@ohos/hvigor-ohos-plugin';
export default { system: appTasks, plugins: [] }
```

### hvigor/hvigor-config.json5
```json5
{
  "modelVersion": "{SDK_VERSION}",
  "dependencies": {},
  "execution": {},
  "logging": {},
  "debugging": {},
  "nodeOptions": {}
}
```

### oh-package.json5（根目录）
```json5
{
  "modelVersion": "{SDK_VERSION}",
  "description": "Please describe the basic information.",
  "dependencies": {},
  "devDependencies": {
    "@ohos/hypium": "1.0.25",
    "@ohos/hamock": "1.0.0"
  }
}
```

### AppScope/app.json5
```json5
{
  "app": {
    "bundleName": "{BUNDLE_NAME}",
    "vendor": "example",
    "versionCode": 1000000,
    "versionName": "1.0.0",
    "icon": "$media:layered_image",
    "label": "$string:app_name"
  }
}
```

### AppScope/resources/base/element/string.json
```json
{
  "string": [{ "name": "app_name", "value": "{APP_NAME}" }]
}
```

### entry/build-profile.json5
```json5
{
  "apiType": "stageMode",
  "buildOption": {},
  "buildOptionSet": [{
    "name": "release",
    "arkOptions": { "obfuscation": { "ruleOptions": { "enable": false } } }
  }],
  "targets": [{ "name": "default" }, { "name": "ohosTest" }]
}
```

### entry/hvigorfile.ts
```typescript
import { hapTasks } from '@ohos/hvigor-ohos-plugin';
export default { system: hapTasks, plugins: [] }
```

### entry/oh-package.json5
```json5
{
  "name": "entry",
  "version": "1.0.0",
  "description": "Please describe the basic information.",
  "main": "",
  "author": "",
  "license": "",
  "dependencies": {}
}
```

### entry/src/main/module.json5
```json5
{
  "module": {
    "name": "entry",
    "type": "entry",
    "description": "$string:module_desc",
    "mainElement": "EntryAbility",
    "deviceTypes": ["phone"],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages",
    "abilities": [{
      "name": "EntryAbility",
      "srcEntry": "./ets/entryability/EntryAbility.ets",
      "description": "$string:EntryAbility_desc",
      "icon": "$media:layered_image",
      "label": "$string:EntryAbility_label",
      "startWindowIcon": "$media:startIcon",
      "startWindowBackground": "$color:start_window_background",
      "exported": true,
      "skills": [{
        "entities": ["entity.system.home"],
        "actions": ["ohos.want.action.home"]
      }]
    }],
    "extensionAbilities": [{
      "name": "EntryBackupAbility",
      "srcEntry": "./ets/entrybackupability/EntryBackupAbility.ets",
      "type": "backup",
      "exported": false,
      "metadata": [{ "name": "ohos.extension.backup", "resource": "$profile:backup_config" }]
    }]
  }
}
```

### entry/src/main/ets/entryability/EntryAbility.ets
```typescript
import { AbilityConstant, UIAbility, Want } from '@kit.AbilityKit';
import { hilog } from '@kit.PerformanceAnalysisKit';
import { window } from '@kit.ArkUI';

export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    hilog.info(0x0000, 'testTag', 'Ability onCreate');
  }

  onDestroy(): void {
    hilog.info(0x0000, 'testTag', 'Ability onDestroy');
  }

  onWindowStageCreate(windowStage: window.WindowStage): void {
    hilog.info(0x0000, 'testTag', 'Ability onWindowStageCreate');
    windowStage.loadContent('pages/Index', (err) => {
      if (err.code) {
        hilog.error(0x0000, 'testTag', 'Failed to load content: %{public}s', JSON.stringify(err));
        return;
      }
      hilog.info(0x0000, 'testTag', 'Succeeded in loading content');
    });
  }

  onWindowStageDestroy(): void {}
  onForeground(): void {}
  onBackground(): void {}
}
```

### entry/src/main/ets/entrybackupability/EntryBackupAbility.ets
```typescript
import { hilog } from '@kit.PerformanceAnalysisKit';
import { BackupExtensionAbility, BundleVersion } from '@kit.CoreFileKit';

export default class EntryBackupAbility extends BackupExtensionAbility {
  async onBackup() {
    hilog.info(0x0000, 'testTag', 'onBackup ok');
    await Promise.resolve();
  }

  async onRestore(bundleVersion: BundleVersion) {
    hilog.info(0x0000, 'testTag', 'onRestore ok');
    await Promise.resolve();
  }
}
```

### entry/src/main/ets/pages/Index.ets
```typescript
@Entry
@Component
struct Index {
  @State message: string = 'Hello World';

  build() {
    Column() {
      Text(this.message)
        .fontSize(50)
        .fontWeight(FontWeight.Bold)
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

### entry/src/main/resources/base/element/string.json
```json
{
  "string": [
    { "name": "module_desc", "value": "module description" },
    { "name": "EntryAbility_desc", "value": "description" },
    { "name": "EntryAbility_label", "value": "label" }
  ]
}
```

### entry/src/main/resources/base/element/color.json
```json
{
  "color": [{ "name": "start_window_background", "value": "#FFFFFF" }]
}
```

### entry/src/main/resources/base/profile/main_pages.json
```json
{ "src": ["pages/Index"] }
```

### entry/src/main/resources/base/profile/backup_config.json
```json
{ "allowToBackupRestore": true }
```

### entry/src/mock/mock-config.json5
```json5
{}
```

### entry/src/ohosTest/module.json5
```json5
{
  "module": {
    "name": "entry_test",
    "type": "feature",
    "deviceTypes": ["phone", "tablet", "2in1"],
    "deliveryWithInstall": true,
    "installationFree": false
  }
}
```

## 关键规则

| 规则 | 说明 |
|------|------|
| SDK版本格式 | `"{SDK_VERSION}({API_VERSION})"` 如 "6.1.0(23)" |
| hvigorfile | 根目录用 `appTasks`，entry用 `hapTasks` |
| app_name | 只在 AppScope 定义 |
| icon | 用 `$media:layered_image` |
| actions | 用 `"ohos.want.action.home"` |
| ArkTS类型 | 必须显式声明，禁止 any/unknown |
| API导入 | 用 `@kit.*` 新API，不用 `@ohos.*` |

## 常见错误处理

| 错误 | 解决 |
|------|------|
| Invalid exports, no system plugins | 根目录改用 appTasks |
| Can not find build config file | 创建 entry/build-profile.json5 |
| Resource 'app_name' conflict | 只在 AppScope 定义 app_name |
| $media:xxx not defined | 确保媒体资源存在 |
| arkts-no-any-unknown | 添加显式类型声明 |
| Cannot find module '@ohos.*' | 改用 @kit.* API |
