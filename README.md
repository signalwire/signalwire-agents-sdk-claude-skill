# SignalWire AI Agents SDK - Claude Code Skill

A comprehensive Claude Code skill for building SignalWire AI Agents in Python. This skill automatically activates when working with AgentBase, SWAIG functions, SWML, voice configuration, or any `signalwire_agents` code.

## Installation

### Option 1: Personal Skill (Single User)

Copy the skill to your Claude Code skills directory:

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Copy the skill
cp -r /path/to/signalwire-agents-sdk ~/.claude/skills/
```

The skill will be available in all your Claude Code sessions.

### Option 2: Project Skill (Per-Project)

Copy the skill into a specific project:

```bash
# From your project root
mkdir -p .claude/skills
cp -r /path/to/signalwire-agents-sdk .claude/skills/
```

The skill will be available when working in that project.

### Option 3: SDK Repository Integration

For the SignalWire Agents SDK repository itself, place the skill at:

```
signalwire-agents/.claude/skills/signalwire-agents-sdk/
```

This makes the skill available to all SDK users automatically.

You should see `signalwire-agents-sdk` listed.

## Usage

The skill activates automatically when you:

- Import from `signalwire_agents`
- Create classes extending `AgentBase`
- Work with SWAIG functions or tools
- Configure voice, language, or TTS
- Use DataMap for server-side functions
- Ask about SWML, prompts, or call flow
- Deploy agents (serverless, Docker, multi-agent)

### Example Prompts

```
"Create a customer service agent that can look up orders and transfer to humans"

"Add a SWAIG function to check account balance"

"How do I configure voice for my agent?"

"Set up a multi-agent server with support and sales agents"

"Add debug webhooks to monitor my agent"
```

## Skill Contents

```
signalwire-agents-sdk/
├── SKILL.md                     # Main skill instructions
├── troubleshooting.md           # Common issues and solutions
├── reference/
│   ├── agent-base.md            # AgentBase class reference
│   ├── agent-server.md          # Multi-agent deployment
│   ├── environment-variables.md # Environment configuration
│   ├── function-result.md       # SwaigFunctionResult actions
│   ├── signalwire-integration.md # Fabric API, WebRTC, phone
│   ├── swaig-functions.md       # Function definition patterns
│   └── webhooks-debugging.md    # Debug webhooks
├── patterns/
│   ├── common-patterns.md       # Frequently used patterns
│   ├── error-handling.md        # Error handling best practices
│   ├── security.md              # Security best practices
│   └── testing.md               # Testing with swaig-test
└── examples/
    ├── simple-agent.py          # Basic agent template
    ├── faq-bot.py               # FAQ bot with skills
    ├── multi-agent-server.py    # Multiple agents + static files
    ├── datamap-agent.py         # Server-side API functions
    └── webrtc-enabled-agent.py  # Browser-based voice
```

## Publishing

### Publish to GitHub

1. Create a new repository:

```bash
cd /path/to/signalwire-agents-sdk
git init
git add .
git commit -m "Initial commit: SignalWire AI Agents SDK skill for Claude Code"
```

2. Push to GitHub:

```bash
gh repo create signalwire-agents-sdk-claude-skill --public --source=. --push
```

Or manually:

```bash
git remote add origin https://github.com/signalwire/signalwire-agents-sdk-claude-skill.git
git push -u origin main
```

### Share Installation Instructions

Users can install directly from GitHub:

```bash
# Clone to personal skills
git clone https://github.com/signalwire/signalwire-agents-sdk-claude-skill.git \
    ~/.claude/skills/signalwire-agents-sdk

# Or clone to project
git clone https://github.com/signalwire/signalwire-agents-sdk-claude-skill.git \
    .claude/skills/signalwire-agents-sdk
```

### Bundle with SignalWire Agents SDK

To include the skill with the SDK itself:

```bash
# From signalwire-agents repo root
mkdir -p .claude/skills
cp -r /path/to/signalwire-agents-sdk .claude/skills/

git add .claude/skills/signalwire-agents-sdk
git commit -m "Add Claude Code skill for AI Agents SDK"
git push
```

This makes the skill automatically available to anyone who clones the SDK.

## Development

### Testing Examples

The skill includes a Python virtual environment for testing:

```bash
cd signalwire-agents-sdk

# Create venv (if not exists)
python3 -m venv .venv
.venv/bin/pip install signalwire-agents

# Test examples
cd examples
../.venv/bin/swaig-test simple-agent.py --dump-swml
../.venv/bin/swaig-test faq-bot.py --list-tools
../.venv/bin/swaig-test multi-agent-server.py --agent-class SupportAgent --list-tools
```

### Updating the Skill

When the SDK changes:

1. Update reference documentation in `reference/`
2. Update patterns in `patterns/`
3. Test examples still work
4. Update version/changelog if maintained

## License

MIT License - See the SignalWire Agents SDK for full license terms.

## Links

- [This Skill on GitHub](https://github.com/signalwire/signalwire-agents-sdk-claude-skill)
- [SignalWire AI Agents SDK](https://github.com/signalwire/signalwire-agents)
- [SignalWire Documentation](https://developer.signalwire.com)
- [Claude Code](https://claude.ai/code)
