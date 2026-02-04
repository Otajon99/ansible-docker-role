
---

# Ansible Role: Install Docker (RHEL & Debian)

This Ansible role installs the **latest version of Docker CE** on both:

- **RHEL-based systems** (RHEL, CentOS, Rocky, AlmaLinux)
- **Debian-based systems** (Debian, Ubuntu)

The role also **enables and starts** the Docker service using **systemd**.

---

## 📁 Role Structure

```
ansible-docker-role/
│
├── defaults/
│   └── main.yml
│
├── tasks/
│   ├── main.yml
│   ├── debian.yml
│   └── rhel.yml
│
├── handlers/
│   └── main.yml
│
├── meta/
│   └── main.yml
│
└── README.md
````


---

## ⚙️ defaults/main.yml

```yaml
---
docker_service_name: docker
````

---

## 🧠 tasks/main.yml

Determines OS type and runs correct task file.

```yaml
---
- name: Include Debian tasks
  include_tasks: debian.yml
  when: ansible_os_family == "Debian"

- name: Include RHEL tasks
  include_tasks: rhel.yml
  when: ansible_os_family == "RedHat"

- name: Ensure Docker service is enabled and started
  ansible.builtin.systemd:
    name: "{{ docker_service_name }}"
    enabled: true
    state: started
```

---

## 🐧 tasks/debian.yml

```yaml
---
- name: Install required packages
  ansible.builtin.apt:
    name:
      - ca-certificates
      - curl
      - gnupg
      - lsb-release
    state: present
    update_cache: true

- name: Add Docker GPG key
  ansible.builtin.apt_key:
    url: https://download.docker.com/linux/{{ ansible_distribution | lower }}/gpg
    state: present

- name: Add Docker repository
  ansible.builtin.apt_repository:
    repo: >
      deb [arch=amd64]
      https://download.docker.com/linux/{{ ansible_distribution | lower }}
      {{ ansible_distribution_release }} stable
    state: present

- name: Install Docker CE
  ansible.builtin.apt:
    name:
      - docker-ce
      - docker-ce-cli
      - containerd.io
    state: latest
    update_cache: true
```

---

## 🟥 tasks/rhel.yml

```yaml
---
- name: Install required packages
  ansible.builtin.yum:
    name:
      - yum-utils
      - device-mapper-persistent-data
      - lvm2
    state: present

- name: Add Docker repository
  ansible.builtin.get_url:
    url: https://download.docker.com/linux/centos/docker-ce.repo
    dest: /etc/yum.repos.d/docker-ce.repo

- name: Install Docker CE
  ansible.builtin.yum:
    name:
      - docker-ce
      - docker-ce-cli
      - containerd.io
    state: latest
```

---

## 🔁 handlers/main.yml

```yaml
---
- name: Restart Docker
  ansible.builtin.systemd:
    name: docker
    state: restarted
```

---

## 📦 meta/main.yml

```yaml
---
galaxy_info:
  role_name: docker_install
  author: your_name
  description: Install Docker CE on Debian and RHEL systems
  min_ansible_version: 2.9
  platforms:
    - name: EL
      versions: [7, 8, 9]
    - name: Debian
      versions: [buster, bullseye]
    - name: Ubuntu
      versions: [focal, jammy]

dependencies: []
```

---

## ▶ Example Playbook

```yaml
---
- name: Install Docker
  hosts: all
  become: true
  roles:
    - docker_install
```

---

## 🧪 How to Run

```bash
ansible-playbook -i inventory.ini playbook.yml
```

---

## ✅ Verification

```bash
docker --version
systemctl status docker
```

---

## 🚀 GitHub Submission Steps

```bash
git init
git add .
git commit -m "Initial Docker install role"
git branch -M main
git remote add origin <your-repo-url>
git push -u origin main
```

