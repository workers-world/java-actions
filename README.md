# workers-world/java-actions

Java / Maven 仓可复用的 GitHub Actions（verify、Packages publish、SDK CI）。

许可证由 GitHub 建仓时生成（MIT）。**不要** `@master`，pin `actions/vX.Y.Z`。

Worker CI（Release PR / Qodana / OCR）在独立仓：[workers-world/worker-actions](https://github.com/workers-world/worker-actions)。Java 业务仓模板会同时 `uses` 两个仓。

## Caller

SDK 仓：

```yaml
jobs:
  ci:
    permissions:
      contents: read
      packages: write
    uses: workers-world/java-actions/.github/workflows/java-maven-sdk-ci.yml@actions/v0.5.0
    secrets: inherit
    with:
      github_packages_repo: your-sdk-repo
      server_id: github-your-sdk
      # 含 github.ref_type == 'branch' 守卫：防止同名 dev_* tag 触发 publish
      publish_on_push: ${{ github.event_name == 'push' && startsWith(github.ref_name, 'dev_') && github.ref_type == 'branch' }}
```

业务仓 verify：

```yaml
jobs:
  verify:
    permissions:
      contents: read
      packages: read
    uses: workers-world/java-actions/.github/workflows/java-maven-verify.yml@actions/v0.5.0
    secrets: inherit
    with:
      github_packages_repos: your-sdk:github-your-sdk
      enable_mysql: true
```

模板：[templates/java/](templates/java/)。说明：[docs/java/maven-github-packages.md](docs/java/maven-github-packages.md)。

嵌套 composite 已 pin 全路径 `workers-world/java-actions/.github/actions/…@actions/v0.5.0`（勿用 `$/`）。

## 调用方配置（名称，不含值）

| 名称 | 类型 | 用途 |
|------|------|------|
| `GITHUB_PACKAGES_TOKEN` 或 `GHA_TOKEN` | Secret | classic PAT：`read:packages` / `write:packages`。**跨仓拉包必填**（缺省 fail-fast，不再静默回落 `GITHUB_TOKEN`——它拉不了跨仓 Maven Packages）；发布到**本仓** Packages 可省（走 `GITHUB_TOKEN` + `packages: write`） |
| `GHA_RUNNER` | Variable | 空则 `ubuntu-latest`；在 **caller** 上下文求值。可被各 workflow 的 `runner` input 覆盖（fork PR 场景须显式传 `ubuntu-latest`，防 Org Variable 把 fork PR 路由到 self-hosted） |
| `RELEASE_BOT_APP_ID` | Variable | 可选。本仓发版（release-actions-bundle）走 GitHub App 的开关：配置即自动启用 App token（与 worker-actions 同一枚 `workers-world-release-bot`），未配置回落 `GHA_TOKEN` |
| `RELEASE_BOT_PRIVATE_KEY` | Secret | 同上，App 私钥（PEM） |

`java-maven-publish` **禁止**挂到 `pull_request` / `pull_request_target`。

本仓自己的 CI **写死 `ubuntu-latest`**。
