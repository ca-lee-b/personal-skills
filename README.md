# Personal Agent Skills

A collection of personal AI agent skills compatible with **GitHub Copilot**, **Claude Code**, **OpenCode**, and other tools supporting the [Agent Skills](https://skills.sh) open standard.

These skills are designed to automate repetitive workflows and extend the capabilities of AI coding assistants.

## 🚀 Quick Start

### Install via skills.sh

You can install these skills using the `skills` CLI:

```bash
# Install all skills
npx skills add ca-lee-b/personal-skills

# Install a specific skill
npx skills add ca-lee-b/personal-skills --skill create-prd
```

## 🛠 Available Skills

| Skill | Description | Trigger Keywords |
|-------|-------------|------------------|
| [**create-prd**](./skills/create-prd/SKILL.md) | Synthesizes conversation into a concise, implementation-ready PRD. | "write a PRD", "create a PRD", "document requirements" |
| [**implement-prd**](./skills/implement-prd/SKILL.md) | Executes implementation based on a PRD document. Best suited and optimized for `create-prd` output. | "implement this PRD", "build the spec" |

## Getting the Best Performance

I personally use these skills with Opencode, which allows
me to define custom agents. For example, I have a frontend agent
using Kimi K2.6 and backend agent using GLM 5.1.

This allows me to delegate the PRD to the agent best suited
for a requirement.

## 📄 License

MIT
