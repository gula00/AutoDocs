# AutoClaw macOS 打包、签名、公证

本文档适用于在 macOS 上产出可分发的 `dmg`/`zip`，并完成 Apple 签名与公证。

## 1. 前置要求

- macOS（Apple Silicon + Xcode Command Line Tools）
- Apple Developer Program 账号
- `Developer ID Application` 证书已导入钥匙串
- Node.js + npm
- 项目根目录有 `.env` 文件（包含签名和公证凭据）

检查基础环境：

```bash
xcode-select -p
node -v
npm -v
security find-identity -v -p codesigning  # 确认证书已导入
```

## 2. 签名证书准备

当前使用的证书：`Developer ID Application: Beijing Knowledge Atlas Technology Joint Stock Company Limited (8A5X4JJ39T)`

证书来源：`/Users/zhaohanlin/Downloads/证书.p12`（密码：`autoglm`）

导入方式：

```bash
security import /path/to/证书.p12 -k ~/Library/Keychains/login.keychain-db -P autoglm -T /usr/bin/codesign
```

在 `.env` 中配置（二选一）：

```bash
# 方式一：通过 .p12 文件（CI 推荐）
CSC_LINK="/absolute/path/DeveloperID_Application.p12"
CSC_KEY_PASSWORD="<p12_password>"

# 方式二：通过钥匙串中的证书名
APPLE_IDENTITY="Developer ID Application: Beijing Knowledge Atlas Technology Joint Stock Company Limited (8A5X4JJ39T)"
```

## 3. 公证凭据

### 方案 A：Apple ID（当前使用）

```bash
APPLE_ID="your-apple-id@example.com"
APPLE_APP_SPECIFIC_PASSWORD="xxxx-xxxx-xxxx-xxxx"
APPLE_TEAM_ID="8A5X4JJ39T"
```

App-specific password 在 https://appleid.apple.com 生成。

### 方案 B：App Store Connect API Key（CI 推荐）

```bash
APPLE_API_KEY="/absolute/path/AuthKey_XXXXXX.p8"
APPLE_API_KEY_ID="XXXXXX"
APPLE_API_ISSUER="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

## 4. .env 完整示例

```bash
APPLE_IDENTITY="Developer ID Application: Beijing Knowledge Atlas Technology Joint Stock Company Limited (8A5X4JJ39T)"
APPLE_ID="your-apple-id@example.com"
APPLE_APP_SPECIFIC_PASSWORD="xxxx-xxxx-xxxx-xxxx"
APPLE_TEAM_ID="8A5X4JJ39T"
CSC_LINK="/path/to/证书.p12"
CSC_KEY_PASSWORD="autoglm"
```

## 5. 打包流程（手动，推荐）

由于本地网络代理会导致 `electron-builder` 内置公证超时，当前采用**分步手动流程**：
`electron-builder.yml` 中 `notarize: false`，公证步骤手动执行。

### 第一步：构建 + 打包签名

```bash
# 清理旧产物
rm -rf dist/mac-arm64 dist/autoclaw-*.dmg dist/AutoClaw-*.zip dist/*.blockmap

# 加载 .env 签名凭据
set -a && source .env && set +a

# 构建 + 打包（签名由 electron-builder 自动完成）
npm run build
npx electron-builder --mac --publish never
```

输出：
- `dist/mac-arm64/AutoClaw.app` — 已签名的 app
- `dist/autoclaw-<version>.dmg` — DMG 安装包
- `dist/AutoClaw-<version>-arm64-mac.zip` — zip 包

### 第二步：公证 DMG

**重要：必须先关闭代理**，否则上传到 Apple S3 会超时。

```bash
# 关闭代理
unset http_proxy https_proxy all_proxy

# 加载凭据
set -a && source .env && set +a

# 提交公证（会等待 Apple 处理，通常 3-10 分钟）
xcrun notarytool submit dist/autoclaw-<version>.dmg \
  --apple-id "$APPLE_ID" \
  --password "$APPLE_APP_SPECIFIC_PASSWORD" \
  --team-id "$APPLE_TEAM_ID" \
  --wait
```

如果超时断开，可以用 submission ID 查询状态：

```bash
xcrun notarytool info <submission-id> \
  --apple-id "$APPLE_ID" \
  --password "$APPLE_APP_SPECIFIC_PASSWORD" \
  --team-id "$APPLE_TEAM_ID"
```

### 第三步：Staple 公证票据

公证通过（`status: Accepted`）后，将票据附加到文件：

```bash
xcrun stapler staple dist/mac-arm64/AutoClaw.app
xcrun stapler staple dist/autoclaw-<version>.dmg
```

### 第四步：验证

```bash
# 验证 app 签名 + 公证
codesign --verify --deep --strict --verbose=2 dist/mac-arm64/AutoClaw.app
spctl -a -t exec -vv dist/mac-arm64/AutoClaw.app
# 期望输出: accepted, source=Notarized Developer ID

# 验证 DMG staple
xcrun stapler validate dist/autoclaw-<version>.dmg
# 期望输出: The validate action worked!
```

## 6. 打包流程（release 脚本，备选）

如果网络环境没有代理干扰，也可以用一键脚本：

```bash
bash scripts/release-mac.sh
```

脚本会自动执行：加载 .env → build → electron-builder → staple app → notarize DMG → staple DMG → 验证签名。

**注意**：脚本会遍历 `dist/` 下所有 DMG 做 staple，打包前建议清理旧产物。

## 7. 环境区分

代码中有两处根据 `NODE_ENV` 区分测试/正式环境：

### 模型服务 URL（`src/shared/constants.ts`）

| 环境 | URL |
|------|-----|
| `development`（npm run dev） | `https://autoglm-inner-3.zhipuai.cn/autoclaw-proxy/proxy/autoclaw` |
| `production`（build 出包） | `https://autoglm-api.zhipuai.cn/autoclaw-proxy/proxy/autoclaw` |

### 登录/用户 API（`src/main/auth/endpoints.ts`）

| 环境 | URL |
|------|-----|
| `development` / `test` | `https://autoglm-test-2.zhipuai.cn` |
| `pre` | `https://autoglm-pre-api.zhipuai.cn` |
| `production` | `https://autoglm-api.zhipuai.cn` |

如果正式环境后端未上线，需要临时让出包也走测试环境，修改这两处即可。

## 8. 构建配置要点

### electron-builder.yml

- `notarize: false` — 公证手动做（避免代理超时）
- `extraResources` 包含 `resources/gateway`（255MB，gateway 运行必需）
- `npmRebuild: false`

### electron.vite.config.ts

- `express` 和 `@datarangers/sdk-electron` 从 externalize 列表排除，直接打包进 bundle
- 原因：这两个纯 JS 包被 externalize 后，深层依赖（如 `call-bind-apply-helpers`）在 asar 中缺失

## 9. 常见问题

### 公证超时 `HTTPClientError.connectTimeout`

本地 shell 设了代理（`http_proxy`/`https_proxy`/`all_proxy`），导致上传 Apple S3 失败。

解决：`unset http_proxy https_proxy all_proxy` 后重试。

### `Cannot find module 'call-bind-apply-helpers'`

`express` 被 externalize 导致深层依赖未打进 asar。

解决：确保 `electron.vite.config.ts` 中 `externalizeDepsPlugin({ exclude: ['express', '@datarangers/sdk-electron'] })`。

### `OpenClaw entry not found at: .../gateway/openclaw/openclaw.mjs`

`resources/gateway` 未打进包。

解决：确保 `electron-builder.yml` 的 `extraResources` 包含：
```yaml
- from: resources/gateway
  to: gateway
  filter:
    - '**/*'
```

### `No identity found` / 签名失败

证书未导入钥匙串或 `CSC_*` / `APPLE_IDENTITY` 配置不正确。

```bash
security find-identity -v -p codesigning
```

### DMG staple 失败 `Record not found`

DMG 尚未完成公证。等公证状态变为 `Accepted` 后再 staple。

### 旧 DMG 干扰 release 脚本

`scripts/release-mac.sh` 会遍历 `dist/` 下所有 `.dmg` 做 staple。打包前先清理：

```bash
rm -rf dist/mac-arm64 dist/autoclaw-*.dmg dist/AutoClaw-*.zip dist/*.blockmap
```
