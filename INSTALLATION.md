# Installation Guide

## 1. Prerequisites

| Requirement | Minimum Version | Notes |
|---|---|---|
| Python | 3.9+ | `asyncio`, dataclasses, type hints used throughout |
| Go | 1.20+ | Required to build ProjectDiscovery/tomnomnom tools |
| pip | latest | For Python dependencies |
| OS | Linux (Kali/Ubuntu/Debian recommended) | macOS works; Windows via WSL2 |

## 2. Clone the repository

```bash
git clone https://github.com/<your-username>/vulnrecon.git
cd vulnrecon
```

## 3. Create a virtual environment

```bash
python3 -m venv venv
source venv/bin/activate      # Linux/macOS
# venv\Scripts\activate       # Windows
```

## 4. Install Python dependencies

```bash
pip install -r requirements.txt
```

`requirements.txt` (suggested contents for this repo):

```
aiohttp>=3.9
rich>=13.7
python-dotenv>=1.0
```

## 5. Install external recon tools

VulnRecon shells out to well-known Go-based recon tools. Install them and confirm each is on `$PATH`:

```bash
go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install github.com/projectdiscovery/httpx/cmd/httpx@latest
go install github.com/tomnomnom/waybackurls@latest
go install github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
go install github.com/ffuf/ffuf/v2@latest

# Add Go bin to PATH if not already present
export PATH=$PATH:$(go env GOPATH)/bin
```

Standard system tools also expected: `dig`, `nmap`, `curl` (install via `apt install dnsutils nmap curl` on Debian/Kali).

## 6. Verify installation

```bash
python3 vulnrecon.py --version
python3 test.py          # if the test/dependency-check script is included
```

You should see a dependency status table confirming `subfinder`, `httpx`, `waybackurls`, `nuclei`, `dig`, `nmap`, and `curl` are all marked **Available**.

## 7. Configure API keys (optional but recommended)

Copy the example config and fill in your own keys — **never commit real keys**:

```bash
cp config.example.txt config.txt
nano config.txt
```

Alternatively, use environment variables (preferred, once implemented per the roadmap):

```bash
export VULNRECON_SHODAN_KEY="your_key_here"
export VULNRECON_TELEGRAM_TOKEN="bot_token"
export VULNRECON_TELEGRAM_CHAT_ID="chat_id"
```

## 8. Run your first scan

```bash
python3 vulnrecon.py -t example.com --live-preview
```

## Docker (planned — see Roadmap)

A `Dockerfile` bundling Python + Go tool binaries is on the roadmap so contributors can run:

```bash
docker build -t vulnrecon .
docker run --rm -it vulnrecon -t example.com
```

If you'd like to contribute this, see [CONTRIBUTING.md](../CONTRIBUTING.md).

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `command not found: subfinder` | Go bin not on PATH | `export PATH=$PATH:$(go env GOPATH)/bin` |
| Scan hangs at JS Discovery | Older `V1`/unpatched script, no timeout handling | Use the V3.2+ codebase which adds timeouts |
| No Shodan results | Free-tier API key limits, or query returned 0 matches | Verify key with `curl "https://api.shodan.io/api-info?key=YOUR_KEY"` |
| Telegram notifications silent | Bad `BOT_TOKEN:CHAT_ID` format | Format must be exactly `token:chat_id`, no spaces |
| `ModuleNotFoundError: rich` | Dependencies not installed in the active venv | Re-run `pip install -r requirements.txt` inside the venv |
