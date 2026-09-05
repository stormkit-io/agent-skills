# Agent skills

Skills for coding agents, in the [Agent Skills](https://agentskills.io) format —
a `SKILL.md` with its reference files beside it. The format is an open standard,
so a skill here is plain Markdown any agent can be pointed at, and Claude Code
can install it as a plugin.

## The skills

| Skill | What it does |
| --- | --- |
| [corgy:demo](plugins/corgy/skills/demo) | Plans a product demo from your codebase — scenes, a recording prompt for each, and the narration — then hands it to the [Corgy](https://corgy.ai) macOS app as a demo sheet, which you record scene by scene. |

## Install

In Claude Code, add the marketplace once and then take what you want:

```
/plugin marketplace add stormkit-io/agent-skills
/plugin install corgy@stormkit
```

Adding the marketplace installs nothing on its own. It registers the catalogue;
each plugin is installed by name.

## Using them elsewhere

Every skill's frontmatter sticks to the six fields in the Agent Skills spec —
`name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools` —
with no runtime-specific extensions. So a skill directory can be copied into
whatever your agent reads, or handed to it as context directly.

## Layout

```
.claude-plugin/marketplace.json   the catalogue
plugins/<name>/
├── .claude-plugin/plugin.json    what gets installed
└── skills/<name>/SKILL.md        the skill itself
```

One copy of each skill, in the plugin that ships it.

## Licence

MIT. See [LICENSE](LICENSE).
