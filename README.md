# s-skills

自用的 Agent Skill 库，按需往 `skills/` 里加 skill。

当前 skill：

- `kb-route`：把知识库查找和可复用知识写入路由到 Obsidian vault
- `kimi-code`：告知其他 agent 如何调用 Kimi Code CLI（`kimi`）
- `local-wsl-env`：本机 WSL 特有执行环境（目前仅 ssh-askpass / `sudo -A`）
- `zhibaidian`（直白点）：揪出生造词、"准确但读者不需要"的术语（门控、闭环、合同这类）和"不是 X 而是 Y"式没人问过的对立项，改回直白说法；用户说"直白点"或点名时启用，启用后也约束 agent 在这次任务里自己写的内容；中英文同规则；和 shuorenhua 分工，只管"正确但费解"
