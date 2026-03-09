# How to Set Up Your Own AI Assistant (OpenClaw) — Guide From Scratch

> **🤖 For Claude Code:** This file is a step-by-step guide for installing OpenClaw on a VPS. You can execute it automatically. Ask the user to provide: (1) SSH access to the server (IP + key or password), (2) Anthropic API key, (3) Telegram bot token and Telegram ID. Then execute steps 2-4 sequentially via SSH. Steps 0 and 1 should be done by the user themselves (registrations and accounts).

---

## Quick Path (if you have Claude Code)

If you already have Claude Code installed and a VPS rented:

1. Open Claude Code: `claude`
2. Tell it:

```
Read this guide: https://github.com/kalichkin/openclaw-setup-guide

Connect to my server via SSH (root@MY_IP) and execute steps 2-4.
Here's my info:
- Anthropic API key: sk-ant-...
- Telegram bot token: 71234...:AAH...
- My Telegram ID: 123456789
```

3. Claude Code will do the rest. If something goes wrong, it will show the error and suggest a fix.

**If you don't have Claude Code, read the full guide below.**

---

This guide will take you from absolute zero to a working AI assistant available 24/7 via Telegram on your phone.

**What you'll get:** An AI assistant (Claude) that lives on your server, accessible through a Telegram bot. You can message it from your phone anywhere in the world. It can work with files, run code, execute scheduled tasks, and much more.

**Time:** ~45-60 minutes manually (with Claude Code: ~15 minutes)  
**Cost:** ~$5/mo for the server + ~$10-30/mo for AI model API  
**Skill level:** Copy-pasting commands into a terminal is enough

---

## How It Works (Big Picture)

```
  You                              Your Server (VPS)
┌────────┐                        ┌──────────────────────┐
│📱      │── Telegram message ────>│ OpenClaw Gateway     │
│Phone   │                        │   │                  │
│        │<── response ───────────│   ↓                  │
└────────┘                        │ Claude (Anthropic AI) │
                                  └──────────────────────┘
```

OpenClaw is an open-source bridge between messengers and AI. It runs on your server, your data stays with you.

---

## Step 0. What You'll Need Before Starting

Get these ready before you begin:

### 0.1. Anthropic API Key

This key gives access to Claude (the AI model). Without it, OpenClaw can't "think."

1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Sign up (or log in)
3. Add a payment method (Settings → Billing)
4. Go to Settings → API Keys → Create Key
5. Copy the key (starts with `sk-ant-...`) — **save it, it's shown only once!**

Cost: pay-as-you-go. For a personal assistant, typically $10-30/mo.

### 0.2. Telegram Bot

1. Open Telegram, find [@BotFather](https://t.me/BotFather)
2. Send `/newbot`
3. Choose a name for your bot (e.g., "My AI Assistant")
4. Choose a username for your bot (e.g., `my_ai_assistant_bot`) — must end with `bot`
5. BotFather will give you a **token** — a long string like `7123456789:AAH...`. Save it
6. To let the bot read messages in groups, send BotFather: `/setprivacy`, select your bot, choose `Disable`

### 0.3. Find Your Telegram ID

1. Open Telegram, find [@userinfobot](https://t.me/userinfobot)
2. Send it anything
3. It will reply with your numeric ID (e.g., `123456789`). Save it

This ID is needed for the allowlist, so only you can message the bot.

### 0.4. Terminal on Your Computer

- **macOS:** Spotlight → "Terminal" (or iTerm2 if you have it)
- **Windows:** Install [Windows Terminal](https://apps.microsoft.com/detail/9N0DX20HK701) from the Microsoft Store. Then install WSL2: open PowerShell as admin and run `wsl --install`. After restarting, you'll have a Linux terminal
- **Linux:** You already know :)

---

## Step 1. Rent a Server on Hetzner

### 1.1. Create an Account

1. Go to [hetzner.com/cloud](https://www.hetzner.com/cloud/)
2. Click "Sign Up" (or "Get Started")
3. Complete the registration, confirm your email
4. Add a payment method (card or PayPal)
5. You may need identity verification (passport/ID) — Hetzner takes KYC seriously. Usually approved within a few hours

### 1.2. Create an SSH Key (if you don't have one)

An SSH key is a secure way to connect to your server without a password. Set it up once and forget about it.

**Check if you already have a key:**

```bash
ls ~/.ssh/id_ed25519.pub
```

If the file exists, you already have a key. Skip to the next step.

**If you don't have a key, create one:**

```bash
ssh-keygen -t ed25519 -C "your@email.com"
```

Press Enter for all prompts (empty passphrase is fine for now).

**Copy the public key:**

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire line starting with `ssh-ed25519 ...`

### 1.3. Create the Server

1. In Hetzner Cloud Console, click **"Create Server"**
2. **Location:** Choose the nearest region (Falkenstein or Helsinki for EU; Ashburn for US)
3. **Image:** Ubuntu 24.04
4. **Type:** Shared vCPU → **CX22** (2 vCPU, 4GB RAM) — ~€4.50/mo (~$5). More than enough
5. **SSH Key:** Click "Add SSH Key", paste your public key (from step 1.2)
6. **Name:** Give it a name (e.g., `ai-assistant`)
7. Click **"Create & Buy Now"**

The server will be ready in about 30 seconds. Note the **IP address** (visible in the dashboard).

### 1.4. Connect to the Server

```bash
ssh root@YOUR_IP_ADDRESS
```

If asked "Are you sure you want to continue connecting?" type `yes`.

**You're in.** You should see a prompt like `root@ai-assistant:~#`

---

## Step 2. Install OpenClaw on the Server

You're now connected to the server via SSH. All commands below run on the server.

### 2.1. Update the System

```bash
apt update && apt upgrade -y
```

Wait a couple of minutes for everything to update.

### 2.2. Install OpenClaw

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

The script will install Node.js (if needed) and OpenClaw. Takes 2-3 minutes.

### 2.3. Run the Setup Wizard

```bash
openclaw onboard --install-daemon
```

The wizard will ask a few things:
- **API provider:** choose Anthropic
- **API key:** paste your key from Step 0.1
- **Install as service:** Yes (to keep it running 24/7)
- Leave other settings at their defaults

### 2.4. Verify It Works

```bash
openclaw gateway status
```

Should show that the gateway is running and listening.

---

## Step 3. Connect Telegram

### 3.1. Edit the Config

Open the config file:

```bash
nano ~/.openclaw/openclaw.json
```

Find (or add) the `channels` section and add Telegram:

```json
{
  "channels": {
    "telegram": {
      "token": "PASTE_YOUR_BOTFATHER_TOKEN",
      "allowFrom": ["PASTE_YOUR_TELEGRAM_ID"]
    }
  }
}
```

**Important:**
- `token` — the string from BotFather (Step 0.2)
- `allowFrom` — an array of strings (!) with your Telegram ID from Step 0.3

Save the file: `Ctrl+O`, Enter, `Ctrl+X`

### 3.2. Restart the Gateway

```bash
openclaw gateway restart
```

### 3.3. Test It

Open Telegram, find your bot, and send it a message. If everything is set up correctly, it will respond via Claude!

---

## Step 4. Basic Security

A few settings to keep your server secure:

### 4.1. API Keys in .env (Not in Config)

```bash
nano ~/.openclaw/.env
```

Add:

```
ANTHROPIC_API_KEY=sk-ant-your-key-here
```

Then remove the key from `openclaw.json` (OpenClaw will pick it up from `.env` automatically).

```bash
chmod 600 ~/.openclaw/.env
```

### 4.2. Firewall

```bash
apt install -y ufw
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw enable
```

This will block all incoming ports except SSH. The gateway listens on localhost by default, so it's already not accessible from outside.

### 4.3. Automatic Security Updates

```bash
apt install -y unattended-upgrades
dpkg-reconfigure -plow unattended-upgrades
```

Choose "Yes" — the system will automatically install security patches.

---

## Done!

You now have:
- ✅ An AI assistant running 24/7
- ✅ Access via Telegram from your phone
- ✅ Data on your own server (not someone else's)
- ✅ Basic security configured

### What's Next (Optional)

- **Web interface:** From your laptop, run `ssh -L 18789:127.0.0.1:18789 root@YOUR_IP` and open `http://127.0.0.1:18789/` in your browser
- **WhatsApp:** Run `openclaw channels login` on the server and scan the QR code
- **Discord:** Add a Discord bot token to the config
- **Customization:** Set up a workspace, agents, skills — [docs.openclaw.ai](https://docs.openclaw.ai)
- **Security audit:** `openclaw security audit` will show what can be improved

---

## About OpenClaw Security (For the Skeptics)

A fair question when installing an open-source project with shell access.

**Facts:**
- Code is fully open: [github.com/openclaw/openclaw](https://github.com/openclaw/openclaw) (MIT license)
- No telemetry, no "phone home," no cloud dependency
- Data lives only on your VPS — the only external traffic is API calls to the model (Anthropic/OpenAI), which you'd be making anyway
- Formal Threat Model based on the MITRE ATLAS framework
- Built-in security audit: `openclaw security audit --deep`
- Active community (Discord) + security contact (trust.openclaw.ai)
- Security model: one user = one server. Not multi-tenant

**Compared to "conventional" products:**
- Claude.ai / ChatGPT: your data on their servers, their rules, their retention policy
- OpenClaw: your data on your server, your rules, you control everything

**A dedicated VPS for $5/mo is the security model itself.** Even if a bug is found in OpenClaw, it would only affect an isolated server with nothing on it but the assistant.

---

## Links

| What | Where |
|------|-------|
| Documentation | [docs.openclaw.ai](https://docs.openclaw.ai) |
| GitHub | [github.com/openclaw/openclaw](https://github.com/openclaw/openclaw) |
| Discord | [discord.com/invite/clawd](https://discord.com/invite/clawd) |
| Security | [trust.openclaw.ai](https://trust.openclaw.ai) |
| VPS Guides | [docs.openclaw.ai/vps](https://docs.openclaw.ai/vps) |
| Claude Code | [code.claude.com](https://code.claude.com) |

---

## Troubleshooting

**"openclaw: command not found"**
```bash
export PATH="$(npm prefix -g)/bin:$PATH"
echo 'export PATH="$(npm prefix -g)/bin:$PATH"' >> ~/.bashrc
```

**Gateway won't start**
```bash
openclaw doctor          # diagnostics
journalctl -u openclaw-gateway -n 50   # logs
```

**Bot doesn't respond in Telegram**
- Check that the token is correct
- Check that your ID is in `allowFrom` (as a string!)
- Check `openclaw gateway status`
- Check logs: `journalctl -u openclaw-gateway -f`

**Hetzner won't verify your account**
Try DigitalOcean ($6/mo) or Oracle Cloud (free tier). Installation steps are the same.
