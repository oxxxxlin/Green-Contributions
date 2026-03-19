# 🌱 Green-Contributions

这是一个通过 GitHub Actions 自动生成提交记录的仓库，旨在帮助你保持 GitHub 贡献图的活跃度。只需配置一次，即可实现：

· 每天自动提交：北京时间 4:00 自动生成多次提交（默认 12 次），让你的贡献格每天保持深绿色。
· 批量填充历史：支持手动触发，一次性生成从指定日期到今天的提交记录，用于补全过去的贡献日历。

所有操作均在 GitHub 网页端完成，无需本地环境。

---

✨ 功能特性

· ⏰ 每日定时任务
    利用 GitHub Actions 的 schedule 触发器，每天 UTC 20:00（北京时间次日 4:00）运行，自动生成 12 次空提交并推送至仓库。
· 📅 手动批量填充
    当你想补全历史贡献时，可通过 workflow_dispatch 手动触发工作流，设置开始日期、结束日期和每日提交次数，系统将自动生成指定日期范围内的所有提交。
· 🔧 灵活配置
    一个工作流文件同时处理两种场景，代码逻辑清晰，易于修改（如调整提交次数、运行时间等）。

---

📁 仓库结构

```
.github/
└── workflows/
    └── fill-history.yml       # 主工作流文件（包含每日自动 + 手动批量）
```

---

🚀 快速开始

1. 创建仓库（如果尚未创建）

· 点击右上角 + → New repository
· 输入仓库名称（例如 green-contributions）
· 选择 Public
· 勾选 Add a README file（可选）
· 点击 Create repository

2. 添加工作流文件

· 进入仓库，点击 Add file → Create new file
· 在文件路径中输入：.github/workflows/fill-history.yml
· 将下面的代码粘贴到编辑区（请务必修改邮箱和用户名为你自己的）：

```yaml
name: Fill History (Daily Auto + Manual Batch)

on:
  schedule:
    - cron: '0 20 * * *'          # 每天 UTC 20:00（北京时间 4:00）
  workflow_dispatch:
    inputs:
      start_date:
        description: '开始日期 (YYYY-MM-DD)'
        required: true
        default: '2025-04-23'
      end_date:
        description: '结束日期 (YYYY-MM-DD) - 留空则默认为今天'
        required: false
        default: ''
      daily_commits:
        description: '每天提交次数 (用于历史填充)'
        required: true
        default: '12'

jobs:
  fill:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - name: Configure Git
        run: |
          git config --global user.email "你的邮箱"      # 替换为你的 GitHub 验证邮箱
          git config --global user.name "你的用户名"     # 替换为你的 GitHub 用户名

      - name: Generate commits
        env:
          START_DATE: ${{ github.event.inputs.start_date }}
          END_DATE: ${{ github.event.inputs.end_date }}
          DAILY_COMMITS: ${{ github.event.inputs.daily_commits }}
        run: |
          if [ "${{ github.event_name }}" == "schedule" ]; then
            # 定时触发：生成今天提交
            TODAY=$(date +%Y-%m-%d)
            COMMITS_PER_DAY=12
            echo "Daily run: generating $COMMITS_PER_DAY commits for $TODAY"
            for i in $(seq 1 $COMMITS_PER_DAY); do
              git commit --allow-empty -m "daily fill #$i on $TODAY"
            done
          else
            # 手动触发：生成历史提交
            if [ -z "$END_DATE" ]; then
              END_DATE=$(date +%Y-%m-%d)
            fi
            current_date="$START_DATE"
            while [ "$current_date" != "$END_DATE" ]; do
              echo "Generating $DAILY_COMMITS commits for $current_date"
              for i in $(seq 1 $DAILY_COMMITS); do
                export GIT_AUTHOR_DATE="$current_date 12:00:00"
                export GIT_COMMITTER_DATE="$current_date 12:00:00"
                git commit --allow-empty -m "chore: sync #$i on $current_date"
              done
              current_date=$(date -d "$current_date + 1 day" +%Y-%m-%d)
            done
          fi

      - name: Push commits
        run: |
          if [ "${{ github.event_name }}" == "schedule" ]; then
            git push origin HEAD          # 定时任务：普通推送
          else
            git push --force origin HEAD  # 手动任务：强制推送（覆盖历史）
          fi
```

· 在页面下方填写提交信息，例如 “Add auto-green workflow”，然后点击 Commit new file。

3. 验证自动运行

· 进入 Actions 标签页，左侧点击 Fill History (Daily Auto + Manual Batch)。
· 如果一切正常，第二天北京时间 4:00 会看到一条新的运行记录，事件（Event）为 schedule。
· 你也可以立即手动触发一次来测试（点击 Run workflow，保持默认参数，运行后会在今天生成 12 次提交）。

---

🛠️ 自定义配置

参数 位置 说明
每天提交次数 第 45 行 COMMITS_PER_DAY=12 定时任务每次生成的提交数（≥6 次即可达到深绿色）
运行时间 第 6 行 cron: '0 20 * * *' 使用 UTC 时间，例如 0 20 * * * = 北京时间 4:00
邮箱和用户名 第 23-24 行 必须替换为你在 GitHub 验证过的邮箱和用户名
手动填充默认开始日期 第 12 行 default: '2025-04-23' 可根据需要修改

---

📖 使用方法

每日自动提交（无需干预）

配置好并提交文件后，系统会自动在北京时间每天 4:00 运行，生成 12 次空提交。你只需在第二天查看 Actions 日志或贡献图确认即可。

批量填充历史

· 进入 Actions 标签页。
· 左侧点击本工作流，右侧点击 Run workflow。
· 填写参数：
  · 开始日期：你想要填充的起始日期（如 2025-01-01）
  · 结束日期：留空则默认为今天；若指定日期，填充到该日期前一天
  · 每天提交次数：建议 12 次，确保深绿色
· 点击 Run workflow 开始执行。
· 注意：批量填充会强制推送，覆盖当前分支的所有历史提交。如果你有其他重要代码，建议先备份。

---

⚠️ 注意事项

· 邮箱验证：提交时使用的邮箱必须与你的 GitHub 账户中验证的邮箱一致，否则贡献不会被计入。
· 默认分支：工作流会自动推送到当前分支（通常为 main 或 master）。确保该分支为仓库的默认分支。
· 权限设置：如果遇到推送失败，请检查仓库 Settings → Actions → General → Workflow permissions，确保已勾选 Read and write permissions。
· 贡献图更新延迟：GitHub 贡献图可能有几小时到一天的缓存，新产生的提交不会立即变色，请耐心等待。
· 合理设置提交次数：每天 10-15 次足以达到深绿色，无需设置过高，以免仓库历史过于庞大。
· 强制推送风险：手动批量填充会强制覆盖历史，如果你希望保留旧记录，建议提前创建分支备份。

---

❓ 常见问题

Q: 为什么我配置后第二天没有自动运行？

A: 请检查：

· Actions 页面是否有失败的记录？
· 仓库的 Actions 权限是否开启？（Settings → Actions → General → Allow all actions）
· 工作流文件中的邮箱和用户名是否正确？
· 是否修改了 cron 表达式？UTC 时间与本地时间的转换是否正确？

Q: 手动填充时，结束日期留空和指定日期有什么区别？

A: 留空则自动填充到今天（包含今天）；指定日期则填充到该日期的前一天（例如开始 2025-01-01，结束 2025-01-05，会填充 1月1日-4日）。

Q: 我可以将工作流用在其他仓库吗？

A: 当然可以。只需将本文件复制到任意仓库的 .github/workflows/ 目录下，并修改邮箱和用户名即可。

Q: 私有仓库的贡献会显示在个人主页吗？

A: 会的。只要提交邮箱经过验证，且推送到默认分支，无论仓库是公开还是私有，贡献都会计入你的公开贡献图。

---

📄 开源协议

本项目采用 MIT 协议，你可以自由使用、修改和分发。

---

Happy Contributing! 💚
