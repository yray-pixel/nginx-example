---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on RedHat and Debian-based systems. It needs modernization to use FQCN module names, proper YAML syntax, quoted octal modes, modern loop syntax, and Python 3 compatibility in templates. The role also requires updates to SSL protocols and package dependencies.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Configures SELinux support on RedHat systems
- Creates directory structure for site configurations
- Configures main Nginx configuration file
- Sets up default site configuration
- Creates and enables custom site configurations based on variables
- Manages the Nginx service (start, restart, reload)

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

1. **Install and Configure Prerequisites** (`roles/nginx/tasks/main.yml`):
   - Installs SELinux Python module on RedHat systems
   - Copies EPEL repository configuration for RedHat systems
   - Uses legacy module names, fact access, and inline key=value syntax

2. **Install Nginx Packages** (`roles/nginx/tasks/main.yml`):
   - Installs Nginx packages on RedHat systems using `yum` module with `with_items` loop
   - Installs Nginx packages on Debian systems using `apt` module with `with_items` loop
   - Uses legacy module names, fact access, and inline key=value syntax

3. **Configure Nginx Directory Structure** (`roles/nginx/tasks/main.yml`):
   - Creates sites-available and sites-enabled directories
   - Uses legacy module names, unquoted octal mode, and inline key=value syntax

4. **Configure Nginx** (`roles/nginx/tasks/main.yml`):
   - Copies main Nginx configuration file using templates
   - Copies default configuration files
   - Creates site-specific configurations based on variables
   - Creates symbolic links to enable sites
   - Uses legacy module names, missing mode parameters, and inline key=value syntax

5. **Start Nginx Service** (`roles/nginx/tasks/main.yml`):
   - Starts and enables the Nginx service
   - Uses legacy module name, `yes` instead of `true`, and inline key=value syntax

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
| `yes` | `true` | tasks/main.yml | Boolean modernization |
| `name=libselinux-python state=present` | `name: libselinux-python`<br>`state: present` | tasks/main.yml | YAML dictionary format |
| `src=epel.repo dest=/etc/yum.repos.d/epel_ansible.repo` | `src: epel.repo`<br>`dest: /etc/yum.repos.d/epel_ansible.repo` | tasks/main.yml | YAML dictionary format |
| `name={{ item }} state=present` | `name: "{{ item }}"`<br>`state: present` | tasks/main.yml | YAML dictionary format |
| `name={{ item }} state=present update_cache=yes` | `name: "{{ item }}"`<br>`state: present`<br>`update_cache: true` | tasks/main.yml | YAML dictionary format |
| `path=/etc/nginx/{{ item }} state=directory owner=root group=root mode=0755` | `path: "/etc/nginx/{{ item }}"`<br>`state: directory`<br>`owner: root`<br>`group: root`<br>`mode: '0755'` | tasks/main.yml | YAML dictionary format |
| `src=nginx.conf.j2 dest=/etc/nginx/nginx.conf` | `src: nginx.conf.j2`<br>`dest: /etc/nginx/nginx.conf`<br>`mode: '0644'` | tasks/main.yml | YAML dictionary format with mode |
| `path=/etc/nginx/sites-enabled/default state=link src=/etc/nginx/sites-available/default` | `path: /etc/nginx/sites-enabled/default`<br>`state: link`<br>`src: /etc/nginx/sites-available/default` | tasks/main.yml | YAML dictionary format |
| `name=nginx state=started enabled=yes` | `name: nginx`<br>`state: started`<br>`enabled: true` | tasks/main.yml | YAML dictionary format |
| `name=nginx state=restarted` | `name: nginx`<br>`state: restarted` | handlers/main.yml | YAML dictionary format |
| `name=nginx state=reloaded` | `name: nginx`<br>`state: reloaded` | handlers/main.yml | YAML dictionary format |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| `nginx_http_params.iteritems()` | `nginx_http_params.items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| `sendfile: "on"` | `sendfile: true` | defaults/main.yml | Boolean modernization |
| `tcp_nopush: "on"` | `tcp_nopush: true` | defaults/main.yml | Boolean modernization |
| `tcp_nodelay: "on"` | `tcp_nodelay: true` | defaults/main.yml | Boolean modernization |
| `keepalive_timeout: "65"` | `keepalive_timeout: 65` | defaults/main.yml | Native type enforcement |
| `nginx_separate_logs_per_site == True` | `nginx_separate_logs_per_site` | templates/site.j2 | Boolean comparison simplification |
| `ssl_protocols TLSv1 TLSv1.1 TLSv1.2` | `ssl_protocols TLSv1.2 TLSv1.3` | templates/nginx.conf.j2 | Security update |
| `python-selinux` | `python3-selinux` | vars/main.yml | Python 3 compatibility |

## Dependencies

**Collection dependencies** (for requirements.yml):
- ansible.builtin: latest

**Role dependencies**: None (from meta/main.yml)

**External packages**:
- RedHat: libselinux-python, nginx
- Debian: python-selinux (to be updated to python3-selinux), nginx

**Services managed**: nginx

## Template Modernization

- **nginx.conf.j2**:
  - Replace `ansible_os_family` with `ansible_facts['os_family']`
  - Replace `ansible_processor_count` with `ansible_facts['processor_count']`
  - Replace `nginx_http_params.iteritems()` with `nginx_http_params.items()`
  - Update SSL protocols to remove TLSv1 and TLSv1.1, use only TLSv1.2 and TLSv1.3
  - Fix missing whitespace in `{{ nginx_log_dir}}/{{ nginx_access_log_name}}` to `{{ nginx_log_dir }}/{{ nginx_access_log_name }}`

- **site.j2**:
  - Replace `nginx_separate_logs_per_site == True` with just `nginx_separate_logs_per_site`
  - Replace `item.server.iteritems()` with `item.server.items()`
  - Replace `v.iteritems()` with `v.items()`
  - Fix missing whitespace in `{{ nginx_log_dir}}/{{ item.server.server_name}}-{{ nginx_access_log_name}}` to `{{ nginx_log_dir }}/{{ item.server.server_name }}-{{ nginx_access_log_name }}`
  - Consider replacing string find method with more readable alternatives:
    - `k.find('location') == -1` to `'location' not in k`
    - `k.find('location') != -1` to `'location' in k`

## Argument Specification

Variables for meta/argument_specs.yml:
- `nginx_max_clients`: integer, default: 512, description: "Maximum number of simultaneous client connections"
- `nginx_http_params`: dictionary, default: see defaults/main.yml, description: "HTTP parameters for nginx configuration"
- `nginx_log_dir`: string, default: "/var/log/nginx", description: "Directory for nginx log files"
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
- Validate site configurations: `ls -la /etc/nginx/sites-enabled/`
- Check for syntax errors in configuration: `grep -r "[error]" /var/log/nginx/error.log`
- Verify listening ports: `ss -tulpn | grep nginx`
- Check SELinux context for Nginx files (on RedHat systems): `ls -Z /etc/nginx/`