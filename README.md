# SlashGame

Unreal Engine 5.0 C++ 项目（第三人称角色 + 开放世界地图）。

本仓库只包含**源码与自有蓝图/地图**，编译产物和第三方素材包不纳入版本管理。换到新电脑后，请按本文档准备环境，否则会出现「模块需要重新编译」「资产缺失（红色）」等问题。

---

## 一、环境要求

| 项目 | 要求 | 说明 |
| --- | --- | --- |
| 引擎 | **Unreal Engine 5.0** | `SlashGame.uproject` 里 `"EngineAssociation": "5.0"`，版本不一致会导致工程打不开 |
| 系统 | Windows 10 / 11 64 位 | |
| IDE | Visual Studio 2019（16.11+）或 2022 | 工作负载勾选「使用 C++ 的桌面开发」，组件勾选 MSVC、Windows 10/11 SDK（10.0.18362+）、.NET Framework 4.8 SDK |
| 磁盘 | 建议 100 GB 以上可用空间 | 引擎安装约 40–60 GB，工程加素材约 7 GB，首次编译中间产物另需数 GB |

> 引擎通过 Epic Games Launcher 安装；安装时建议勾选 **Editor Debug Symbols**，方便调试 C++。

---

## 二、获取代码

```bash
git clone https://github.com/fastspiders/SlashGame.git
cd SlashGame
```

clone 下来的内容约几十 MB，包含：`Config/`、`Source/`（全部 C++）、`Content/BluePrints/`、`Content/Map/`、`Content/__ExternalActors__/`。

**以下内容没有被上传，需要你自己补齐**（见第三节）：

- `Binaries/`、`Intermediate/`、`Saved/`——编译产物，本地编译会重新生成，**不用管**
- `*.sln`——右键 `.uproject` 生成，不用管
- 第三方素材包——**必须补**，否则角色和道具是空壳

---

## 三、必须补齐的素材包（重点）

工程里的蓝图直接引用了第三方素材，这些包因体积和许可原因没有上传。缺了它们，打开蓝图会看到 Mesh / Animation 显示为红色缺失，运行起来角色不可见或报错。

| 依赖它的资产 | 引用的资源 | 需要的素材包 | 怎么补 |
| --- | --- | --- | --- |
| `Content/BluePrints/Characters/BP_SlashCharacter`（默认角色） | `/Game/AncientContent/Characters/Echo/Meshes/Echo`<br>`/Game/AncientContent/Characters/Echo/Animations/Idle` | **AncientContent** | ⚠️ 仓库里没有（1.54 GB，其中单文件 105 MB 超过 GitHub 100 MB 限制）。需从原机器拷贝 `Content/AncientContent/` 整个目录（移动硬盘最快），或在新机器上重新导入同名包，且**目录结构必须保持 `/Game/AncientContent/Characters/Echo/...`** |
| `Content/BluePrints/Pawns/BP_Bird` | `/Game/AnimalVarietyPack/Crow/Meshes/SK_Crow`<br>`/Game/AnimalVarietyPack/Crow/Animations/ANIM_Crow_Fly` | **AnimalVarietyPack** | Epic 商城免费包，Launcher → 添加到工程 |
| `Content/BluePrints/Items/BP_Item` | `/Game/StarterContent/Shapes/Shape_QuadPyramid` | **StarterContent** | Epic 自带免费内容包，新建/添加内容时勾选 Starter Content |
| 编辑器启动地图 | `EditorStartupMap=/Game/StarterContent/Maps/Minimal_Default` | **StarterContent**（同上） | 没有它时首次打开会落在空白地图，手动打开 `Content/Map/SlashOpenWorld` 即可 |
| `Content/Map/WorldDungeon.umap` | 可能用到地牢素材 | MedievalDungeon（可选） | 不用这张图可以不装 |

**目录结构必须与引用路径一致**：这些包要放在工程 `Content/` 下对应的子目录里（如 `Content/AncientContent/`、`Content/AnimalVarietyPack/`、`Content/StarterContent/`），否则引用会断链。

---

## 四、插件与模块依赖

`.uproject` 中启用的插件（新环境安装引擎时这些默认都有，但请确认没被禁用）：

- ModelingToolsEditorMode（仅 Editor）
- AudioModulation
- ControlRig
- Niagara

`Source/SlashGame/SlashGame.Build.cs` 中的模块依赖：

```
Core, CoreUObject, Engine, InputCore, HairStrandsCore
```

其中 **HairStrandsCore** 依赖引擎的 Groom（Hair Strands）插件，若引擎是精简安装、缺少该插件，编译会直接失败——此时在编辑器 Plugins 里启用 Groom 后重新编译。

---

## 五、生成工程文件并编译

1. 右键 `SlashGame.uproject` → **Switch Unreal Engine Version…** → 选 `5.0`（版本不对时先做这步）
2. 右键 `SlashGame.uproject` → **Generate Visual Studio project files**，生成 `SlashGame.sln`
3. 双击打开 `SlashGame.sln`，配置选 **Development Editor / Win64**，生成解决方案
4. 编译完成后双击 `SlashGame.uproject` 打开编辑器

也可以直接在编辑器里点 Compile / 使用 Live Coding，不必开 VS。

> 首次编译需要几分钟到十几分钟（要重建 `Intermediate/`，着色器编译另计），属正常现象。

---

## 六、运行

- 打开地图：`Content/Map/SlashOpenWorld`（这是 `GameDefaultMap`）
- GameMode：`/Game/BluePrints/GameMode/BP_BirdGameMode`，默认 Pawn 为 `BP_SlashCharacter`
- 操作：`ASlashCharacter` 绑定了 MoveForward / MoveRight / Turn / LookUp（`Source/SlashGame/Private/Characters/SlashCharacter.cpp`），走的是传统 Input 轴映射，见 `Config/DefaultInput.ini`
- 点击编辑器工具栏 **Play (PIE)** 即可运行

---

## 七、常见问题

**Q：打开时提示 "The following modules are missing or built with a different engine version: SlashGame"**
A：点 Yes 重新编译；若失败，回到第五节用 VS 手动 Build。这是正常现象——`Binaries/` 没有上传。

**Q：双击 .uproject 提示版本不匹配 / 找不到引擎**
A：右键 → Switch Unreal Engine Version → 选 5.0。若没装 5.0，先装引擎。

**Q：角色/鸟/道具显示不出来，蓝图里资源是红的**
A：第三节的素材包没补齐，或放错了目录。

**Q：编译时报找不到 HairStrandsCore**
A：编辑器 Edit → Plugins 里启用 Groom，重启后重新编译。

**Q：VS 打开后没有 UE 相关配置 / 报工具集不匹配**
A：UE 5.0 官方支持 VS2019；用 VS2022 时需在编辑器 Edit → Editor Preferences → Source Code 里把 Source Code Editor 设为 Visual Studio 2022 并重启。

---

## 八、版本管理说明

`.gitignore` 有意排除了以下内容，不要手动 `git add -f`：

```
Binaries/  Intermediate/  Saved/  DerivedDataCache/  Build/  .vs/
*.sln  *.pdb  *.lib  *.dll  *.exe
Content/MedievalDungeon/  Content/StarterContent/
Content/AnimalVarietyPack/  Content/Characters/
```

`Content/AncientContent/` 目前未被 ignore 但也未提交（1.54 GB，含 105 MB 单文件，超出 GitHub 单文件 100 MB 限制；Git LFS 免费额度也不够）。如需让 `git status` 保持干净，可自行将其加入 `.gitignore`。

规则很简单：**只提交 `Source/` 和自有的 `Content/BluePrints/`、`Content/Map/`，其余靠 Epic Launcher 重新获取或本地编译生成。**
