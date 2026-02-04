# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 1.0.x   | :white_check_mark: |

## Reporting a Vulnerability

We take security seriously. If you discover a security vulnerability, please follow these steps:

### 1. Do NOT Create a Public Issue

Please do not disclose security vulnerabilities through public GitHub issues.

### 2. Email Us

Send your report to: **security@a2n.io**

Include:
- Type of vulnerability
- Steps to reproduce
- Potential impact
- Any suggested fixes (optional)

### 3. What to Expect

- **Acknowledgment**: Within 48 hours
- **Status Update**: Within 7 days
- **Resolution**: Depends on severity (critical: <7 days, high: <30 days)

### 4. Disclosure Policy

- We'll work with you to understand and resolve the issue
- We'll credit you (unless you prefer anonymity) in our changelog
- We ask that you give us reasonable time to fix before public disclosure

## Security Best Practices

When deploying a2n.io:

1. **Use HTTPS** - Always use a reverse proxy with SSL in production
2. **Set ENCRYPTION_KEY** - Encrypt stored credentials
3. **Strong Passwords** - Use complex database passwords
4. **Network Security** - Don't expose the database port publicly
5. **Regular Updates** - Pull the latest Docker image regularly
6. **Backup** - Regularly backup your data volume

## Security Features

- AES-256-GCM encryption for credentials
- JWT tokens with expiration
- Password hashing with bcrypt
- CORS protection
- Rate limiting
- Input validation

Thank you for helping keep a2n.io secure! 🔒
