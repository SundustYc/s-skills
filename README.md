# s-skills

自用的 Agent Skill 库，按需往 `skills/` 里加 skill。

当前 skill：

- `bieshengzao`（别生造）：揪出文档里费解的生造词，改回直写内容（压缩 vs 生造、标签概括结论、不占用既有词、缩写展开、行文可复述）；中英文同规则；和 shuorenhua 分工，只管"正确但费解"
- `kb-route`：把知识库查找和可复用知识写入路由到 Obsidian vault
- `kimi-code`：告知其他 agent 如何调用 Kimi Code CLI（`kimi`）
- `local-wsl-env`：本机 WSL 特有执行环境（目前仅 ssh-askpass / `sudo -A`）
