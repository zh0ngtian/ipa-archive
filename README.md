# IPA Archive

保存第三方已编译的 iOS IPA，通过 GitHub Releases 提供下载，供 SignOs 软件源订阅。

- 仓库为公开仓库，Release 附件可通过 HTTPS 地址直接下载。
- IPA、校验文件与构建信息放在 Releases 附件中；Git 工作树只保存说明和索引。
- 本仓库不自动构建或重新签名 IPA；上传时保留包的原始字节。

## Release 组织

每个应用的每个版本建立独立 Release：

- Tag：`<app-slug>-v<version>-b<build>`，例如 `example-app-v1.0.0-b12`。
- Title：`<App Name> <version> (<build>)`。
- 附件：应用 `.ipa` 与 `SHA256SUMS`；有可追溯构建信息时，一起上传 `build-info.json`。
- 描述：应用名称、Bundle ID、包内版本与构建号、来源、编译者、编译方式及更新说明。
- 同一应用的不同分支或构建变体使用不同的 app slug，避免误当成同一升级渠道。
- 已发布的包使用具体版本下载链接；新包创建新版本，不覆盖历史包。

这是多应用仓库。聚合器应遍历 Releases，并按应用或明确的 tag 前缀选择版本；
不要把整个仓库的 `releases/latest` 当成每个应用的最新版本。
Release tag 只用于分类；Bundle ID、应用版本与构建号以 IPA 内的信息为准。

## 上传 IPA

先计算原始包的 SHA-256：

```sh
shasum -a 256 /path/App.ipa > /path/SHA256SUMS
```

准备 Release 描述文件 `/path/release-notes.md` 后，使用 GitHub CLI：

```sh
gh release create example-app-v1.0.0-b12 \
  /path/App.ipa /path/SHA256SUMS \
  --repo zh0ngtian/ipa-archive \
  --title 'Example App 1.0.0 (12)' \
  --notes-file /path/release-notes.md
```

将示例应用名、tag、版本、构建号和路径替换为实际值。
也可以在 GitHub 的 Releases 页面创建版本并拖入附件。
如包内版本与构建号相同，但重新编译后的内容不同，应记录差异并使用新的唯一 tag，
不要替换已发布附件。是否属于可升级版本仍由软件源根据包内版本与构建号判断。

GitHub 要求每个 Release 附件小于 2 GiB，详见
[About releases](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases)。

## Release 描述模板

```text
应用名称：
Bundle ID：
版本：
构建号：
最低 iOS 版本：
来源项目或页面：
源代码 commit / tag（已知时）：
编译者与编译日期（已知时）：
构建变体或修改说明：
更新说明：
```

无法确认的构建信息标记为未知，保留实际可追溯的来源。

## 软件源接入约定

聚合软件源需要同时支持两类输入：

1. **GitHub Release 订阅**：仓库地址、Release tag 前缀或应用标识、IPA 附件匹配规则、
   是否包含 prerelease；用于自动发现后续版本。
2. **直接 IPA 地址**：HTTPS 下载地址及必要的人工元数据；后台临时下载解析，
   公开软件源保留原始下载地址。优先使用明确版本的链接。

后台统一提取包内标识、版本、构建号、最低系统版本，记录大小和 SHA-256，生成软件源条目。
直接地址的更新由人工替换链接或重新检查内容触发；ETag 和 Last-Modified 仅用于辅助判断变化。

此文件记录接入约定，仓库当前不包含聚合服务或 SignOs 客户端实现。
