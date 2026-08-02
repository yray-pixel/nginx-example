---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on RedHat and Debian-based systems. It needs modernization to use FQCN module names, proper boolean values, quoted octal modes, loop syntax updates, and Python 3 compatibility in templates.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Creates directory structure for site configurations
- Configures Nginx with main configuration file and default site
- Sets up custom site configurations based on variables
- Manages the Nginx service (start, restart, reload)

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
   - Configures Nginx with main configuration file and default site
   - Creates and enables custom site configurations
   - Starts the Nginx service

2. **Handlers** (`roles/nginx/handlers/main.yml`):
   - Provides handlers to restart or reload Nginx when configuration changes

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
| `yes` | `true` | tasks/main.yml | Boolean modernization |
| `"on"` | `true` | defaults/main.yml | Boolean modernization in nginx_http_params |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| Missing mode parameter | Add `mode: '0644'` | All template and copy tasks | File permissions |
| Inline key=value syntax | YAML dictionary format | tasks/main.yml, handlers/main.yml | Syntax modernization |
| `python-selinux` | `python3-selinux` | vars/main.yml | Python 3 compatibility |

## Dependencies

**Collection dependencies** (for requirements.yml):
- ansible.builtin: 2.9.0

**Role dependencies**: None (from meta/main.yml)

**External packages**:
- RedHat: libselinux-python, nginx
- Debian: python-selinux (to be updated to python3-selinux), nginx

**Services managed**: nginx

## Template Modernization

- **nginx.conf.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Fix missing spaces in `{{ nginx_log_dir}}` and `{{ nginx_access_log_name}}` to `{{ nginx_log_dir }}` and `{{ nginx_access_log_name }}`
  - Update SSL protocols to remove TLSv1 and TLSv1.1 (insecure) and use only TLSv1.2 and TLSv1.3
  - Replace `ansible_os_family` with `ansible_facts['os_family']`
  - Replace `ansible_processor_count` with `ansible_facts['processor_count']`

- **site.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Fix missing spaces in `{{ nginx_log_dir}}` to `{{ nginx_log_dir }}`
  - Improve readability of conditional expressions in for loops
  - Replace string method `find()` with `in` operator or `startswith()` for better readability

## Argument Specification

Variables that should be in meta/argument_specs.yml:
- `nginx_max_clients`: integer, default: 512, description: "Maximum number of simultaneous client connections"
- `nginx_http_params`: dictionary, default: {'sendfile': true, 'tcp_nopush': true, 'tcp_nodelay': true, 'keepalive_timeout': 65}, description: "HTTP parameters for nginx configuration"
- `nginx_log_dir`: string, default: "/var/log/nginx", description: "Directory for nginx log files"
- `nginx_access_log_name`: string, default: "access.log", description: "Name of the access log file"
- `nginx_error_log_name`: string, default: "error.log", description: "Name of the error log file"
- `nginx_separate_logs_per_site`: boolean, default: false, description: "Whether to create separate log files for each site"
- `nginx_sites`: list, default: [site configurations], description: "List of site configurations to create"

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

**Services to check**: nginx

**Templates to validate**:
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2

## Pre-flight checks:
- Verify nginx configuration: `nginx -t`
- Check nginx service status: `systemctl status nginx`
- Verify site configurations: `ls -la /etc/nginx/sites-enabled/`
- Test HTTP response: `curl -I http://localhost`
- Check for syntax errors in configuration files: `find /etc/nginx -type f -name "*.conf" -exec nginx -c {} -t \;`
- Verify log file permissions: `ls -la /var/log/nginx/`