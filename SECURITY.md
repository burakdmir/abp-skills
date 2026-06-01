# Security Policy

## Supported Versions

| Version | Supported |
|---|---|
| ABP Framework v10.4 | Yes |

## Reporting a Vulnerability

If you discover a security issue in this repository (e.g., skill content that exposes sensitive patterns, insecure code examples), please report it responsibly:

1. **Do NOT** open a public issue
2. Email: [your-email@example.com]
3. Include:
   - Description of the vulnerability
   - Affected file(s)
   - Steps to reproduce
   - Potential impact

You will receive a response within 48 hours.

## Security Best Practices in Skills

All skill files follow these security principles:

- No hardcoded credentials, API keys, or secrets in examples
- Use placeholder values (`"your-connection-string"`, `"smtp-password"`)
- Demonstrate encryption options where applicable (`isEncrypted: true` for settings)
- Warn against production-use of development-only settings (`SendExceptionsDetailsToClients = false`)
