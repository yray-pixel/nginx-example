---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on RedHat and Debian-based systems. It needs modernization to use FQCN module names, proper YAML dictionary syntax, quoted octal modes, modern loop syntax, and Python 3 compatibility in templates.

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
   - Configures Nginx with main configuration file
   - Sets up default site configuration
   - Creates custom site configurations based on variables
   - Ensures the Nginx service is started and enabled

2. **Handlers** (`roles/nginx/handlers/main.yml`):
   - Provides handlers to restart and reload Nginx when configuration changes

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
| `update_cache=yes` | `update_cache: true` | tasks/main.yml | Boolean modernization |
| `enabled=yes` | `enabled: true` | tasks/main.yml | Boolean modernization |
| `name=nginx state=restarted` | `name: nginx`<br>`state: restarted` | handlers/main.yml | YAML dictionary syntax |
| `name=nginx state=reloaded` | `name: nginx`<br>`state: reloaded` | handlers/main.yml | YAML dictionary syntax |
| `name=libselinux-python state=present` | `name: libselinux-python`<br>`state: present` | tasks/main.yml | YAML dictionary syntax |
| `src=epel.repo dest=/etc/yum.repos.d/epel_ansible.repo` | `src: epel.repo`<br>`dest: /etc/yum.repos.d/epel_ansible.repo` | tasks/main.yml | YAML dictionary syntax |
| `name={{ item }} state=present` | `name: "{{ item }}"`<br>`state: present` | tasks/main.yml | YAML dictionary syntax |
| `path=/etc/nginx/{{ item }} state=directory owner=root group=root mode=0755` | `path: "/etc/nginx/{{ item }}"`<br>`state: directory`<br>`owner: root`<br>`group: root`<br>`mode: '0755'` | tasks/main.yml | YAML dictionary syntax |
| `src=nginx.conf.j2 dest=/etc/nginx/nginx.conf` | `src: nginx.conf.j2`<br>`dest: /etc/nginx/nginx.conf` | tasks/main.yml | YAML dictionary syntax |
| `src=default.conf.j2 dest=/etc/nginx/conf.d/default.conf` | `src: default.conf.j2`<br>`dest: /etc/nginx/conf.d/default.conf` | tasks/main.yml | YAML dictionary syntax |
| `src=default.j2 dest=/etc/nginx/sites-available/default` | `src: default.j2`<br>`dest: /etc/nginx/sites-available/default` | tasks/main.yml | YAML dictionary syntax |
| `path=/etc/nginx/sites-enabled/default state=link src=/etc/nginx/sites-available/default` | `path: /etc/nginx/sites-enabled/default`<br>`state: link`<br>`src: /etc/nginx/sites-available/default` | tasks/main.yml | YAML dictionary syntax |
| `src=site.j2 dest=/etc/nginx/sites-available/{{ item['server']['file_name'] }}` | `src: site.j2`<br>`dest: "/etc/nginx/sites-available/{{ item['server']['file_name'] }}"` | tasks/main.yml | YAML dictionary syntax |
| `path=/etc/nginx/sites-enabled/{{ item['server']['file_name'] }} state=link src=/etc/nginx/sites-available/{{ item['server']['file_name'] }}` | `path: "/etc/nginx/sites-enabled/{{ item['server']['file_name'] }}"`<br>`state: link`<br>`src: "/etc/nginx/sites-available/{{ item['server']['file_name'] }}"` | tasks/main.yml | YAML dictionary syntax |
| `name=nginx state=started enabled=yes` | `name: nginx`<br>`state: started`<br>`enabled: true` | tasks/main.yml | YAML dictionary syntax |
| `sendfile: "on"` | `sendfile: true` | defaults/main.yml | Boolean modernization |
| `tcp_nopush: "on"` | `tcp_nopush: true` | defaults/main.yml | Boolean modernization |
| `tcp_nodelay: "on"` | `tcp_nodelay: true` | defaults/main.yml | Boolean modernization |
| `keepalive_timeout: "65"` | `keepalive_timeout: 65` | defaults/main.yml | Native type enforcement |
| `nginx_sites\|lower != 'none'` | `nginx_sites \| lower != 'none'` | tasks/main.yml | Proper Jinja2 spacing |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| `k.find('location') == -1` | `'location' not in k` | templates/site.j2 | Modern Python syntax |
| `k.find('location') != -1` | `'location' in k` | templates/site.j2 | Modern Python syntax |
| Missing `mode` parameter | Add `mode: '0644'` | tasks/main.yml | For template and copy tasks |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Modern fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Modern fact access |
| `python-selinux` | `python3-selinux` | vars/main.yml | Python 3 compatibility |
| SSL protocols include TLSv1 and TLSv1.1 | `ssl_protocols TLSv1.2 TLSv1.3;` | templates/nginx.conf.j2 | Security enhancement |

## Dependencies

**Collection dependencies** (for requirements.yml):
- ansible.builtin: '*'

**Role dependencies**: None (from meta/main.yml)

**External packages**:
- RedHat: libselinux-python, nginx
- Debian: python-selinux (to be updated to python3-selinux), nginx

**Services managed**: nginx

## Template Modernization

- **nginx.conf.j2**:
  - Replace `ansible_os_family` with `ansible_facts['os_family']`
  - Replace `ansible_processor_count` with `ansible_facts['processor_count']`
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Update SSL protocols to only include TLSv1.2 and TLSv1.3 for security

- **site.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Replace `k.find('location') == -1` with `'location' not in k`
  - Replace `k.find('location') != -1` with `'location' in k`

## Argument Specification

```yaml
argument_specs:
  main:
    short_description: Install and configure Nginx web server
    description: This role installs and configures Nginx web server on RedHat and Debian-based systems
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
        description: List of site configurations
        default: [...]
```

## Checks for the Migration

**Files to verify**:
- roles/nginx/tasks/main.yml
- roles/nginx/handlers/main.yml
- roles/nginx/defaults/main.yml
- roles/nginx/vars/main.yml
- roles/nginx/meta/main.yml
- roles/nginx/meta/argument_specs.yml (new file)
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
- Check for SELinux issues: `ausearch -m avc -ts recent`
- Validate SSL configuration: `nmap --script ssl-enum-ciphers -p 443 localhost`