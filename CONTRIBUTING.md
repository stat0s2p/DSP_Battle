# 贡献与本机开发指南

本文档说明如何在本机配置 `DSP_Battle` 的开发与测试环境，避免将个人绝对路径写入项目文件。

## 1. 前置条件

- Windows + Visual Studio 2019/2022（含 .NET Framework 4.7.2 开发组件）
- 已安装《Dyson Sphere Program》与 BepInEx
- 建议将仓库克隆到任意可写目录

## 2. 关键路径变量

项目通过 `DSP_GAME_DIR` 定位游戏目录，不再依赖固定盘符。

- `DSP_GAME_DIR` 示例：
  - `D:\SteamLibrary\steamapps\common\Dyson Sphere Program`
  - `E:\Games\Dyson Sphere Program`

可选变量：

- `DeployToGameDir=true`：构建后自动复制 `DLL/PDB` 到 `$(DSP_GAME_DIR)\BepInEx\plugins\`
- 默认 `DeployToGameDir=false`：仅输出到 `bin\Debug` 或 `bin\Release`

## 3. 本机构建（不自动部署）

### Visual Studio

1. 打开 `DSP_Battle.sln`
2. 在项目属性或 `Directory.Build.props`（本地不提交）中设置：
   - `DSP_GAME_DIR=你的游戏目录`
3. 直接 Build（Debug/Release）

### 命令行

```powershell
msbuild .\DSP_Battle.csproj /t:Build /p:Configuration=Debug /p:DSP_GAME_DIR="D:\SteamLibrary\steamapps\common\Dyson Sphere Program"
```

## 4. 本机构建并自动部署到游戏插件目录

```powershell
msbuild .\DSP_Battle.csproj /t:Build /p:Configuration=Debug /p:DSP_GAME_DIR="D:\SteamLibrary\steamapps\common\Dyson Sphere Program" /p:DeployToGameDir=true
```

说明：
- 若 `DeployToGameDir=true` 但未设置 `DSP_GAME_DIR`，构建会报错并提示。

## 5. 本机测试建议

1. 启动游戏并确认 BepInEx 正常加载。
2. 检查 `BepInEx\plugins` 下是否存在最新 `DSP_Battle.dll`（以及对应 `pdb`）。
3. 进入游戏后验证：
   - 模组可加载；
   - 关键功能（事件、UI、联机相关功能）无明显报错；
   - `BepInEx\LogOutput.log` 无新增严重异常。

## 6. 协作约定

- 不要把个人绝对路径写回 `*.csproj`。
- 如需本地个性化路径，请使用：
  - 命令行 `/p:DSP_GAME_DIR=...`
  - 或本地私有配置文件（不提交）。
- 提交前至少执行一次本地 Build，并确认项目仍可在未自动部署模式下输出到 `bin` 目录。
