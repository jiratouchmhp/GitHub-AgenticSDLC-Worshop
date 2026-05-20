# Prerequisites & install guide

> **Audience:** every workshop attendee. Complete every step here **before** Lab 0 starts. The labs build on each other and assume this environment is in place.

This guide bundles the install commands for macOS, Windows, and Linux, plus how to configure Copilot, the GitHub CLI, and commit signing.

---

## 1. Accounts & licenses (do these first — they often need admin help)

| What | How to confirm |
|---|---|
| A **GitHub Enterprise Cloud** user account in the workshop org | Sign in at <https://github.com> and verify the org appears under your avatar menu |
| An assigned **GitHub Copilot Business** (or Enterprise) seat | Visit <https://github.com/settings/copilot> — you should see "Copilot Business is enabled" |
| Org-level policies for **Coding Agent**, **Code Review**, **MCP servers** | Ask your org owner if any of the lab features are greyed out for you |
| **GitHub Advanced Security** license on the workshop repo (Labs 4–5) | Org Settings → Code security and analysis → confirm GHAS is enabled |

> [!NOTE]
> If your org has no GHAS license, Labs 4 and 5 still work on a **public** repo — CodeQL, Secret Scanning, and Dependabot are free on public repos.

📖 Reference: [About GitHub Copilot for individuals/organizations](https://docs.github.com/en/copilot/get-started/what-is-github-copilot) · [Managing Copilot policies](https://docs.github.com/en/copilot/how-tos/administer/organizations/managing-policies-and-features-for-copilot-in-your-organization)

---

## 2. VS Code (1.95 or newer)

Plan Mode and the Agent Customizations editor require **VS Code 1.95+**. Update if you're on anything older.

**macOS**

```bash
brew install --cask visual-studio-code
# or download from https://code.visualstudio.com/download
```

**Windows (PowerShell, admin)**

```powershell
winget install -e --id Microsoft.VisualStudioCode
```

**Linux (Debian/Ubuntu)**

```bash
sudo apt update && sudo apt install -y code
# or follow https://code.visualstudio.com/docs/setup/linux
```

Verify:

```bash
code --version
# Expect 1.95.x or newer
```

📖 Reference: [VS Code setup overview](https://code.visualstudio.com/docs/setup/setup-overview)

---

## 3. Node.js 20 LTS + npm 10+

The workshop stack is fixed to Node 20 LTS. Anything older breaks `better-sqlite3` and Vite 5.

**Recommended: install via `nvm`** so you can pin Node 20 without affecting other projects.

**macOS / Linux**

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
# Restart your shell, then:
nvm install 20
nvm use 20
nvm alias default 20
```

**Windows (PowerShell)**

Use [nvm-windows](https://github.com/coreybutler/nvm-windows/releases) or:

```powershell
winget install -e --id OpenJS.NodeJS.LTS
```

Verify:

```bash
node --version    # v20.x.x
npm  --version    # 10.x.x or newer
```

📖 Reference: [Node.js downloads](https://nodejs.org/en/download) · [nvm](https://github.com/nvm-sh/nvm)

---

## 4. Git 2.40+ and identity

**macOS**

```bash
brew install git
```

**Windows**

```powershell
winget install -e --id Git.Git
```

**Linux (Debian/Ubuntu)**

```bash
sudo apt update && sudo apt install -y git
```

Configure your identity (used on every commit):

```bash
git --version            # expect 2.40 or newer
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase false
```

📖 Reference: [Git — first-time setup](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup)

---

## 5. Commit signing (REQUIRED for Lab 2's ruleset)

Lab 2 turns on **Require signed commits** on `main`. Pick **one** option below. If you skip this, you can still merge by going through the GitHub web UI (which signs via `web-flow`) — but inline `git push` to `main` will be rejected.

### Option A — SSH commit signing (simplest if you already use SSH for Git)

```bash
# Generate (or reuse) an Ed25519 SSH key
ssh-keygen -t ed25519 -C "you@example.com"

# Tell Git to sign commits with that key
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

Then add the **same public key** to GitHub under <https://github.com/settings/keys> as a **Signing key** (not just an Authentication key).

📖 Reference: [Telling Git about your SSH signing key](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key#telling-git-about-your-ssh-key)

### Option B — GPG commit signing

```bash
# Install GPG
brew install gnupg                # macOS
sudo apt install -y gnupg         # Linux
winget install -e --id GnuPG.GnuPG  # Windows

# Generate a key (accept defaults; use the same email as your GitHub account)
gpg --full-generate-key

# Get the key ID
gpg --list-secret-keys --keyid-format=long

# Export the public key and add it under https://github.com/settings/keys (GPG key)
gpg --armor --export <KEY_ID>

# Tell Git
git config --global user.signingkey <KEY_ID>
git config --global commit.gpgsign true
```

📖 Reference: [Generating a GPG key](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key)

### Option C — Use the GitHub web UI / `gh` CLI for merges

Both sign automatically via GitHub's `web-flow` key. No local setup needed — just remember to merge PRs through the website or `gh pr merge --squash`.

Test your setup:

```bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
# Should print "Good signature from ..."
```

---

## 6. GitHub CLI (`gh`) 2.50+

General-purpose GitHub automation (repo creation, auth, ad-hoc queries). Lab 3 itself uses the GitHub MCP Server (§9) rather than `gh` for issue creation, but the CLI is still useful across the workshop.

**macOS**

```bash
brew install gh
```

**Windows**

```powershell
winget install -e --id GitHub.cli
```

**Linux (Debian/Ubuntu)**

```bash
# See https://github.com/cli/cli/blob/trunk/docs/install_linux.md for the full key+repo setup
sudo apt update && sudo apt install -y gh
```

Authenticate:

```bash
gh --version                  # 2.50+
gh auth login                 # follow the prompts; pick HTTPS + browser
gh auth status                # should print "Logged in to github.com as <you>"
```

Confirm you can see the workshop org:

```bash
gh org list
```

📖 Reference: [gh CLI installation](https://github.com/cli/cli#installation) · [gh auth](https://cli.github.com/manual/gh_auth)

---

## 7. VS Code extensions

Install all three. The Pull Requests extension is what powers the *"Delegate to coding agent"* button used in Lab 3.

```bash
code --install-extension GitHub.copilot
code --install-extension GitHub.copilot-chat
code --install-extension GitHub.vscode-pull-request-github
```

Then in VS Code:

1. Open the **Accounts** menu in the lower-left → **Sign in to GitHub**.
2. Open the Command Palette → **GitHub Copilot: Sign In Status** — confirm you have a Business/Enterprise seat.
3. Open the Chat view (`Ctrl/Cmd + Alt + I`) → click the agents dropdown at the top → confirm **Plan**, **Agent**, and **Edit** are all listed.

📖 Reference: [Copilot in VS Code — setup](https://code.visualstudio.com/docs/copilot/setup) · [Copilot Chat](https://code.visualstudio.com/docs/copilot/chat/copilot-chat)

---

## 8. (Recommended) Quick scratch directory for the workshop

You'll create the project in Lab 0. Pick a sensible location now:

```bash
mkdir -p ~/code/workshops && cd ~/code/workshops
```

---

## 9. GitHub MCP Server (remote) — used in Lab 3

Lab 3 creates issues from Copilot Chat using the **GitHub MCP Server** — then delegates each issue to the Coding Agent via the `/delegate` slash command, the chat **Cloud** mode, or github.com (no MCP needed for delegation itself). The **remote** server (hosted by GitHub) is the easiest path — no Docker, no PAT, OAuth sign-in is enough.

### Install in VS Code (one-click)

Click the install button from the [GitHub MCP Server README](https://github.com/github/github-mcp-server#remote-github-mcp-server), or add the config manually.

### Install manually (`.vscode/mcp.json` workspace-scoped or user-scoped `mcp.json`)

```jsonc
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    }
  }
}
```

Reload VS Code. Open Copilot Chat → click the **tools** icon (`🛠`) above the input box and confirm the **github** server is listed with the `issues` toolset enabled.

> [!NOTE]
> First use will trigger an OAuth flow in your browser. Sign in with the same GitHub account that holds your Copilot seat.

### Verify the tool Lab 3 needs

In Copilot Chat, type `#` and start typing `create_issue` — it must autocomplete. If it doesn't:

- Confirm the **MCP servers on GitHub.com** org policy is enabled (cross-reference [§1 Accounts & licenses](#1-accounts--licenses-do-these-first--they-often-need-admin-help)).
- Confirm the `issues` toolset is not filtered out (check the tools picker — it should be ON).

Also confirm the **/delegate** slash command is available in chat (the GitHub Pull Requests extension provides it — listed under §7).

📖 Reference: [github/github-mcp-server README](https://github.com/github/github-mcp-server) · [Remote server documentation](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md) · [Use MCP servers in VS Code](https://code.visualstudio.com/docs/copilot/chat/mcp-servers)

---

## ✅ Final preflight checklist

Run this end-to-end. Every line should print a sensible version or "OK":

```bash
node --version           # v20.x.x
npm  --version           # 10.x.x or newer
git  --version           # 2.40+
gh   --version           # 2.50+
gh   auth status         # "Logged in to github.com as <you>"
code --version           # 1.95+
code --list-extensions | grep -i 'github\.\(copilot\|copilot-chat\|vscode-pull-request-github\)'
# Expect all three extension IDs to appear
```

Then in a browser:

- [ ] <https://github.com/settings/copilot> shows your seat is active
- [ ] You can see the workshop organization under your account avatar
- [ ] (If using SSH/GPG signing) <https://github.com/settings/keys> lists your signing key

And in Copilot Chat:

- [ ] Typing `#create_issue` autocompletes (GitHub MCP Server — §9)
- [ ] The `/delegate` slash command is recognized (GitHub Pull Requests extension — §7)
- [ ] The agents dropdown above the chat input lists a **Cloud** mode (used in Lab 3)

You're ready. Head to **[Lab 0 — Bootstrap: Custom Agents + Skills](./lab-00-bootstrap-plan-agents.md)**.

---

## 🩹 Troubleshooting

| Symptom | Fix |
|---|---|
| `node` is on the wrong version after install | `nvm use 20 && nvm alias default 20` (macOS/Linux) or restart your shell after nvm-windows install |
| `gh auth login` keeps failing | Try `gh auth login --hostname github.com --web` and copy the one-time code into the browser |
| `code` command not found (macOS) | In VS Code: Command Palette → **Shell Command: Install 'code' command in PATH** |
| Copilot Chat is missing the Plan/Agent dropdown | Update VS Code to 1.95+ and update the Copilot Chat extension; sign out + back in |
| `commit.gpgsign` fails with "secret key not available" | Run `git config --global user.signingkey <ID>` with the correct key id from `gpg --list-secret-keys --keyid-format=long` |
| Push to `main` rejected for unsigned commits in Lab 2 | Pick one of the three signing options above, or merge PRs via the GitHub web UI (signed by `web-flow`) |
