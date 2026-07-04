# Security Policy

## Responsible Use

VulnRecon performs active reconnaissance (subdomain probing, historical URL analysis, third-party API queries). Only run it against:
- Domains/assets you own, **or**
- Targets explicitly covered by a bug bounty program's scope, **or**
- Systems you have written authorization to test (e.g., signed pentest agreement)

Unauthorized scanning may be illegal in your jurisdiction. Contributors and users are solely responsible for how they use this tool.

## Reporting a Vulnerability in VulnRecon Itself

If you discover a security issue in VulnRecon's own code (e.g., credential leakage, insecure deserialization, command injection via crafted target input), please **do not open a public GitHub issue**. Instead:

1. Email the maintainers at `security@<your-domain>.example` (replace with a real contact before publishing)
2. Include a proof-of-concept and affected version/commit
3. Allow a reasonable disclosure window (suggested: 90 days) before public disclosure

We will acknowledge receipt within 5 business days.

## Known Security-Relevant Design Notes

- **API keys via CLI flags** (`--shodan`, `--telegram`) are visible in shell history and process listings (`ps aux`). Use `config.txt` (excluded from git) or environment variables instead, and never pass secrets on a shared/multi-user machine's command line.
- **No input sanitization guarantee for subprocess calls** to external tools (`subfinder`, `httpx`, etc.) — target strings should be validated as legitimate domains before being interpolated into any subprocess command to avoid command/argument injection. Review this in any PR that touches subprocess invocation.
- **Wayback/Shodan responses are treated as trusted input** — malformed or adversarial API responses are not defended against beyond basic `.get()` fallbacks. Do not run this tool against attacker-controlled Shodan/Wayback proxies.
