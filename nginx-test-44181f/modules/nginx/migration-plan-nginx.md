---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on RedHat and Debian-based systems. It needs modernization to use FQCN module names, proper YAML syntax, quoted octal modes, modern loop syntax, and Python 3 compatible templates. The role also requires updates to SSL protocols and proper spacing in Jinja2 templates.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Configures Nginx with main configuration file and site-specific configurations
- Creates directory structure for sites-available and sites-enabled
- Sets up default site configuration
- Creates and enables custom site configurations based on variables
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

1. **Install and Configure Dependencies** (`roles/nginx/tasks/main.yml`):
   - Installs SELinux Python module on RedHat systems
   - Copies EPEL repository configuration for RedHat systems
   - Legacy patterns: short module names, legacy fact access, inline key=value syntax
   - Modern equivalent: FQCN module names, structured YAML syntax, proper fact access

2. **Install Nginx Packages** (`roles/nginx/tasks/main.yml`):
   - Installs Nginx packages on RedHat systems using `yum` module with `with_items` loop
   - Installs Nginx packages on Debian systems using `apt` module with `with_items` loop and environment variables
   - Legacy patterns: short module names, `with_items` loops, bare variables, `yes` boolean
   - Modern equivalent: FQCN module names, `loop` directive, proper variable references, `true` boolean

3. **Create Directory Structure** (`roles/nginx/tasks/main.yml`):
   - Creates sites-available and sites-enabled directories
   - Legacy patterns: short module name, `with_items` loop, unquoted octal mode
   - Modern equivalent: FQCN module name, `loop` directive, quoted octal mode

4. **Configure Nginx** (`roles/nginx/tasks/main.yml`):
   - Copies main Nginx configuration file (nginx.conf.j2)
   - Copies default configuration file (default.conf.j2)
   - Copies default site configuration (default.j2)
   - Creates symlink for default site
   - Legacy patterns: short module names, missing mode parameters
   - Modern equivalent: FQCN module names, add mode parameters

5. **Configure Custom Sites** (`roles/nginx/tasks/main.yml`):
   - Creates site-specific configurations based on `nginx_sites` variable
   - Creates symlinks to enable the sites
   - Legacy patterns: short module names, `with_items` loops, bare variables, missing mode parameters
   - Modern equivalent: FQCN module names, `loop` directive, proper variable references, add mode parameters

6. **Start Nginx Service** (`roles/nginx/tasks/main.yml`):
   - Starts and enables the Nginx service
   - Legacy patterns: short module name, `yes` boolean
   - Modern equivalent: FQCN module name, `true` boolean

## Modernization Mapping

| Legacy Pattern | Modern Equivalent | Files Affected | Notes |
|---|---|---|---|
| `yum:` | `ansible.builtin.yum:` | tasks/main.yml | FQCN |
| `copy:` | `ansible.builtin.copy:` | tasks/main.yml | FQCN |
| `apt:` | `ansible.builtin.apt:` | tasks/main.yml | FQCN |
| `file:` | `ansible.builtin.file:` | tasks/main.yml | FQCN |
| `template:` | `ansible.builtin.template:` | tasks/main.yml | FQCN |
| `service:` | `ansible.builtin.service:` | tasks/main.yml, handlers/main.yml | FQCN |
| `with_items:` | `loop:` | tasks/main.yml | Loop modernization |
| `with_items: redhat_pkg` | `loop: "{{ redhat_pkg }}"` | tasks/main.yml | Bare variable in loop |
| `with_items: ubuntu_pkg` | `loop: "{{ ubuntu_pkg }}"` | tasks/main.yml | Bare variable in loop |
| `with_items: nginx_sites` | `loop: "{{ nginx_sites }}"` | tasks/main.yml | Bare variable in loop |
| `mode=0755` | `mode: '0755'` | tasks/main.yml | Quoted octal mode |
| `yes` | `true` | tasks/main.yml | Boolean modernization |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| Inline key=value syntax | YAML dictionary format | tasks/main.yml, handlers/main.yml | Syntax modernization |
| Missing mode parameter | Add `mode: '0644'` | tasks/main.yml (template tasks) | File permissions |
| Missing spaces in Jinja2 | Add proper spacing | templates/nginx.conf.j2 | Template formatting |
| TLSv1, TLSv1.1 | TLSv1.2, TLSv1.3 | templates/nginx.conf.j2 | Security improvement |
| `"on"` in nginx_http_params | `true` | defaults/main.yml | Boolean modernization |
| `"65"` in nginx_http_params | `65` | defaults/main.yml | Native type enforcement |

## Dependencies

**Collection dependencies** (for requirements.yml):
- ansible.builtin: latest

**Role dependencies**: None (from meta/main.yml)

**External packages**:
- RedHat: libselinux-python, nginx
- Debian: python-selinux, nginx

**Services managed**: 
- nginx (start, restart, reload)

## Template Modernization

- **nginx.conf.j2**: 
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Fix missing spaces in `{{ nginx_log_dir}}` to `{{ nginx_log_dir }}`
  - Fix missing spaces in `{{ nginx_access_log_name}}` to `{{ nginx_access_log_name }}`
  - Fix missing spaces in `{{ nginx_error_log_name}}` to `{{ nginx_error_log_name }}`
  - Update SSL protocols to use only TLSv1.2 and TLSv1.3
  - Replace `ansible_os_family` with `ansible_facts['os_family']`
  - Replace `ansible_processor_count` with `ansible_facts['processor_count']`

- **site.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Fix missing spaces in `{{ nginx_log_dir}}` to `{{ nginx_log_dir }}`
  - Fix missing spaces in `{{ item.server.server_name}}` to `{{ item.server.server_name }}`
  - Fix missing spaces in `{{ nginx_access_log_name}}` to `{{ nginx_access_log_name }}`
  - Fix missing spaces in `{{ nginx_error_log_name}}` to `{{ nginx_error_log_name }}`
  - Consider refactoring the conditional expression in the for loop for better readability

## Argument Specification

Variables that should be in meta/argument_specs.yml:
- `nginx_max_clients`: integer, default: 512, description: "Maximum number of simultaneous client connections"
- `nginx_http_params`: dict, default: see defaults/main.yml, description: "HTTP parameters for nginx configuration"
- `nginx_log_dir`: string, default: "/var/log/nginx", description: "Directory for nginx logs"
- `nginx_access_log_name`: string, default: "access.log", description: "Name of the access log file"
- `nginx_error_log_name`: string, default: "error.log", description: "Name of the error log file"
- `nginx_separate_logs_per_site`: boolean, default: false, description: "Whether to create separate log files for each site"
- `nginx_sites`: list, default: see defaults/main.yml, description: "List of site configurations to create"

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

**Services to check**:
- nginx

**Templates to validate**:
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2

## Pre-flight checks:
- Verify Nginx configuration: `nginx -t`
- Check Nginx service status: `systemctl status nginx`
- Verify site configurations: `ls -la /etc/nginx/sites-enabled/`
- Test HTTP response: `curl -I http://localhost`
- Test HTTPS response (if configured): `curl -I https://localhost`
- Check for SELinux issues (on RedHat): `ausearch -m avc -ts recent`