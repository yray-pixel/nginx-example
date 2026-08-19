---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on RedHat and Debian-based systems. It needs modernization for FQCN module names, loop syntax, boolean values, file permissions, Python 3 compatibility in templates, and proper YAML dictionary format instead of inline key=value syntax.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Configures Nginx with main configuration file and site-specific configurations
- Creates directory structure for sites-available and sites-enabled
- Sets up default site configuration
- Enables and starts the Nginx service
- Supports multiple virtual hosts through the nginx_sites variable

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

1. **Install and Configure Nginx** (`roles/nginx/tasks/main.yml`):
   - Installs SELinux Python module on RedHat systems
   - Copies EPEL repository configuration for RedHat systems
   - Installs Nginx packages based on OS family (RedHat or Debian)
   - Creates directory structure for site configurations
   - Configures main Nginx configuration files
   - Sets up site-specific configurations based on nginx_sites variable
   - Enables and starts the Nginx service

## Modernization Mapping

| Legacy Pattern | Modern Equivalent | Files Affected | Notes |
|---|---|---|---|
| `yum:` | `ansible.builtin.yum:` | tasks/main.yml | FQCN |
| `copy:` | `ansible.builtin.copy:` | tasks/main.yml | FQCN |
| `apt:` | `ansible.builtin.apt:` | tasks/main.yml | FQCN |
| `file:` | `ansible.builtin.file:` | tasks/main.yml | FQCN |
| `template:` | `ansible.builtin.template:` | tasks/main.yml | FQCN |
| `service:` | `ansible.builtin.service:` | tasks/main.yml, handlers/main.yml | FQCN |
| `with_items: redhat_pkg` | `loop: "{{ redhat_pkg }}"` | tasks/main.yml | Loop modernization |
| `with_items: ubuntu_pkg` | `loop: "{{ ubuntu_pkg }}"` | tasks/main.yml | Loop modernization |
| `with_items: nginx_sites` | `loop: "{{ nginx_sites }}"` | tasks/main.yml | Loop modernization |
| `mode=0755` | `mode: '0755'` | tasks/main.yml | Quoted octal mode |
| `update_cache=yes` | `update_cache: true` | tasks/main.yml | Boolean modernization |
| `enabled=yes` | `enabled: true` | tasks/main.yml | Boolean modernization |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| `nginx_http_params.iteritems()` | `nginx_http_params.items()` | templates/nginx.conf.j2 | Python 3 compatibility |
| `item.server.iteritems()` | `item.server.items()` | templates/site.j2 | Python 3 compatibility |
| `v.iteritems()` | `v.items()` | templates/site.j2 | Python 3 compatibility |
| Inline key=value syntax | YAML dictionary format | tasks/main.yml, handlers/main.yml | Syntax modernization |
| Missing mode parameter | Add `mode: '0644'` | template and copy tasks | File permissions |
| `sendfile: "on"` | `sendfile: true` | defaults/main.yml | Boolean modernization |
| `tcp_nopush: "on"` | `tcp_nopush: true` | defaults/main.yml | Boolean modernization |
| `tcp_nodelay: "on"` | `tcp_nodelay: true` | defaults/main.yml | Boolean modernization |
| `keepalive_timeout: "65"` | `keepalive_timeout: 65` | defaults/main.yml | Native type enforcement |
| `TLSv1 TLSv1.1 TLSv1.2` | `TLSv1.2 TLSv1.3` | templates/nginx.conf.j2 | Security modernization |

## Dependencies

**Collection dependencies** (for requirements.yml):
- ansible.builtin: 2.9.0

**Role dependencies**: None (from meta/main.yml)

**External packages**:
- RedHat: libselinux-python, nginx
- Debian: python-selinux, nginx

**Services managed**: nginx

## Template Modernization

- **nginx.conf.j2**:
  - Replace `ansible_os_family` with `ansible_facts['os_family']`
  - Replace `ansible_processor_count` with `ansible_facts['processor_count']`
  - Replace `nginx_http_params.iteritems()` with `nginx_http_params.items()`
  - Update SSL protocols to use only TLSv1.2 and TLSv1.3 for better security
  - Parameterize SSL protocols as a variable for better flexibility

- **site.j2**:
  - Replace `item.server.iteritems()` with `item.server.items()`
  - Replace `v.iteritems()` with `v.items()`
  - Replace string comparison method `k.find('location') == -1` with `'location' not in k`
  - Replace string comparison method `k.find('location') != -1` with `'location' in k`

## Argument Specification

```yaml
# meta/argument_specs.yml
---
argument_specs:
  main:
    short_description: Install and configure Nginx web server
    description: Install and configure Nginx web server with virtual hosts support
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
        description: List of server blocks to configure
```

## Checks for the Migration

**Files to verify**:
- roles/nginx/tasks/main.yml
- roles/nginx/handlers/main.yml
- roles/nginx/defaults/main.yml
- roles/nginx/vars/main.yml
- roles/nginx/meta/main.yml
- roles/nginx/meta/argument_specs.yml
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2
- roles/nginx/files/epel.repo

**Services to check**: nginx

**Templates to validate**:
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2

## Pre-flight checks:
- Verify Nginx configuration: `nginx -t`
- Check Nginx service status: `systemctl status nginx`
- Verify site configurations: `ls -la /etc/nginx/sites-enabled/`
- Test HTTP connectivity: `curl -I http://localhost`
- Check for Python 3 compatibility: Replace python-selinux with python3-selinux on Debian systems
- Verify SSL configuration: `nmap --script ssl-enum-ciphers -p 443 localhost`