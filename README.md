# Extended fail2ban Configuration for Apache

A collection of advanced fail2ban filters and jail configurations designed to protect Apache web servers from automated probes, enumeration attacks, and sensitive file discovery attempts.

## Overview

This project provides three complementary fail2ban security modules that detect and block malicious scanning activity targeting Apache web servers. Each module uses pattern matching on Apache access logs to identify attack signatures and trigger automatic IP bans through UFW (Uncomplicated Firewall).

## Modules

### 1. PHP Script Probe Detection (`apache-php-probe-extended`)

**File:** `filter.d/apache-php-probe-extended.conf`

Detects attempts to probe for executable scripts on the server, including both common and obfuscated file extensions.

**Matched Patterns:**
- `.php` and `.php` variants (`.php3`, `.php4`, etc.)
- `.phtml` (PHP HTML files)
- `.phar` (PHP archives)
- `.cgi` (CGI scripts)
- `.asp` and `.aspx` (ASP extensions)
- Case-insensitive matching for evasion attempts

**Smart Exclusions:**
- `index.php` - standard entry point
- `wp-login.php` - WordPress login
- `wp-cron.php` - WordPress background tasks
- `xmlrpc.php` - XML-RPC interface
- `wp-admin/admin-ajax.php` - WordPress AJAX handler

**Activation Rules:**
- **Max Retries:** 6 failed probes
- **Time Window:** 2 minutes
- **Ban Duration:** 1 day

### 2. Sensitive File & Secret Discovery (`apache-secret-scan-extended`)

**File:** `filter.d/apache-secret-scan-extended.conf`

Detects probes for sensitive configuration files, framework files, and credentials that could expose application secrets.

**Matched Patterns:**
- `.env` files (environment configuration) - including URL-encoded variants
- `.git/config` and `.git/HEAD` (Git repository files)
- `.aws/credentials` and `.aws/config` (AWS credentials)
- `composer.json` and `composer.lock` (PHP dependency files)
- `config.php` and backup variants (`.bak`, `.old`, `~`)
- `phpinfo.php` (PHP information disclosure)
- `debug/default/view` (Framework debug pages)
- `admin/config` (Admin configuration)
- `server-status` (Apache status page)
- `HNAP1` (Device administration protocol)
- `cgi-bin/*` (CGI executable directory)

**Activation Rules:**
- **Max Retries:** 3 failed probes
- **Time Window:** 10 minutes
- **Ban Duration:** 1 week (aggressive - indicates serious reconnaissance)

### 3. WordPress Enumeration Probe Detection (`apache-wp-enum-probe`)

**File:** `filter.d/apache-wp-enum-probe.conf`

Targets WordPress-specific enumeration and probing attempts that are high-signal indicators of malicious scanning.

**Matched Patterns:**
- `wp-admin/*` (WordPress admin directory)
- `wp-includes/*` (WordPress core files)
- `wp-content/plugins/*` (Plugin directory enumeration)
- `wp-json/wp/v2/users*` (API user enumeration)
- `xmlrpc.php` (XML-RPC attacks)

**Activation Rules:**
- **Max Retries:** 12 failed probes
- **Time Window:** 5 minutes
- **Ban Duration:** 12 hours

## Installation

### Prerequisites
- fail2ban installed and running
- UFW (Uncomplicated Firewall) installed and configured
- Apache with access logging enabled

### Setup

1. Copy filter files to fail2ban filter directory:
   ```bash
   sudo cp filter.d/*.conf /etc/fail2ban/filter.d/
   ```

2. Copy jail configuration:
   ```bash
   sudo cp jail.d/20-web-extended.local /etc/fail2ban/jail.d/
   ```

3. Verify the configuration:
   ```bash
   sudo fail2ban-client status
   ```

4. Reload fail2ban to activate:
   ```bash
   sudo systemctl reload fail2ban
   ```

## Configuration

All three jails are pre-configured and enabled in `jail.d/20-web-extended.local`. They monitor `/var/log/apache2/access.log` with UFW as the blocking action.

### Customization

To adjust ban parameters, edit `jail.d/20-web-extended.local`:

- **maxretry:** Number of matches before banning (lower = more aggressive)
- **findtime:** Time window in minutes for counting matches
- **bantime:** Ban duration in seconds (use `1d`, `1w` for days/weeks)
- **logpath:** Path to Apache access log if different
- **action:** Change firewall action if not using UFW

### Testing Filters

Test a filter without banning:
```bash
sudo fail2ban-regex /var/log/apache2/access.log /etc/fail2ban/filter.d/apache-php-probe-extended.conf
```

## Monitoring

Check active jails:
```bash
sudo fail2ban-client status
```

Check specific jail status:
```bash
sudo fail2ban-client status apache-php-probe-extended
```

Unban an IP address:
```bash
sudo fail2ban-client set apache-php-probe-extended unbanip <IP_ADDRESS>
```

## Notes

- These filters are designed to catch **attack patterns**, not legitimate traffic. Test in your environment before deployment.
- The secret scan module is particularly aggressive (1-week bans) due to high confidence that such probes are malicious.
- All actions use UFW with the "Apache Full" application profile, blocking HTTP and HTTPS ports.
- Adjust `maxretry` and `findtime` values based on your traffic patterns to minimize false positives.

## License

This configuration is provided as-is for security purposes.
