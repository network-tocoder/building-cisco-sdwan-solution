# 📘 Automation — Ansible for Cisco SD-WAN

[← Back to Main](../../README.md) | [Python →](../python/README.md) | [Postman →](../postman/README.md) | [Terraform →](../terraform/README.md)

---

## 📦 Installation

```bash
pip3 install ansible
ansible-galaxy collection install cisco.catalystwan
```

---

## 🏥 Health Check Playbook

```yaml
# playbooks/00-health-check.yml
- name: SD-WAN Health Check
  hosts: localhost
  tasks:
    - name: Get all device status
      cisco.catalystwan.devices_info:
        manager_credentials:
          url: "https://{{ vmanage_host }}:{{ vmanage_port }}"
          username: "{{ vmanage_user }}"
          password: "{{ vmanage_pass }}"
      register: device_info

    - name: Print device reachability
      debug:
        msg: "{{ item['host-name'] }} | {{ item['system-ip'] }} | {{ item['reachability'] }}"
      loop: "{{ device_info.devices }}"

    - name: Fail if any device unreachable
      fail:
        msg: "{{ item['host-name'] }} is unreachable!"
      when: item['reachability'] != 'reachable'
      loop: "{{ device_info.devices }}"
```

---

## 📦 Onboard Device Playbook

```yaml
# playbooks/01-onboard-device.yml
- name: Onboard New WAN Edge
  hosts: localhost
  tasks:
    - name: Add device to vManage
      cisco.catalystwan.device_action:
        manager_credentials:
          url: "https://{{ vmanage_host }}:{{ vmanage_port }}"
          username: "{{ vmanage_user }}"
          password: "{{ vmanage_pass }}"
        action: add
        payload:
          deviceIP: "{{ item.system_ip }}"
          generateCSR: true
          personality: "{{ item.personality }}"
      loop: "{{ new_devices }}"
```

---

## 🚀 Running Playbooks

```bash
# Health check
ansible-playbook -i inventory/hosts.yml playbooks/00-health-check.yml

# Onboard device
ansible-playbook -i inventory/hosts.yml playbooks/01-onboard-device.yml

# Push template
ansible-playbook -i inventory/hosts.yml playbooks/02-push-template.yml

# Deploy policy
ansible-playbook -i inventory/hosts.yml playbooks/03-deploy-policy.yml

# Collect stats
ansible-playbook -i inventory/hosts.yml playbooks/04-collect-stats.yml
```

---

[← Back to Main](../../README.md) | [Python →](../python/README.md)
