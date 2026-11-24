# claude-skills

Personal collection of [Claude Code](https://claude.com/claude-code) skills.

| Skill | What it does |
|---|---|
| [repo-restructure](repo-restructure/) | Reorganizes any repo (single-stack or polyglot) into a clean layered architecture. Plans first, executes on a branch after approval, verifies against a baseline. |

## Install a skill

Copy or clone the skill folder into a Claude Code skills directory, then start a new session.

```powershell
git clone https://github.com/sivamanivenkat/claude-skills.git
# personal (all projects)
Copy-Item -Recurse claude-skills\repo-restructure "$env:USERPROFILE\.claude\skills\repo-restructure"
# or one project only
Copy-Item -Recurse claude-skills\repo-restructure "<project>\.claude\skills\repo-restructure"
```
