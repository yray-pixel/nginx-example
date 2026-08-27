---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on RedHat and Debian-based systems. It needs modernization to use FQCN module names, proper YAML dictionary format, quoted octal modes, boolean values, loop syntax, and Python 3 compatible Jinja2 templates.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Configures Nginx with main configuration file and site-specific configurations
- Creates directory structure for sites-available and sites-enabled
- Sets up default site and custom sites based on variables
- Manages Nginx service (start, restart, reload)

## File Structure

**Task Files:**
- roles/nginx/tasks/main.yml

**Handler Files:**
- roles/nginx/handlers/main.yml

**Variable Files:**
- roles/nginx/defaults/main.yml
- roles/nginx/vars/main.yml

**Meta:**
- roles/nginx/meta/main.yml

**Templates:**
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2

**Static Files:**
- roles/nginx/files/epel.repo

## Module Explanation

The role performs operations in this order:

1. **Install and Configure Nginx** (`roles/nginx/tasks/main.yml`):
   - Installs SELinux Python module on RedHat systems
   - Copies EPEL repository configuration for RedHat systems
   - Installs Nginx packages on RedHat systems using a variable list
   - Installs Nginx packages on Debian systems using a variable list
   - Creates directory structure for site configurations
   - Copies main Nginx configuration file
   - Copies default configuration files
   - Creates links for default site
   - Creates configurations for custom sites based on variables
   - Creates links for custom sites
   - Starts and enables the Nginx service

2. **Handlers** (`roles/nginx/handlers/main.yml`):
   - Restarts Nginx service when configuration changes
   - Reloads Nginx service when site configurations change

## Modernization Mapping

| Legacy Pattern | Modern Equivalent | Files Affected | Notes |
|---|---|---|---|
| `yum:` | `ansible.builtin.yum:` | tasks/main.yml | FQCN |
| `copy:` | `ansible.builtin.copy:` | tasks/main.yml | FQCN |
| `apt:` | `ansible.builtin.apt:` | tasks/main.yml | FQCN |
| `file:` | `ansible.builtin.file:` | tasks/main.yml | FQCN |
| `template:` | `ansible.builtin.template:` | tasks/main.yml | FQCN |
| `service:` | `ansible.builtin.service:` | tasks/main.yml, handlers/main.yml | FQCN |
| `with_items: redhat_pkg` | `loop: "{{ redhat_pkg }}"` | tasks/main.yml | Loop modernization with proper variable syntax |
| `with_items: ubuntu_pkg` | `loop: "{{ ubuntu_pkg }}"` | tasks/main.yml | Loop modernization with proper variable syntax |
| `with_items: nginx_sites` | `loop: "{{ nginx_sites }}"` | tasks/main.yml | Loop modernization with proper variable syntax |
| `mode=0755` | `mode: '0755'` | tasks/main.yml | Quoted octal mode |
| `name=nginx state=restarted` | `name: nginx`<br>`state: restarted` | handlers/main.yml | YAML dictionary format |
| `name=nginx state=reloaded` | `name: nginx`<br>`state: reloaded` | handlers/main.yml | YAML dictionary format |
| `enabled=yes` | `enabled: true` | tasks/main.yml | Boolean value |
| `update_cache=yes` | `update_cache: true` | tasks/main.yml | Boolean value |
| `sendfile: "on"` | `sendfile: true` | defaults/main.yml | Boolean value |
| `tcp_nopush: "on"` | `tcp_nopush: true` | defaults/main.yml | Boolean value |
| `tcp_nodelay: "on"` | `tcp_nodelay: true` | defaults/main.yml | Boolean value |
| `keepalive_timeout: "65"` | `keepalive_timeout: 65` | defaults/main.yml | Native integer type |
| `nginx_sites\|lower != 'none'` | `nginx_sites\|lower != 'none'` | tasks/main.yml | No change needed, but consider using `nginx_sites != none` with jinja2_native: true |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| `k.find('location')` | `'location' in k` | templates/site.j2 | Python 3 compatibility |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| Missing `mode` parameter | Add `mode: '0644'` | All template and copy tasks | File permissions |

## Dependencies

**Collection dependencies** (for requirements.yml):
- ansible.builtin: '*'

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
  - Update SSL protocols to remove TLSv1 and TLSv1.1 (insecure)
  - Parameterize SSL protocols as a variable

- **site.j2**:
  - Replace `iteritems()` with `items()` in two locations
  - Replace `k.find('location') == -1` with `'location' not in k`
  - Replace `k.find('location') != -1` with `'location' in k`
  - Simplify `nginx_separate_logs_per_site == True` to `nginx_separate_logs_per_site`

## Argument Specification

```yaml
# meta/argument_specs.yml
---
argument_specs:
  main:
    short_description: Install and configure Nginx web server
    description: Installs and configures Nginx web server on RedHat and Debian-based systems
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
        description: List of site configurations
```

## Checks for the Migration

**Files to verify**:
- roles/nginx/tasks/main.yml
- roles/nginx/handlers/main.yml
- roles/nginx/defaults/main.yml
- roles/nginx/vars/main.yml
- roles/nginx/meta/main.yml
- roles/nginx/meta/argument_specs.yml (new)
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2
- roles/nginx/files/epel.repo

**Services to check**: nginx

**Templates to validate**:
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2

## Pre-flight checks:
- Verify Nginx configuration: `nginx -t`
- Check Nginx service status: `systemctl status nginx`
- Verify site configurations: `ls -la /etc/nginx/sites-enabled/`
- Test HTTP response: `curl -I http://localhost`
- Check for SSL configuration (if enabled): `curl -I https://localhost`
- Verify log file permissions: `ls -la /var/log/nginx/`
- Check SELinux context on Nginx directories (RedHat systems): `ls -Z /etc/nginx/`