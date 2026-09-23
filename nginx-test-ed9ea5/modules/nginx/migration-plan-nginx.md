---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on RedHat and Debian-based systems. It needs modernization for FQCN module names, loop syntax, boolean values, file permissions, and Python 3 compatibility in templates. The role also requires updates to SSL protocols and proper variable spacing in templates.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Creates directory structure for site configurations
- Configures Nginx with main configuration file and default site
- Sets up custom site configurations based on variables
- Manages Nginx service (start, restart, reload)
- Configures SELinux compatibility on RedHat systems

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
| `with_items: redhat_pkg` | `loop: "{{ redhat_pkg }}"` | tasks/main.yml | Loop modernization with proper variable quoting |
| `with_items: ubuntu_pkg` | `loop: "{{ ubuntu_pkg }}"` | tasks/main.yml | Loop modernization with proper variable quoting |
| `with_items: nginx_sites` | `loop: "{{ nginx_sites }}"` | tasks/main.yml | Loop modernization with proper variable quoting |
| `mode=0755` | `mode: '0755'` | tasks/main.yml | Quoted octal mode |
| `name=libselinux-python state=present` | `name: libselinux-python`<br>`state: present` | tasks/main.yml | YAML dictionary format |
| `src=epel.repo dest=/etc/yum.repos.d/epel_ansible.repo` | `src: epel.repo`<br>`dest: /etc/yum.repos.d/epel_ansible.repo`<br>`mode: '0644'` | tasks/main.yml | YAML dictionary format with mode |
| `name={{ item }} state=present` | `name: "{{ item }}"`<br>`state: present` | tasks/main.yml | YAML dictionary format |
| `update_cache=yes` | `update_cache: true` | tasks/main.yml | Boolean true instead of yes |
| `enabled=yes` | `enabled: true` | tasks/main.yml | Boolean true instead of yes |
| `ansible_os_family == "RedHat"` | `ansible_facts['os_family'] == "RedHat"` | tasks/main.yml, templates/nginx.conf.j2 | Modern fact access |
| `ansible_os_family == "Debian"` | `ansible_facts['os_family'] == "Debian"` | tasks/main.yml, templates/nginx.conf.j2 | Modern fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Modern fact access |
| `nginx_http_params.iteritems()` | `nginx_http_params.items()` | templates/nginx.conf.j2 | Python 3 compatibility |
| `item.server.iteritems()` | `item.server.items()` | templates/site.j2 | Python 3 compatibility |
| `v.iteritems()` | `v.items()` | templates/site.j2 | Python 3 compatibility |
| Missing `mode` parameter | Add `mode: '0644'` | All template tasks | File permissions |
| `"on"` in nginx_http_params | `true` | defaults/main.yml | Boolean true instead of "on" |
| `"65"` in nginx_http_params | `65` | defaults/main.yml | Integer instead of string |
| `nginx_sites\|lower != 'none'` | `nginx_sites is defined and nginx_sites != 'none'` | tasks/main.yml | Better conditional check |
| `{{ nginx_log_dir}}` | `{{ nginx_log_dir }}` | templates/nginx.conf.j2, templates/site.j2 | Proper spacing in Jinja2 variables |
| `{{ nginx_access_log_name}}` | `{{ nginx_access_log_name }}` | templates/nginx.conf.j2, templates/site.j2 | Proper spacing in Jinja2 variables |
| `nginx_separate_logs_per_site == True` | `nginx_separate_logs_per_site` | templates/site.j2 | Simplified boolean comparison |
| `TLSv1 TLSv1.1 TLSv1.2` | `TLSv1.2 TLSv1.3` | templates/nginx.conf.j2 | Remove insecure SSL protocols |
| `python-selinux` | `python3-selinux` | vars/main.yml | Python 3 compatibility |

## Dependencies

**Collection dependencies** (for requirements.yml):
- ansible.builtin: latest

**Role dependencies**: None (from meta/main.yml)

**External packages**:
- RedHat: libselinux-python, nginx
- Debian: python-selinux (to be updated to python3-selinux), nginx

**Services managed**: nginx (start, restart, reload)

## Template Modernization

- **nginx.conf.j2**:
  - Replace `ansible_os_family` with `ansible_facts['os_family']`
  - Replace `ansible_processor_count` with `ansible_facts['processor_count']`
  - Add proper spacing in `{{ nginx_log_dir }}` and `{{ nginx_access_log_name }}`
  - Replace `nginx_http_params.iteritems()` with `nginx_http_params.items()`
  - Update SSL protocols to use only `TLSv1.2 TLSv1.3` (remove TLSv1 and TLSv1.1)

- **site.j2**:
  - Add proper spacing in `{{ nginx_log_dir }}` and other variables
  - Replace `item.server.iteritems()` with `item.server.items()`
  - Replace `v.iteritems()` with `v.items()`
  - Replace `nginx_separate_logs_per_site == True` with `nginx_separate_logs_per_site`

- **default.conf.j2** and **default.j2**:
  - Ensure proper spacing in Jinja2 variables

## Argument Specification

**meta/argument_specs.yml**:
```yaml
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
        description: List of site configurations to create
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
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2

## Pre-flight checks:
- Verify Nginx configuration: `nginx -t`
- Check Nginx service status: `systemctl status nginx`
- Verify site configurations: `curl -I http://localhost:8080` and `curl -I http://localhost:9090`
- Check SSL configuration: `nmap --script ssl-enum-ciphers -p 443 localhost`
- Verify SELinux context for Nginx files: `ls -Z /etc/nginx/`
- Check log file permissions: `ls -la /var/log/nginx/`