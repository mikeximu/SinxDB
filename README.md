# 📦 **SinxDB – Lightweight Hybrid Key-Value Storage Engine**

SinxDB 是一个由 Go 实现的轻量级、高性能键值存储引擎。
整体设计注重 **速度、可控性、可读性**，强调 **可嵌入性** 与 **可预测行为**。

SinxDB 并不是模仿任何现有数据库，而是基于以下核心理念重新构建：

* 数据写入应该是 **顺序、可恢复、易维护** 的
* 热数据应该 **留在内存**，冷数据 **稳定落盘**
* 数据文件应该 **可检查、可重建、可审计**
* 整个引擎应尽量保持 **透明而简洁的内部结构**

---

## ✨ Highlights

### 🔥 **创新 · “三层混合存储架构”**

SinxDB 使用原创的“三层混合结构”：

1. **ActiveTable（活跃表）**

   * 位于内存
   * 用于记录最新写入的数据
   * 可视为热数据区

2. **StableBlocks（稳定块）**

   * 有序、不可变
   * 按时间片存储的键值段文件
   * 用于存储已固化的数据

3. **LogStream（日志流）**

   * 追加写文件
   * 用于崩溃恢复
   * 为 ActiveTable 提供持久性保障

这三者在引擎内部形成一个 **互补的存储循环**：

```
Write → LogStream → ActiveTable → StableBlocks (via consolidation)
```

这种结构带来：

✔ 写入顺畅（顺序写）
✔ 查询有序（StableBlocks 可二分检索）
✔ 恢复快速（LogStream 自描述）
✔ 结构简单可控

---

### ⚡ **高效检索路径**

SinxDB 在检索时会自动依次检查：

1. **ActiveTable（当前内存数据）**
2. **StableBlocks 索引结构**
3. **必要时触发的跳过判定（Bloom-like 机制，可选）**

这种路径在保证数据一致性的情况下自动优化访问成本。

---

### 🧩 **可扩展的模块化设计**

SinxDB 的每个核心模块彼此独立，接口清晰：

* ActiveTable（内存表）
* LogStream（日志流）
* StableBlocks（固化文件）
* Consolidator（固化器，进行块合并）
* CacheLayer（可选缓存层）
* Inspector（文件检查工具）

开发者可以替换或扩展这些模块，使 SinxDB 更适用于：

* 游戏数据存储
* 本地配置存储
* 嵌入式业务 KV
* 本地缓存持久化
* 轻量服务数据层

---

### 💾 **可靠的数据恢复机制**

* 所有写操作均先写入 LogStream
* ActiveTable 不需要长久存在，可通过日志重建
* StableBlocks 为最终一致结构，可重复生成
* 恢复过程可控且透明
* 文件格式严格可解析，便于诊断

---

### 🛠 **简洁的 API**

```go
db, _ := sinxDB.Open("./data")

db.Set("hero", []byte("QinZheng"))
val, _ := db.Get("hero")

db.Remove("hero")
db.Close()
```

---

## 📐 Architecture

```
┌─────────────────────────────────────┐
│               SinxDB                │
├─────────────────────────────────────┤
│   API Layer                         │
├─────────────────────────────────────┤
│   ActiveTable (memory)              │
│   LogStream (append-only file)      │
│   StableBlocks (immutable segments) │
│   Consolidator (merger)             │
├─────────────────────────────────────┤
│   Storage Files                     │
└─────────────────────────────────────┘
```

---

## 🗺 Roadmap

### ✅ v0.1

* ActiveTable（内存结构）
* LogStream 可写入
* 引擎基础操作（Set / Get / Remove）

### 🔜 v0.2

* 稳定块 StableBlocks 写入
* 对稳定块的检索
* 合并器初版

### 🔜 v0.3

* 自动块整合（Consolidator）
* 可选快速跳过结构（Bloom-like）

### 🔜 v0.4

* 数据快照
* 完整恢复机制

### 🔜 v0.5

* TTL 支持
* 前缀扫描与迭代器

### 🔜 v1.0

* 稳定版本发布
* 正式文档、Dashboard 工具

---

## 🧙‍♂️ 原创理念总结

SinxDB 的目标不是成为“最快”或“最强”的数据库，
而是成为 **结构可理解、行为可预测、代码可阅读** 的数据库。

SinxDB的核心关注点是：

* 写入永远顺序化
* 数据文件结构永远可分析
* 组件永远可替换
* 引擎永远可自修复


