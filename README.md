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
      publish_on_push: ${{ github.event_name == 'push' && startsWith(github.ref_name, 'dev_') }}
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
| `GITHUB_PACKAGES_TOKEN` 或 `GHA_TOKEN`（兼容旧名 `WORKERS_WORLD_GHA_TOKEN`） | Secret | classic PAT：`read:packages` / `write:packages` |
| `GHA_RUNNER`（兼容旧名 `WORKERS_WORLD_GHA_RUNNER`） | Variable | 空则 `ubuntu-latest`；在 **caller** 上下文求值 |

`java-maven-publish` **禁止**挂到 `pull_request` / `pull_request_target`。

本仓自己的 CI **写死 `ubuntu-latest`**。
