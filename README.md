

-----

# 自动同步上游 Worker

[](https://www.google.com/search?q=https://github.com/YOUR_USERNAME/YOUR_REPOSITORY/actions/workflows/auto_update.yml)

这是一个通过 GitHub Actions 实现的自动化项目，旨在定期从上游仓库 `` 同步最新的 Worker 文件。

本项目的核心优势在于，它**不依赖版本号，而是通过比较文件的内容哈希 (SHA256) 来判断是否需要更新**。这确保了即使上游仓库只是修改了文件内容而没有发布新版本，我们也能及时获取到最新的变更。

## 核心功能

  * **定时自动更新**：默认配置为每3小时自动检查一次更新。
  * **内容感知更新**：通过比较 `worker.zip` 的 SHA256 哈希值，确保只有在文件内容真正发生变化时才执行更新，精准高效。
  * **手动触发**：支持通过 GitHub Actions 页面手动运行工作流，并可选择是否“强制更新”。
  * **自动提交**：更新成功后，`github-actions[bot]` 会自动将变更（包括新的 `worker.zip` 和解压后的文件）提交到本仓库。

## 工作原理

本仓库包含一个 GitHub Actions 工作流文件 `.github/workflows/auto_update.yml`，其工作流程如下：

1.  **触发执行**：工作流可以通过以下三种方式触发：

      * **定时任务**：每3小时自动运行。
      * **代码推送**：当 `main` 分支有新的 push 时运行。
      * **手动执行**：在仓库的 "Actions" 标签页点击 "Run workflow" 手动运行。

2.  **获取上游信息**：工作流首先通过 GitHub API 获取  仓库最新 Release 的信息，并找到目标文件 `worker.zip` 的下载地址。

3.  **哈希比较 (核心)**：

      * 计算本地仓库中 `worker.zip` 文件（如果存在）的 SHA256 哈希值。
      * 下载上游的 `worker.zip` 文件到一个临时位置，并计算其 SHA256 哈希值。
      * 比较两个哈希值。

4.  **判断与更新**：

      * **如果哈希值相同** (且未强制更新)，则表示文件内容没有变化，工作流结束。
      * **如果哈希值不同** 或用户选择了“强制更新”，则证明有新内容。工作流会执行以下操作：
        1.  用新下载的 `worker.zip` 覆盖本地的同名文件。
        2.  使用 `unzip -o` 命令解压，覆盖所有旧的 Worker 文件。

5.  **自动提交**：如果上一步执行了更新，一个自动提交步骤会被触发，将所有变动（包括更新后的 `worker.zip` 和解压后的文件）提交回本仓库的 `main` 分支。

## 如何使用

你可以 Fork 本仓库，以同步任何你需要的上游仓库资源。

1.  **Fork 本仓库**。

2.  **修改工作流文件**：进入你 Fork 后的仓库，编辑 `.github/workflows/auto_update.yml` 文件，修改 `env` 部分的配置：

    ```yaml
    env:
      REPO_OWNER: "xxx"  # 修改为目标仓库的拥有者
      REPO_NAME: "xxx"   # 修改为目标仓库的名称
      TARGET_FILE: "worker.zip"       # 修改为需要同步的文件名
    ```

3.  **启用 Actions**：Fork 后的仓库默认禁用 Actions。你需要手动进入仓库的 "Actions" 标签页，点击 "I understand my workflows, go ahead and enable them" 按钮来启用它。

4.  **修改 Badge URL** (可选): 为了让状态徽章生效，请编辑本 `README.md` 文件，将第一行的 URL 中的 `YOUR_USERNAME` 和 `YOUR_REPOSITORY` 替换为你自己的 GitHub 用户名和仓库名。



-----
