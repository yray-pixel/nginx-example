---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on both RedHat and Debian-based systems. It needs modernization to use FQCN module names, proper YAML dictionary format instead of inline key=value syntax, modern loop syntax, quoted octal modes, boolean values as true/false, and Python 3 compatible Jinja2 templates.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Configures SELinux for RedHat systems
- Creates directory structure for site configurations
- Configures main Nginx configuration file
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

1. **Install and Configure Prerequisites** (`roles/nginx/tasks/main.yml`):
   - Installs SELinux Python module on RedHat systems
   - Copies EPEL repository configuration for RedHat systems
   - Uses legacy module names, fact access, and inline key=value syntax

2. **Install Nginx Packages** (`roles/nginx/tasks/main.yml`):
   - Installs Nginx packages on RedHat systems using `yum` module with `with_items` loop
   - Installs Nginx packages on Debian systems using `apt` module with `with_items` loop
   - Uses legacy module names, fact access, and string booleans

3. **Create Directory Structure** (`roles/nginx/tasks/main.yml`):
   - Creates sites-available and sites-enabled directories
   - Uses unquoted octal mode and legacy module names

4. **Configure Nginx** (`roles/nginx/tasks/main.yml`):
   - Copies main Nginx configuration file using templates
   - Copies default configuration files
   - Missing mode parameters for template modules

5. **Configure Sites** (`roles/nginx/tasks/main.yml`):
   - Creates site configurations from templates using `with_items` loop
   - Creates symbolic links to enable sites
   - Uses bare variables in loops and legacy module names

6. **Start Nginx Service** (`roles/nginx/tasks/main.yml`):
   - Starts and enables the Nginx service
   - Uses string boolean `yes` instead of `true`

## Modernization Mapping

| Legacy Pattern | Modern Equivalent | Files Affected | Notes |
|---|---|---|---|
| `yum:` | `ansible.builtin.yum:` | tasks/main.yml | FQCN |
| `copy:` | `ansible.builtin.copy:` | tasks/main.yml | FQCN |
| `apt:` | `ansible.builtin.apt:` | tasks/main.yml | FQCN |
| `file:` | `ansible.builtin.file:` | tasks/main.yml | FQCN |
| `template:` | `ansible.builtin.template:` | tasks/main.yml | FQCN |
| `service:` | `ansible.builtin.service:` | tasks/main.yml, handlers/main.yml | FQCN |
| `with_items: redhat_pkg` | `loop: "{{ redhat_pkg }}"` | tasks/main.yml | Loop modernization with proper variable quoting |
| `with_items: ubuntu_pkg` | `loop: "{{ ubuntu_pkg }}"` | tasks/main.yml | Loop modernization with proper variable quoting |
| `with_items: nginx_sites` | `loop: "{{ nginx_sites }}"` | tasks/main.yml | Loop modernization with proper variable quoting |
| `mode=0755` | `mode: '0755'` | tasks/main.yml | Quoted octal mode |
| `yes` | `true` | tasks/main.yml | Boolean modernization |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| Inline key=value syntax | YAML dictionary format | tasks/main.yml, handlers/main.yml | Syntax modernization |
| Missing mode parameter | Add `mode: '0644'` | tasks/main.yml | File permissions |
| `"on"` in nginx_http_params | `true` | defaults/main.yml | Boolean modernization |
| `"65"` in nginx_http_params | `65` | defaults/main.yml | Native type enforcement |
| Missing spaces in templates | Add proper spacing | templates/nginx.conf.j2 | Template formatting |
| `python-selinux` package | `python3-selinux` | vars/main.yml | Python 3 compatibility |
| TLSv1 and TLSv1.1 | Remove and use only TLSv1.2 and TLSv1.3 | templates/nginx.conf.j2 | Security enhancement |

## Dependencies

**Collection dependencies** (for requirements.yml):
- ansible.builtin: latest

**Role dependencies**: None (from meta/main.yml)

**External packages**:
- RedHat: libselinux-python, nginx
- Debian: python-selinux (to be updated to python3-selinux), nginx

**Services managed**: 
- nginx (started, restarted, reloaded)

## Template Modernization

- **nginx.conf.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Fix missing spaces in `{{ nginx_log_dir}}` to `{{ nginx_log_dir }}`
  - Fix missing spaces in `{{ nginx_access_log_name}}` to `{{ nginx_access_log_name }}`
  - Remove TLSv1 and TLSv1.1 from ssl_protocols, use only TLSv1.2 and TLSv1.3
  - Replace `ansible_os_family` with `ansible_facts['os_family']`
  - Replace `ansible_processor_count` with `ansible_facts['processor_count']`

- **site.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Fix missing spaces in `{{ nginx_log_dir}}` to `{{ nginx_log_dir }}`
  - Fix missing spaces in `{{ item.server.server_name}}` to `{{ item.server.server_name }}`
  - Replace string comparison using `find` with `in` operator or `startswith`
  - Remove extra space after `{% endfor %}` on line 10

## Argument Specification

**meta/argument_specs.yml**:
- `nginx_max_clients`: integer, default: 512, description: "Maximum number of simultaneous client connections"
- `nginx_http_params`: dictionary, default: see defaults/main.yml, description: "HTTP parameters for Nginx configuration"
- `nginx_log_dir`: string, default: "/var/log/nginx", description: "Directory for Nginx logs"
- `nginx_access_log_name`: string, default: "access.log", description: "Name of access log file"
- `nginx_error_log_name`: string, default: "error.log", description: "Name of error log file"
- `nginx_separate_logs_per_site`: boolean, default: false, description: "Whether to create separate log files per site"
- `nginx_sites`: list, default: see defaults/main.yml, description: "List of site configurations to create"

## Checks for the Migration

**Files to verify**:
- roles/nginx/tasks/main.yml
- roles/nginx/handlers/main.yml
- roles/nginx/defaults/main.yml
- roles/nginx/vars/main.yml
- roles/nginx/meta/main.yml
- roles/nginx/meta/argument_specs.yml (new)
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2
- roles/nginx/files/epel.repo

**Services to check**:
- nginx

**Templates to validate**:
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2

## Pre-flight checks:
- Verify Nginx configuration: `nginx -t`
- Check Nginx service status: `systemctl status nginx`
- Validate site configurations: `ls -la /etc/nginx/sites-enabled/`
- Verify Nginx is listening on configured ports: `ss -tulpn | grep nginx`
- Check for syntax errors in generated configuration files: `find /etc/nginx -type f -name "*.conf" -exec nginx -c {} -t \;`