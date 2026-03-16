# AWS CLI Skills for OpenClaw 🦞🚀

A comprehensive library of high-density OpenClaw-compatible Agent Skills for the Amazon Web Services (AWS) Command Line Interface.

Built for engineers who want to orchestrate cloud infrastructure with the speed of an AI assistant.

## 🛠️ Included Skills

| Skill | Emoji | Description |
|-------|-------|-------------|
| **[aws-s3](skills/aws-s3/SKILL.md)** | 🪣 | High-performance storage management and syncing. |
| **[aws-lambda](skills/aws-lambda/SKILL.md)** | λ | Orchestrate serverless compute and cloud rendering. |
| **[aws-iam](skills/aws-iam/SKILL.md)** | 🔐 | Security-first identity and access auditing. |
| **[aws-ec2](skills/aws-ec2/SKILL.md)** | 💻 | Elastic compute lifecycle and network security. |
| **[aws-cloudwatch](skills/aws-cloudwatch/SKILL.md)** | 📈 | Advanced observability and automated monitoring. |

## 🚀 Installation

To use these skills in your OpenClaw environment:

1. **Clone the repo** into your workspace:
   ```bash
   cd ~/.openclaw/workspace/skills
   git clone https://github.com/ordiy/aws-cli-skills.git
   ```

2. **Register the skills**:
   OpenClaw will automatically discover the `SKILL.md` files in these subdirectories. Ensure you have the `aws` CLI installed and configured on your host.

## 🧠 Philosophy: "Tech Life, Touch & Free"

This library follows the **Watadot Studio** philosophy:
- **Tech Life**: Deep, source-level technical logic.
- **Touch**: Meaningful connection between human vision and AI execution.
- **Free**: Liberation through high-level automation.

## ⚠️ Requirements
- AWS CLI v2
- Valid AWS Credentials (configured via `aws configure` or environment variables)
- Python 3.x (for some advanced filtering patterns)

---
*Maintained by Claw (AI Agent) for ping chan.*
