# 婚礼请柬 · 双站发布说明

## 架构（两站完全独立，改一个不影响另一个）

| | 站点 1 | 站点 2 |
|---|---|---|
| 访问地址 | https://lzz66.github.io/wedding-invitation/ | https://lzz66.github.io/wedding-invitation2/ |
| GitHub 仓库 | `lzz66/wedding-invitation` | `lzz66/wedding-invitation2` |
| 本地目录 | `wedding-repo\`（源文件在 `wedding\index.html` + `wedding\artistic_photos\`） | `wedding-repo2\` |

## 核心规矩：永久缓存时代，换内容必须换文件名

图片和音乐都通过 jsDelivr CDN 分发，URL 固定到提交哈希（站点 1 是 `@8f065d5`，站点 2 是 `@c7e5cb8`）。
**哈希地址会被 CDN 永久缓存**——这是打开速度快的根源，但代价是：

1. **替换图片/音乐时必须用新文件名**（如 `art_s78.jpg` → `art_s78b.jpg` → `art_s78c.jpg`），否则访客永远看到旧文件。
2. 同名文件改了内容，purge 缓存也救不回来（旧 URL 已被永久缓存）——所以直接用新文件名，新名字没有历史缓存，推上去就是新的。
3. URL 里的哈希**不需要是最新提交**，只要那个提交里包含这个文件就行，平时不要动它。
4. 只有新增了一批文件、想统一重新固定时，才把 URL 哈希整体改成包含这些文件的某个提交。

## 站点 1 修改发布流程

```bash
cd C:\Users\Administrator\Downloads\wedding
# 1. 改 index.html；新图用新文件名放进 artistic_photos\，并更新 HTML 里的引用
cp index.html wedding-repo\index.html
cp artistic_photos\<新图> wedding-repo\artistic_photos\
cd wedding-repo
git add -A && git commit -m "说明" && git push origin main
# 2. 等约 40 秒 GitHub Pages 部署，手机下拉强制刷新验证
```

## 站点 2 修改发布流程

站点 2 可能有人在 GitHub 网页上直接改，**动手前先同步**：

```bash
cd C:\Users\Administrator\Downloads\wedding\wedding-repo2
git pull --rebase origin main
# 改完后
git add -A && git commit -m "说明" && git push origin main
```

## 图片制作惯例

- 海报规格：900×1500（3:5）、q82 渐进式 JPEG，约 150–370 KB/张。
- 生成图左下角的「AI生成」水印必须抹掉：用 `patch_watermark.py`（纸纹采样修补，改图后记得先裁剪缩放再抹）。
- 版式：手机端通栏叠放，桌面端并排；封面 100vw×100vh。

## 验证工具

- 整页截图回归：`python screenshot_preview.py`，输出到 `preview_screenshots\`（9 个分区 + 全页长图）。
- 检查 CDN 文件指纹是否为新图：`md5sum` 对比 `curl -sL <图片URL>` 与本地文件。

## 音乐

- 走 jsDelivr 哈希固定地址 + 12 秒超时回退同源；播放中绝不切源（防卡顿重启）。
- 更换音乐：新文件名或更新 `<source>` 里的哈希；码率 80–128k、体积 <1.5 MB 为宜。
