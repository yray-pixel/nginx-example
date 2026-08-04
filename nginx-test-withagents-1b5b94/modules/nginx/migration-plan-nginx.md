---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on RedHat and Debian-based systems. It needs modernization to use FQCN module names, proper YAML syntax, quoted octal modes, modern loop syntax, and Python 3 compatibility in templates. The role also needs proper file modes for security and updated SSL protocols.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Creates directory structure for site configurations
- Configures Nginx with main configuration file and default site
- Sets up custom site configurations based on variables
- Manages the Nginx service (start, restart, reload)
- Configures SSL protocols and HTTP parameters

## File Structure

**Task Files:**
roles/nginx/tasks/main.yml

**Handler Files:**
roles/nginx/handlers/main.yml

**Variable Files:**
roles/nginx/defaults/main.yml
roles/nginx/vars/main.yml

**Meta:**
roles/nginx/meta/main.yml

**Templates:**
roles/nginx/templates/default.conf.j2
roles/nginx/templates/default.j2
roles/nginx/templates/nginx.conf.j2
roles/nginx/templates/site.j2

**Static Files:**
roles/nginx/files/epel.repo

## Module Explanation

The role performs operations in this order:

1. **Install and Configure Dependencies** (`roles/nginx/tasks/main.yml`):
   - Installs SELinux Python module on RedHat systems
   - Copies EPEL repository configuration for RedHat systems
   - Installs Nginx packages based on OS family (RedHat or Debian)
   - Uses legacy module names, with_items loops, and inline key=value syntax

2. **Configure Nginx Directory Structure** (`roles/nginx/tasks/main.yml`):
   - Creates sites-available and sites-enabled directories
   - Uses unquoted octal mode 0755 and short module name

3. **Configure Nginx Core Files** (`roles/nginx/tasks/main.yml`):
   - Copies main nginx.conf configuration from template
   - Copies default configuration files
   - Missing mode parameters for template modules
   - Notifies restart handler for main config changes

4. **Configure Custom Sites** (`roles/nginx/tasks/main.yml`):
   - Creates site configurations from templates based on nginx_sites variable
   - Creates symbolic links to enable sites
   - Uses with_items loops with bare variables
   - Notifies reload handler for site changes

5. **Start Nginx Service** (`roles/nginx/tasks/main.yml`):
   - Ensures Nginx service is started and enabled
   - Uses 'yes' instead of 'true' for boolean values

## Modernization Mapping

| Legacy Pattern | Modern Equivalent | Files Affected | Notes |
|---|---|---|---|
| `yum:` | `ansible.builtin.yum:` | tasks/main.yml | FQCN |
| `copy:` | `ansible.builtin.copy:` | tasks/main.yml | FQCN |
| `apt:` | `ansible.builtin.apt:` | tasks/main.yml | FQCN |
| `file:` | `ansible.builtin.file:` | tasks/main.yml | FQCN |
| `template:` | `ansible.builtin.template:` | tasks/main.yml | FQCN |
| `service:` | `ansible.builtin.service:` | tasks/main.yml, handlers/main.yml | FQCN |
| `with_items: redhat_pkg` | `loop: "{{ redhat_pkg }}"` | tasks/main.yml | Loop modernization with proper variable reference |
| `with_items: ubuntu_pkg` | `loop: "{{ ubuntu_pkg }}"` | tasks/main.yml | Loop modernization with proper variable reference |
| `with_items: nginx_sites` | `loop: "{{ nginx_sites }}"` | tasks/main.yml | Loop modernization with proper variable reference |
| `mode=0755` | `mode: '0755'` | tasks/main.yml | Quoted octal mode |
| Missing `mode:` | Add `mode: '0644'` | tasks/main.yml | Add file permissions for template tasks |
| `yes` | `true` | tasks/main.yml | Boolean modernization |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| `"on"` | `true` | defaults/main.yml | Boolean values in nginx_http_params |
| `"65"` | `65` | defaults/main.yml | Native type for keepalive_timeout |
| `TLSv1 TLSv1.1 TLSv1.2` | `TLSv1.2 TLSv1.3` | templates/nginx.conf.j2 | Update SSL protocols |
| `{{ nginx_log_dir}}` | `{{ nginx_log_dir }}` | templates/nginx.conf.j2, templates/site.j2 | Fix Jinja2 spacing |
| `python-selinux` | `python3-selinux` | vars/main.yml | Python 3 compatibility |
| Inline key=value syntax | YAML dictionary format | tasks/main.yml, handlers/main.yml | Syntax modernization |

## Dependencies

**Collection dependencies** (for requirements.yml):
- ansible.builtin: 2.13.0

**Role dependencies**: None (from meta/main.yml)

**External packages**:
- RedHat: libselinux-python, nginx
- Debian: python-selinux (to be updated to python3-selinux), nginx

**Services managed**: 
- nginx (started, restarted, reloaded)

## Template Modernization

- **nginx.conf.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Fix spacing in variable references: `{{ nginx_log_dir}}` → `{{ nginx_log_dir }}`
  - Update SSL protocols to remove TLSv1 and TLSv1.1 (security improvement)
  - Replace `ansible_os_family` with `ansible_facts['os_family']`
  - Replace `ansible_processor_count` with `ansible_facts['processor_count']`

- **site.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Fix spacing in variable references
  - Replace `k.find('location')` with `'location' in k` for cleaner code
  - Fix spacing in variable references: `{{ nginx_log_dir}}` → `{{ nginx_log_dir }}`

## Argument Specification

```yaml
# meta/argument_specs.yml
---
argument_specs:
  main:
    short_description: Install and configure Nginx web server
    description: Installs and configures Nginx web server with custom site configurations
    author: Benno Joy
    options:
      nginx_max_clients:
        type: int
        default: 512
        description: Maximum number of simultaneous client connections
      nginx_http_params:
        type: dict
        default:
          sendfile: true
          tcp_nopush: true
          tcp_nodelay: true
          keepalive_timeout: 65
        description: HTTP parameters for Nginx configuration
      nginx_log_dir:
        type: str
        default: /var/log/nginx
        description: Directory for Nginx log files
      nginx_access_log_name:
        type: str
        default: access.log
        description: Name of the access log file
      nginx_error_log_name:
        type: str
        default: error.log
        description: Name of the error log file
      nginx_separate_logs_per_site:
        type: bool
        default: false
        description: Whether to create separate log files for each site
      nginx_sites:
        type: list
        elements: dict
        default: []
        description: List of site configurations to create
```

## Checks for the Migration

**Files to verify**:
- roles/nginx/tasks/main.yml
- roles/nginx/handlers/main.yml
- roles/nginx/defaults/main.yml
- roles/nginx/vars/main.yml
- roles/nginx/meta/main.yml
- roles/nginx/meta/argument_specs.yml
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2
- roles/nginx/files/epel.repo

**Services to check**:
- nginx

**Templates to validate**:
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2

## Pre-flight checks:
```bash
# Verify Nginx configuration syntax
nginx -t

# Check Nginx service status
systemctl status nginx

# Verify site configurations
ls -la /etc/nginx/sites-enabled/
ls -la /etc/nginx/sites-available/

# Test a site configuration
curl -I http://localhost:8080
curl -I http://localhost:9090

# Check SSL configuration (if applicable)
nmap --script ssl-enum-ciphers -p 443 localhost
```