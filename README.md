# BADSC-web

BADSC 的静态网站仓库，负责网站文件与部署说明。

**[在线使用](https://zgggg.top/BADSC-web/)** · [源码、Issue 与 PR](https://github.com/Zggggg123/BADSC)

功能开发和测试在 BADSC 完成；本仓库接收其完整 `web/` 内容。维护者合并 PR 后，独立决定何时更新网站。卡组文件和分享码的转换全部在浏览器内完成。

## 手动更新网站

以下命令在 PowerShell 中执行，示例假设开发仓库为 `D:\BADSC`，网站仓库为 `D:\BADSC-web`。请按实际路径修改。首次准备网站仓库：

```powershell
git clone https://github.com/Zggggg123/BADSC-web.git D:\BADSC-web
```

1. 在 BADSC 完成功能开发并提交改动，运行测试：

   ```powershell
   Set-Location D:\BADSC
   npm test
   npm run test:browser
   git status --short
   ```

   浏览器测试需要开发依赖及 Chromium，环境准备见 [BADSC 开发说明](https://github.com/Zggggg123/BADSC/blob/main/docs/DEVELOPMENT.md)。确认测试通过、工作区干净后再同步。

2. 确认网站仓库没有未提交改动，拉取最新版本，再复制完整网页资源：

   ```powershell
   Set-Location D:\BADSC-web
   git status --short
   git pull --ff-only origin main
   Copy-Item -Path D:\BADSC\web\* -Destination . -Recurse -Force
   ```

   `web/` 中的文件放到网站仓库根目录，不再套一层 `web/`。完整保留历史字典、WASM 和 `vendor/LICENSE`，并保留本仓库的 `.nojekyll`、`.gitattributes`、README 及根目录许可证文件。若第三方来源或许可证发生变化，也同步 BADSC 的 `LICENSE`、`THIRD-PARTY-NOTICES.md` 和 `licenses/`。复制不会删除旧文件；发现已废弃资源时，先核对引用再单独移除，不要删除旧分享码需要的字典快照。

3. 如果网页资源有变化，更新 `release.json` 的来源提交和资源 SHA-256。以下命令保留现有许可文件的校验项，并纳入完整 `web/` 资源：

   ```powershell
   $release = Get-Content .\release.json -Raw | ConvertFrom-Json
   $assets = [ordered]@{}
   foreach ($property in $release.assets.PSObject.Properties) {
     $assets[$property.Name] = (Get-FileHash -LiteralPath $property.Name -Algorithm SHA256).Hash.ToLowerInvariant()
   }
   $webRoot = (Resolve-Path D:\BADSC\web).Path
   Get-ChildItem -LiteralPath $webRoot -File -Recurse | ForEach-Object {
     $relative = $_.FullName.Substring($webRoot.Length + 1).Replace('\', '/')
     $assets[$relative] = (Get-FileHash -LiteralPath $relative -Algorithm SHA256).Hash.ToLowerInvariant()
   }
   $release.sourceCommit = (git -C D:\BADSC rev-parse HEAD).Trim()
   $release.preview = $false
   $release.assets = $assets
   $json = ($release | ConvertTo-Json -Depth 10) + "`n"
   [System.IO.File]::WriteAllText((Join-Path (Get-Location).Path 'release.json'), $json, [System.Text.UTF8Encoding]::new($false))
   ```

   若移除资源或新增许可文件，相应调整清单，确保每项对应实际文件。仅更新本 README 时不修改 `release.json`，它记录当前网页资源的来源。

4. 核对改动并使用 Git 命令提交、推送：

   ```powershell
   git diff --stat
   git diff --check
   git add --all
   git diff --cached --stat
   git commit -m "Update website from BADSC"
   git push origin main
   ```

5. 在本仓库 Actions 中确认 GitHub Pages 部署成功，再打开网站检查上传、复制分享码、还原与下载。推送成功不等于部署成功。

## 部署与维护约定

GitHub Pages 使用 `main` 分支根目录。域名、HTTPS 和 Pages 设置在网站仓库管理；功能贡献提交到 BADSC。

字典采用内容寻址，必须保留原始字节。本仓库 `.gitattributes` 禁止行尾转换；不要覆盖或删除历史字典快照。发布失败时先检查 Actions 日志，回退使用正常的新提交，不强制推送。

目前使用手动同步和 Git 命令，不提供双击更新脚本。

## 许可证

原创代码见 [MIT 许可证](LICENSE)。第三方来源见 [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md)，内置 Brotli WASM 的许可证见 [vendor/LICENSE](vendor/LICENSE)。
