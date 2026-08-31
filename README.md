# LotRO Companion Release 构建与数据预处理指南

本项目用于一键组装并打包 LotRO Companion 中文分发版（包含独立 Java 运行时、精简后的数据与地图切片、补丁覆盖层）。

---

## 一、 数据结构与本地化设计说明

项目采用 **“实体数据骨架与多语言标签解耦”** 的机制：

1. **主数据索引骨架**（`lotro-data/lore/*.xml` 及 `lotro-items-db/items.xml`）：
   - 存储属性数值、装备槽位、任务要求、经纬度坐标等模型骨架。
   - 其中的 `name`、`description` 仅作为 Fallback 兜底默认值。
2. **多语言标签字典**（`lotro-data/lore/labels/{lang}/*.xml`）：
   - 包含 `en`（官方英文）、`zh`（中文汉化）、`ru`、`de`、`fr` 等。
   - 存放所有通用数据文本的 Key-Value 映射表。
3. **地图专属标签字典**（`lotro-data/lore/maps/labels/{lang}/`）：
   - 包含 `basemaps.xml`（底图/区域名称）和 `markers.xml`（NPC、怪物、采集点标记点标签）。
   - 按语言分目录存储。

---

## 二、 新版本数据导出与预处理流程（更新数据时使用）

当从官方上游（Upstream）同步或使用数据提取器（`lotro-data-extractor`）导出新版本数据后，请按以下流程处理：

### 步骤 1：主数据与英文标签同步
1. 从官方 `upstream/master` 拉取最新的 `lotro-data`、`lotro-items-db`、`lotro-maps-db`。
2. 保持 `lotro-data/lore/*.xml`、`lotro-items-db/items.xml` 以及 `lotro-data/lore/labels/en/` 为官方原生英文内容。
3. **地图非标签数据同步**：
   - 将 `lotro-maps-db` 中的 `categories/`、`indexes/`、`maps/`、`markers/` 及 `links.xml` 原样同步到 `lotro-data/lore/maps/`（这些包含官方英文拓扑、连线提示与标记点坐标骨架）。
4. **地图英文标签同步**：
   - 从 `lotro-maps-db/labels/en/` 检出最新的 `basemaps.xml` 与 `markers.xml`，放入 `lotro-data/lore/maps/labels/en/`。

### 步骤 2：生成/更新中文标签数据
1. **通用数据中文标签**：
   - 维护或将汉化字典更新至 `lotro-data/lore/labels/zh/` 目录下；
   - 确保每个 XML 文件头部的 `locale` 属性为 `locale="zh"`：
     ```xml
     <?xml version="1.0" encoding="UTF-8"?><labels locale="zh">
     ```
2. **地图专属中文标签**：
   - 将汉化后的地图与标记点标签分别存入 `lotro-data/lore/maps/labels/zh/basemaps.xml` 和 `markers.xml`；
   - 确保文件头部同样为 `locale="zh"`。

---

## 三、 Maven 自动化打包阶段说明

在执行 Maven 构建时，`lotro-companion-release/pom.xml` 和 `app.xml` 会自动执行以下后处理优化（无需手动干预）：

1. **GZIP 压缩（Verify 阶段）**：
   - 遍历 `app/data/lore` 下的所有 `.xml` 和 `.bin` 索引文件；
   - 自动生成 `.xml.gz` / `.bin.gz` 压缩文件并清理原始大文件（减小体积 > 75%）。
2. **地图图片切片转码（Verify 阶段）**：
   - 自动将 `app/data/lore/maps/maps/` 下的 PNG 格式底图批量转换为质量为 `0.78` 的 JPEG（`.jpg`）文件，大幅压缩分发包体积。
3. **补丁与运行时裁剪**：
   - 自动将 `lotro-core-overrides` 覆盖层注入至 `app/lib/patches/`；
   - 自动排除 JRE 运行时中无用的 `lib/*.lib` 静态链接库与 `keytool.exe`。

---

## 四、 一键编译与构建命令

在项目根目录下，使用 JDK 21+ 和 Maven 执行：

```powershell
mvn clean install -DskipTests
```

构建完成后，完整可执行分发包位于：
```
lotro-companion-release/target/app/
```
双击 `run.bat` 或 `run-console.bat` 即可直接启动测试。
