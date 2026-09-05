# Actions Bundle 发布（workers-world/java-actions）

Java 仓通过 **semver tag** 引用可复用 workflow，**禁止** `@master`。

| 项 | 约定 |
|----|------|
| Tag 格式 | `actions/vX.Y.Z` |
| 当前版本 | [manifest/actions-bundle.yaml](../manifest/actions-bundle.yaml) |
| Caller | `uses: workers-world/java-actions/.github/workflows/java-maven-verify.yml@actions/v0.5.0` |
| 跨仓嵌套 | 全路径 `workers-world/java-actions/.github/…@actions/vX.Y.Z` |

合入 `master` 且改了 workflows/actions 时由 [`release-actions-bundle.yml`](../.github/workflows/release-actions-bundle.yml) 打 tag。tag push 触发 [`gh-release-on-tag.yml`](../.github/workflows/gh-release-on-tag.yml)（leaf 在 [worker-actions `create-gh-release`](https://github.com/workers-world/worker-actions/blob/master/.github/workflows/create-gh-release.yml)）。详见 [worker-actions gh-release.md](https://github.com/workers-world/worker-actions/blob/master/docs/gh-release.md)。

**发版鉴权（双通道，变量开关）**：配置 Org/Repo Variable `RELEASE_BOT_APP_ID` + Secret `RELEASE_BOT_PRIVATE_KEY` 后自动走 GitHub App 短时 installation token（与 worker-actions 同一枚 `workers-world-release-bot`，`contents:write` + `pull-requests:write`，token 仅限本仓）；未配置回落 `GHA_TOKEN`/`GITHUB_TOKEN`。tag push 须由 App/PAT 发起（`GITHUB_TOKEN` 推的 tag 不触发下游 workflow），故 App/PAT 缺失时发版链会断在 `gh-release-on-tag`。App 迁移背景见 [worker-actions docs/github-app-token-migration.md](https://github.com/workers-world/worker-actions/blob/master/docs/github-app-token-migration.md)。

先有 tag，再 bump 消费者。

Worker 侧 Release PR / OCR / Qodana 见 [workers-world/worker-actions](https://github.com/workers-world/worker-actions)。
