# Java / Maven + GitHub Packages

通过 GitHub Packages Maven 消费/发布 Java 制品。可复用 workflow 在 **workers-world/java-actions**（须位于 `.github/workflows/` 顶层）。

`distributionManagement` / `<repositories><url>` 使用 `https://maven.pkg.github.com/<owner>/<repo>`，与源码仓库 org 一致。

## 可复用 Workflow

| Workflow | 用途 |
|----------|------|
| [java-maven-verify.yml](../.github/workflows/java-maven-verify.yml) | 业务仓 verify（可选 MySQL、JaCoCo、SBOM） |
| [java-maven-publish.yml](../.github/workflows/java-maven-publish.yml) | SDK 仓 deploy |
| [java-maven-sdk-ci.yml](../.github/workflows/java-maven-sdk-ci.yml) | SDK 仓 verify；caller 传 `publish_on_push` 时在 `dev_*` push 后 publish |

Composite：[configure-maven-github-packages](../.github/actions/java/configure-maven-github-packages/action.yml)、[ensure-maven](../.github/actions/ensure-maven/action.yml)。

Release PR（qodana / OCR / auto-merge）仍用 [workers-world/worker-actions](https://github.com/workers-world/worker-actions)；Java 业务仓模板见 [templates/java/ci-release-pr.yml](../templates/java/ci-release-pr.yml)。

## Maven 与 npm 权限差异

| 注册表 | 权限模型 |
|--------|----------|
| **npm** scoped packages | 细粒度；Package 页可 Manage Actions access |
| **Maven** | **仅仓库级**；无 Manage Actions access；权限继承 SDK **源码仓库** |

消费方：

- 不要在 Maven 包设置页找 npm 那套 Actions access
- 使用 classic PAT（`read:packages`），且 PAT 所属账号对 SDK 源码仓库有 **Read**
- 跨仓库消费须用 PAT，不能指望 `GITHUB_TOKEN` 访问其它仓的包

## 一次性配置

| Secret / 权限 | 说明 |
|---------------|------|
| `GITHUB_PACKAGES_TOKEN` 或 `GHA_TOKEN`（兼容旧名 `WORKERS_WORLD_GHA_TOKEN`） | classic PAT：消费 `read:packages` / 发布 `write:packages` |
| SDK 源码仓库 Read | Maven 包权限继承该仓库 |
| caller `permissions.packages` | verify：`read`；publish：`write` |

## SDK 仓

1. `pom.xml` 配置 `distributionManagement`（`server <id>` 与 CI `server_id` 一致）
2. 复制 [templates/java/sdk-ci.yml](../templates/java/sdk-ci.yml) 为 `.github/workflows/ci.yml`，改 `github_packages_repo` / `server_id`
3. **勿**在 caller `ci.yml` 顶层声明与 reusable 同名的 `concurrency`
4. push `dev_*` → verify 绿后 `mvn deploy`

## 业务仓

1. `pom.xml` 增加 [maven-repositories-pom.xml.snippet](../templates/java/maven-repositories-pom.xml.snippet) 中的 `<repositories>`
2. 版本号写在 `dependencyManagement` / `<dependency>`
3. verify job 调用 `java-maven-verify.yml`（见仓库 README 示例）

## server id

`package_repos` 支持 `repo` 或 `repo:serverId`。须与 pom 的 `<repository><id>` / `distributionManagement` 一致。

## 常见问题

| 现象 | 处理 |
|------|------|
| Package 页没有 Manage Actions access | Maven 正常；查 PAT + SDK 源码仓 Read |
| `403` clone SDK 源码 | 改用 Packages + `java-maven-verify` |
| `401/403` Maven resolve | PAT 含 `read:packages`；账号能读 SDK 仓；caller `packages: read` |
| `404` Maven resolve | 版本未 publish，或 pom `<url>` 指向错误 org/repo |
| deploy 失败 | `packages: write`；`distributionManagement` id 与 `server_id` 一致 |
