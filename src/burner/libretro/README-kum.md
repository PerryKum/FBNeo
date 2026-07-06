# FinalBurn Neo (Kum) — libretro 构建与部署备忘

本 fork 在 libretro 核心上做了 **CRC 回退 ROM 加载**（`FBNEO_CRC_FALLBACK=1`），用于替换 AURKNIX / ROCKNIX 上的 **fbneoplus** 槽位。

---

## 关键文件

| 文件 | 说明 |
|------|------|
| `libretro.cpp` | CRC 回退、中文名 zip + 同目录 BIOS 等加载逻辑 |
| `Makefile.common` | `FBNEO_CRC_FALLBACK=1`，版本后缀 `-perrykum` |
| `fbneoplus_libretro.info` | 核心描述文件，`display_version` 在此维护 |
| `.github/workflows/build-libretro.yml` | CI 构建 workflow |

当前显示版本：`v1.0.0.03-perrykum`（改版本时同步改 info 和 workflow 里的 release 标签前缀）。

---

## CI 怎么跑

**触发条件：** 每次 push 到 `master` 分支。

**构建产物（5 个架构，并行）：**

| Artifact 名称 | 平台 |
|---------------|------|
| `fbneo_libretro-linux-x86_64` | Linux x86_64 `.so` |
| `fbneo_libretro-linux-aarch64` | Linux aarch64 `.so`（掌机用这个） |
| `fbneo_libretro-linux-armv7` | Linux armv7 `.so` |
| `fbneo_libretro-windows-x86_64` | Windows 64 位 `.dll` |
| `fbneo_libretro-windows-x86` | Windows 32 位 `.dll` |

下载路径：GitHub → **Actions** → 选一次运行 → 底部 **Artifacts**。

---

## 什么时候发 Release

**默认 push 只构建，不发 Release。**

### 方式一：commit message 带 `[release]`

```bash
git commit -m "fix: 某改动 [release]"
git push
```

只有 commit message 里包含 **`[release]`** 且全部 job 成功时，才会创建 GitHub Release。

普通 push（不带 `[release]`）示例：

```bash
git commit -m "fix: 调整 CRC 逻辑"
git push
```

→ 只构建 + Artifacts，**不会**发 Release。

### 方式二：GitHub 手动触发

1. 打开 **Actions** → **Build libretro cores**
2. 点 **Run workflow**
3. 勾选 **Publish GitHub release**
4. Run

适合不 push 代码、或单独补发 Release 的情况。

**Release 标签格式：** `v1.0.0.03-perrykum-<run_number>`（每次唯一）

---

## AURKNIX / ROCKNIX 部署

掌机是 **aarch64**，不要用 x86_64 的 `.so`。

### 从 Release 下载（推荐）

Release 里已包含改名好的文件：

- `fbneoplus_libretro.so`（即 aarch64 构建）
- `fbneoplus_libretro.info`

### 从 Artifacts 下载

1. 下载 `fbneo_libretro-linux-aarch64`
2. 将 `fbneo_libretro.so` **改名为** `fbneoplus_libretro.so`
3. 从仓库复制 `fbneoplus_libretro.info`

### 放到掌机

```
/storage/cores/fbneoplus_libretro.so
/storage/cores/fbneoplus_libretro.info
/storage/roms/fbneo/          ← 游戏 ROM（zip）
```

BIOS（如 `neogeo.zip`）可与游戏 zip 放在同一目录；核心会按 CRC 识别。

---

## ROM 加载说明（简要）

- 认 zip **内文件的 CRC/大小**，不是只看 zip 文件名
- 标准 zip 名找不到时，会走 **CRC 回退**：按内容识别 driver，并在同目录（含一层子目录）找 parent / BIOS
- 非标准文件名（如中文名 zip）+ 同目录标准 BIOS 已支持
- 改版 / hack 仍需独立 driver 或 patched 机制，CRC 回退不能替代

---

## 本地开发备忘

- 本地 Windows 一般编不出 aarch64；掌机用的 `.so` 靠 **GitHub Actions** 交叉编译
- 改 workflow 后若 `.gitignore` 忽略了 `.github`，提交时用：

  ```bash
  git add -f .github/workflows/build-libretro.yml
  ```

- 更新 `display_version` 时改 `fbneoplus_libretro.info`；若 release 标题/标签前缀也绑版本，同步改 `build-libretro.yml` 里 `tag_name` / `name` 字段

---

## 快速对照

| 我想… | 怎么做 |
|--------|--------|
| 只测构建 | 普通 `git push` |
| push 并发 Release | commit message 加 `[release]` 再 push |
| 手动发 Release | Actions → Run workflow → 勾选 Publish |
| 装到掌机 | Release 里下 `fbneoplus_libretro.so` + `.info` → `/storage/cores/` |
| 装到 PC RetroArch | Release 里下对应 Windows `.dll` 或 Linux x86_64 `.so` |
