# Claude Code via AWS Bedrock — Student Setup

How we gave students agentic-coding access to Claude Code during MAGICA School 2026, billed through [ESIP](https://www.esipfed.org/)'s AWS credits via Bedrock rather than a direct Anthropic API key (thanks ESIP!)

## Approach

Each student ran Claude Code on their own [Protocoast JupyterHub](https://protocoast.vm.fedcloud.eu/) server, authenticated to Claude via a single shared AWS IAM keypair scoped to Bedrock access. This is the simplest option when per-student usage attribution/rate-limiting isn't a requirement — trade-off is no individual usage tracking, and the shared key must be rotated after the course ends.

## Setup (run in a terminal on the student's JupyterHub server)

```bash
# Give npm a user-writable global prefix (JupyterHub users don't have sudo)
mkdir -p ~/.npm-global
npm config set prefix ~/.npm-global

# Add Claude Code's Bedrock config to the shell profile
cat >> ~/.profile << 'EOF'
export PATH=$HOME/.npm-global/bin:$PATH
export AWS_ACCESS_KEY_ID=<shared IAM access key id>
export AWS_SECRET_ACCESS_KEY=<shared IAM secret access key>
export AWS_REGION=us-west-2
export CLAUDE_CODE_USE_BEDROCK=1
export ANTHROPIC_DEFAULT_SONNET_MODEL=us.anthropic.claude-sonnet-4-6
EOF

npm install -g @anthropic-ai/claude-code

bash ~/.profile
claude
```

## Notes

- `CLAUDE_CODE_USE_BEDROCK=1` tells Claude Code to route through Amazon Bedrock instead of the Anthropic API directly.
- `ANTHROPIC_DEFAULT_SONNET_MODEL` must be a Bedrock cross-region inference profile ID (e.g. `us.anthropic.claude-sonnet-4-6`), not Anthropic's own model name. Check the AWS Bedrock console for current IDs before reusing this for a future cohort — they change as new models ship.
- The IAM keypair was shared across all students (one IAM role/user, same key pasted into every profile). **Rotate or deactivate it once the course ends** — it's a standing shared secret for the duration it's live.
- The `~/.npm-global` prefix step avoids needing sudo to install `@anthropic-ai/claude-code` globally on a shared JupyterHub image.

## Alternatives for a future cohort

If per-student cost attribution, rate-limiting, or individual revocation becomes necessary:

- **AWS Workshop Studio** — auto-provisions an isolated, credit-loaded, auto-expiring AWS account per participant. Best fit if run through AWS's own workshop program.
- **A proxy in front of Bedrock** (e.g. LiteLLM proxy, Bedrock Access Gateway) issuing each student their own API key — gives per-student usage/cost tracking and instant revocation without touching AWS IAM directly.
- **IAM Identity Center (SSO)** — students authenticate individually and get short-lived STS credentials instead of a long-lived shared key.
