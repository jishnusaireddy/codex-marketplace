# Hireko AI Recruiter — Codex Plugin Handoff Guide

This package contains the **Hireko AI Recruiter Plugin** formatted specifically for the **OpenAI ChatGPT Codex** Plugin Marketplace.

---

## 📁 Package Contents

```text
HirekoOpenPlugin/ (or codex/)
├── .codex-plugin/
│   └── plugin.json           # Codex plugin metadata & prompt configuration
├── assets/
│   └── logo.png              # Plugin brand icon for the marketplace UI
├── skills/
│   └── HirekoOpenPlugin/
│       └── SKILL.md          # Natural language recruiter skills & triggers
├── .mcp.json                 # Stdio MCP configuration (runs node bundle.mjs serve)
├── bundle.mjs                # Zero-dependency, standalone server bundle
├── marketplace-entry.json    # Ready-to-copy snippet for marketplace.json
├── README.md                 # Full documentation
├── AGENTS.md                 # Agent routing & tool rules
└── LICENSE                   # MIT License
```

---

## 🚀 How to Add This Plugin to Your Codex Marketplace

### Option A: If you already have a Codex Marketplace Repository

1. **Copy the plugin folder**:
   Extract `HirekoCodexPlugin.zip` and place the folder into your marketplace repo under `plugins/`:
   ```text
   your-marketplace-repo/
   ├── .agents/
   │   └── plugins/
   │       └── marketplace.json
   └── plugins/
       └── HirekoOpenPlugin/     <-- Drop all extracted files here
   ```

2. **Add this entry to your `.agents/plugins/marketplace.json`**:
   Copy and paste this snippet inside your `"plugins": [ ... ]` array:
   ```json
   {
     "name": "HirekoOpenPlugin",
     "description": "Hireko AI Recruiter MCP Router for Codex: candidate interview analytics, AI interview links, and candidate search scorecards.",
     "category": "Recruiting",
     "policy": {
       "installation": "AVAILABLE",
       "authentication": "ON_INSTALL"
     },
     "source": {
       "source": "local",
       "path": "./plugins/HirekoOpenPlugin"
     }
   }
   ```

3. **Commit & Push to GitHub**:
   ```bash
   git add .
   git commit -m "Add HirekoOpenPlugin to Codex marketplace"
   git push
   ```

---

### Option B: If this repository IS the Marketplace

The complete standalone marketplace structure is already pre-configured in `HirekoCodexMarketplace-complete.zip`:
```text
├── .agents/
│   └── plugins/
│       └── marketplace.json
└── plugins/
    └── HirekoOpenPlugin/
        └── (all plugin files)
```

In the Codex App:
- **Source**: `<your-github-org>/<your-repo>` (or local path)
- **Git ref**: `main`
- Click **Add marketplace**.
