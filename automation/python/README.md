# 🐍 Automation — Python (Requests + Netmiko)

[← Back to Main](../../README.md) | [Ansible →](../ansible/README.md) | [Postman →](../postman/README.md) | [Terraform →](../terraform/README.md)

---

## 📦 Requirements

```bash
pip3 install -r requirements.txt
```

---

## 🔑 auth.py — vManage Authentication

```python
import requests, urllib3, os
from dotenv import load_dotenv

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
load_dotenv()

class vManageSession:
    def __init__(self):
        self.host     = os.getenv("VMANAGE_HOST", "192.168.100.10")
        self.port     = os.getenv("VMANAGE_PORT", "8443")
        self.username = os.getenv("VMANAGE_USER", "admin")
        self.password = os.getenv("VMANAGE_PASS", "admin")
        self.base_url = f"https://{self.host}:{self.port}"
        self.session  = requests.Session()
        self.session.verify = False

    def login(self):
        login_url = f"{self.base_url}/j_security_check"
        self.session.post(login_url, data={
            "j_username": self.username,
            "j_password": self.password
        })
        token = self.session.get(f"{self.base_url}/dataservice/client/token").text
        self.session.headers.update({
            "X-XSRF-TOKEN": token,
            "Content-Type": "application/json"
        })
        print(f"✅ Logged into vManage: {self.host}")
        return self

    def get(self, ep):    return self.session.get(f"{self.base_url}/dataservice{ep}")
    def post(self, ep, d): return self.session.post(f"{self.base_url}/dataservice{ep}", json=d)
    def logout(self):     self.session.get(f"{self.base_url}/logout")
```

---

## 📋 devices.py — Device Inventory

```python
from auth import vManageSession
from tabulate import tabulate

def get_all_devices(s):
    return s.get("/device").json().get("data", [])

def print_device_table(devices):
    rows = [[d.get("host-name"), d.get("system-ip"), d.get("site-id"),
             d.get("device-model"), d.get("version"), d.get("reachability")]
            for d in devices]
    print(tabulate(rows, headers=["Hostname","System-IP","Site","Model","Version","Status"], tablefmt="grid"))

if __name__ == "__main__":
    s = vManageSession().login()
    print_device_table(get_all_devices(s))
    s.logout()
```

---

## 🖥️ Netmiko — Bulk CLI Collection

```python
from netmiko import ConnectHandler
import os
from datetime import datetime

DEVICES = [
    {"host": "192.168.100.20", "name": "vEdge-HQ",  "type": "cisco_xr"},
    {"host": "192.168.100.21", "name": "vEdge-BR1", "type": "cisco_xr"},
    {"host": "192.168.100.23", "name": "cEdge-DC1", "type": "cisco_xe"},
]

COMMANDS = [
    "show sdwan control connections",
    "show sdwan omp peers",
    "show sdwan bfd sessions",
    "show sdwan omp routes",
    "show sdwan ip route vpn 1",
]

def collect(device):
    params = {"device_type": device["type"], "host": device["host"],
              "username": os.getenv("EDGE_USER","admin"),
              "password": os.getenv("EDGE_PASS","admin")}
    results = {}
    with ConnectHandler(**params) as conn:
        for cmd in COMMANDS:
            results[cmd] = conn.send_command(cmd)
    return results

if __name__ == "__main__":
    for d in DEVICES:
        out = collect(d)
        ts = datetime.now().strftime("%Y%m%d_%H%M%S")
        with open(f"output_{d['name']}_{ts}.txt", "w") as f:
            for cmd, res in out.items():
                f.write(f"\n{'='*60}\n{cmd}\n{'='*60}\n{res}\n")
        print(f"✅ Saved output for {d['name']}")
```

---

## 🚀 Usage

```bash
python3 sdwan_api/auth.py          # test connectivity
python3 sdwan_api/devices.py       # print device inventory
python3 sdwan_api/templates.py     # list templates
python3 sdwan_api/policies.py      # list policies
python3 netmiko_scripts/collect_show_commands.py  # CLI bulk collect
```

---

[← Back to Main](../../README.md) | [Ansible →](../ansible/README.md)
