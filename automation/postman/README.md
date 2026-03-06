# 🔌 Automation — Postman / REST API

[← Back to Main](../../README.md) | [Python →](../python/README.md) | [Ansible →](../ansible/README.md) | [Terraform →](../terraform/README.md)

---

## 📦 Setup

1. Download Postman: https://www.postman.com/downloads/
2. Import `Cisco-SDWAN-Collection.json`
3. Set environment variables: `vmanage_host`, `vmanage_port`, `username`, `password`

---

## 🔑 Authentication

```
POST /j_security_check
Body (form): j_username=admin&j_password=admin

GET /dataservice/client/token
→ Use returned value as X-XSRF-TOKEN header on all POST/PUT/DELETE
```

---

## 📋 Key Endpoints

| Operation | Method | Endpoint |
|-----------|--------|----------|
| List devices | GET | `/dataservice/device` |
| Device status | GET | `/dataservice/device/monitor` |
| List templates | GET | `/dataservice/template/device` |
| Attach template | POST | `/dataservice/template/device/config/attachfeature` |
| List vSmart policies | GET | `/dataservice/template/policy/vsmart` |
| Activate policy | POST | `/dataservice/template/policy/vsmart/activate/{id}` |
| BFD sessions | GET | `/dataservice/device/bfd/sessions?deviceId={ip}` |
| OMP routes | GET | `/dataservice/device/omp/routes/received?deviceId={ip}` |
| Control connections | GET | `/dataservice/device/control/connections?deviceId={ip}` |
| Active alarms | GET | `/dataservice/alarms?acknowledged=false` |

---

## 🔍 Swagger UI

Access full interactive API docs directly on your vManage:
```
https://192.168.100.10:8443/apidocs/
```

---

## 💡 Tips

- All POST/PUT/DELETE require `X-XSRF-TOKEN` header
- Action APIs are **async** — poll `/device/action/status/{id}` for result
- Use `?count=100` for more results on paginated endpoints

---

[← Back to Main](../../README.md) | [Python →](../python/README.md)
