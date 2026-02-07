# 贡献与本机开发指南

本文档说明如何在本机配置 `DSP_Battle` 的开发与测试环境，避免将个人绝对路径写入项目文件。

## 1. 前置条件

- Windows + Visual Studio 2019/2022（含 .NET Framework 4.7.2 开发组件）
- 已安装《Dyson Sphere Program》与 BepInEx
- 建议将仓库克隆到任意可写目录

## 2. 关键路径变量

项目采用严格分离：非 BepInEx 资源走游戏目录，BepInEx 相关引用走插件目录（不混用）。

- `DSP_GAME_DIR` 示例：
  - `D:\SteamLibrary\steamapps\common\Dyson Sphere Program`
  - `E:\Games\Dyson Sphere Program`

可选变量：

- `DSP_BEPINEX_DIR`：BepInEx 根目录（可选；默认从 `$(DSP_PLUGIN_DIR)\..` 推导）
- `DSP_PLUGIN_DIR`：插件目录（必填）
- `DeployToGameDir=true`：构建后自动复制 `DLL/PDB` 到 `$(GamePluginsDir)`
- 默认 `DeployToGameDir=false`：仅输出到 `bin\Debug` 或 `bin\Release`

## 3. 本机构建（不自动部署）

### Visual Studio

1. 打开 `DSP_Battle.sln`
2. 推荐在仓库根目录新建 `Directory.Build.props`（本地使用，不提交）并填写如下内容：

   ```xml
   <Project>
     <PropertyGroup>
       <DSP_GAME_DIR>C:\\Program Files (x86)\\Steam\\steamapps\\common\\Dyson Sphere Program</DSP_GAME_DIR>
       <DSP_BEPINEX_DIR>C:\\Users\\<you>\\AppData\\Roaming\\Thunderstore Mod Manager\\DataFolder\\DysonSphereProgram\\profiles\\dsp\\BepInEx</DSP_BEPINEX_DIR>
       <DSP_PLUGIN_DIR>C:\\Users\\<you>\\AppData\\Roaming\\Thunderstore Mod Manager\\DataFolder\\DysonSphereProgram\\profiles\\dsp\\BepInEx\\plugins</DSP_PLUGIN_DIR>
       <DeployToGameDir>false</DeployToGameDir>
     </PropertyGroup>
   </Project>
   ```

   > 注意：`Directory.Build.props` 必须是合法 XML。
   > 不能写成 `DSP_GAME_DIR="..."` 这种纯文本形式，否则会报“根级别上的数据无效”。
   > `DSP_GAME_DIR` 必须指向**游戏根目录**（含 `DSPGAME.exe` 与 `DSPGAME_Data`）。
   > `DSP_PLUGIN_DIR` 为必填；`DSP_BEPINEX_DIR` 可不填（将由插件目录自动推导）。

3. 直接 Build（Debug/Release）

### 命令行

```powershell
msbuild .\DSP_Battle.csproj /t:Build /p:Configuration=Debug /p:DSP_GAME_DIR="D:\SteamLibrary\steamapps\common\Dyson Sphere Program" /p:DSP_BEPINEX_DIR="C:\Users\sascs\AppData\Roaming\Thunderstore Mod Manager\DataFolder\DysonSphereProgram\profiles\dsp\BepInEx" /p:DSP_PLUGIN_DIR="C:\Users\sascs\AppData\Roaming\Thunderstore Mod Manager\DataFolder\DysonSphereProgram\profiles\dsp\BepInEx\plugins"
```

## 4. 本机构建并自动部署到游戏插件目录

```powershell
msbuild .\DSP_Battle.csproj /t:Build /p:Configuration=Debug /p:DSP_GAME_DIR="D:\SteamLibrary\steamapps\common\Dyson Sphere Program" /p:DSP_BEPINEX_DIR="C:\Users\sascs\AppData\Roaming\Thunderstore Mod Manager\DataFolder\DysonSphereProgram\profiles\dsp\BepInEx" /p:DSP_PLUGIN_DIR="C:\Users\sascs\AppData\Roaming\Thunderstore Mod Manager\DataFolder\DysonSphereProgram\profiles\dsp\BepInEx\plugins" /p:DeployToGameDir=true
```

说明：
- 若 `DeployToGameDir=true` 但未设置 `DSP_GAME_DIR` 或 `DSP_PLUGIN_DIR`，构建会报错并提示。
- Thunderstore 目录结构下会自动优先识别如下 DLL 位置：
  - `CommonAPI-CommonAPI\CommonAPI.dll`
  - `CommonAPI-DSPModSave\DSPModSave.dll`
  - `xiaoye97-LDBTool\LDBTool.dll`
  - `jinxOAO-MoreMegaStructure\MoreMegaStructure.dll`
  - `nebula-NebulaMultiplayerModApi\NebulaAPI.dll`

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


## 7. 常见错误

- **错误：** `未能加载导入的项目文件 ... Directory.Build.props。根级别上的数据无效。第 1 行，位置 1。`
  - **原因：** `Directory.Build.props` 不是 XML 文件（例如只写了一行 `DSP_GAME_DIR="..."`）。
  - **修复：** 按上文 XML 模板重建该文件，或直接复制 `Directory.Build.props.example` 为 `Directory.Build.props` 后再改路径。

- **现象：** 已设置 `DSP_GAME_DIR` 但引用仍无法解析。
  - **排查 1：** 确认 `DSP_GAME_DIR` 指向游戏根目录（用于 `DSPGAME_Data`）。
  - **排查 2：** 确认 `DSP_PLUGIN_DIR` 指向 `BepInEx\plugins`（用于 BepInEx 相关引用）。
  - **排查 3：** 如 `core` 不在 `plugins` 上级目录，额外设置 `DSP_BEPINEX_DIR`。
  - **排查 4：** 若用系统环境变量，修改后需重启 Visual Studio 使其生效。

- **现象：** 只有 `CommonAPI` / `NebulaAPI` / `DSPModSave` / `LDBTool` / `MoreMegaStructure` 等引用报未解析。
  - **原因：** 这些 DLL 来自 BepInEx 插件目录，需要本机已安装对应前置模组。
  - **修复：** 在游戏 `BepInEx\plugins` 中安装对应前置模组（或将其 DLL 放入约定位置）后重新加载项目。

## 8. 提交前同步主分支（避免冲突）

在发起 PR 前，建议先同步主分支并解决冲突：

```bash
git fetch origin
git checkout work
git merge origin/master
# 或使用 rebase: git rebase origin/master
```

如果 `DSP_Battle.csproj` 冲突，优先保留：
- `DSP_GAME_DIR` 相关属性与路径参数化；
- `ValidateGameDirectory` / `DeployToGame` / `PrintResolvedPaths` 目标；
- 其余与业务代码无关的格式性改动尽量最小化。



- 调试路径解析：`msbuild .\DSP_Battle.csproj /t:PrintResolvedPaths`
