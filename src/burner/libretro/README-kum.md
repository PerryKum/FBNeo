# CI / Release 备忘

**默认 push 什么都不跑。**

## 只构建

```bash
git commit -m "xxx [ci]"
git push
```

## 构建 + 发 Release

```bash
git commit -m "xxx [release]"
git push
```

`[release]` 会自动跑 CI，不用再加 `[ci]`。

## 手动

Actions → **Build libretro cores** → Run workflow  
勾选 **Publish GitHub release** 才会发 Release，不勾只构建。
