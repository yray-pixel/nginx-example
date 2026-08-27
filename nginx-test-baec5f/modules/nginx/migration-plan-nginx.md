---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on RedHat and Debian-based systems. It needs modernization for FQCN module names, loop syntax, boolean values, file permissions, and Python 3 compatibility in templates. The role also requires updates to handle deprecated Jinja2 methods and improve security configurations.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Configures SELinux support on RedHat systems
- Creates directory structure for site configurations
- Configures main Nginx settings via templates
- Sets up default site and custom virtual hosts
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
   - Legacy patterns: short module names, legacy fact access, key=value syntax
   - Modern equivalent: FQCN module names, structured facts, YAML dictionary format

2. **Install Nginx Packages** (`roles/nginx/tasks/main.yml`):
   - Installs Nginx packages on RedHat systems using `yum`
   - Installs Nginx packages on Debian systems using `apt`
   - Legacy patterns: `with_items`, bare variables, short module names, `yes` boolean
   - Modern equivalent: `loop`, proper variable references, FQCN module names, `true` boolean

3. **Configure Nginx Directory Structure** (`roles/nginx/tasks/main.yml`):
   - Creates sites-available and sites-enabled directories
   - Legacy patterns: unquoted octal mode, short module name
   - Modern equivalent: quoted octal mode, FQCN module name

4. **Configure Nginx** (`roles/nginx/tasks/main.yml`):
   - Copies main Nginx configuration file
   - Copies default configuration files
   - Legacy patterns: missing mode parameters, short module names
   - Modern equivalent: add mode parameters, FQCN module names

5. **Configure Virtual Hosts** (`roles/nginx/tasks/main.yml`):
   - Creates site configurations from templates
   - Creates symbolic links to enable sites
   - Legacy patterns: `with_items`, bare variables, missing mode parameters
   - Modern equivalent: `loop`, proper variable references, add mode parameters

6. **Start Nginx Service** (`roles/nginx/tasks/main.yml`):
   - Starts and enables the Nginx service
   - Legacy patterns: `yes` boolean, short module name
   - Modern equivalent: `true` boolean, FQCN module name

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
| `yes` | `true` | tasks/main.yml | Boolean modernization |
| `mode=0755` | `mode: '0755'` | tasks/main.yml | Quoted octal mode |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| Missing mode parameters | Add `mode: '0644'` | tasks/main.yml | File permissions |
| `key=value` syntax | YAML dictionary format | tasks/main.yml, handlers/main.yml | Syntax modernization |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| `"on"` boolean strings | `true` | defaults/main.yml | Native boolean types |
| `"65"` string number | `65` | defaults/main.yml | Native number type |
| `k.find('location') == -1` | `'location' not in k` | templates/site.j2 | Modern Python syntax |
| Missing spaces in templates | Add spaces in `{{ var }}` | templates/nginx.conf.j2 | Template formatting |
| TLSv1, TLSv1.1 | Remove, use only TLSv1.2, TLSv1.3 | templates/nginx.conf.j2 | Security improvement |
| `python-selinux` | `python3-selinux` | vars/main.yml | Python 3 compatibility |

## Dependencies

**Collection dependencies** (for requirements.yml):
- ansible.builtin: latest

**Role dependencies**: None (from meta/main.yml)

**External packages**:
- RedHat: libselinux-python, nginx
- Debian: python-selinux (to be updated to python3-selinux), nginx

**Services managed**: 
- nginx (start, restart, reload)

## Template Modernization

- **nginx.conf.j2**: 
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Add spaces in `{{ nginx_log_dir}}` to `{{ nginx_log_dir }}`
  - Add spaces in `{{ nginx_access_log_name}}` to `{{ nginx_access_log_name }}`
  - Update SSL protocols to remove TLSv1 and TLSv1.1, use only TLSv1.2 and TLSv1.3

- **site.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Replace `k.find('location') == -1` with `'location' not in k`
  - Add spaces in `{{ nginx_log_dir}}` to `{{ nginx_log_dir }}`
  - Add spaces in `{{ item.server.server_name}}-{{ nginx_access_log_name}}` to `{{ item.server.server_name }}-{{ nginx_access_log_name }}`

## Argument Specification

Variables that should be in meta/argument_specs.yml:
- `nginx_max_clients`: integer, default: 512, description: "Maximum number of simultaneous client connections"
- `nginx_http_params`: dict, default: see defaults/main.yml, description: "HTTP parameters for nginx configuration"
- `nginx_log_dir`: string, default: "/var/log/nginx", description: "Directory for nginx log files"
- `nginx_access_log_name`: string, default: "access.log", description: "Name of the access log file"
- `nginx_error_log_name`: string, default: "error.log", description: "Name of the error log file"
- `nginx_separate_logs_per_site`: boolean, default: false, description: "Whether to create separate log files for each site"
- `nginx_sites`: list, default: see defaults/main.yml, description: "List of server blocks to configure"

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
- Verify site configurations: `ls -la /etc/nginx/sites-enabled/`
- Test HTTP connectivity: `curl -I http://localhost`
- Test HTTPS connectivity (if configured): `curl -I https://localhost`
- Check for SELinux issues (on RedHat): `ausearch -m avc -ts recent`