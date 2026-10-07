# install_sftpgo

Ansible role to deploy [SFTPGo](https://github.com/drakkan/sftpgo) with Nginx using Docker Compose.

## Requirements

**Docker must already be installed** on the target host.

You can use the `install_docker` role from this repository:

```yaml
roles:
  - install_docker
  - install_sftpgo
```

Or install Docker using the [official Docker documentation](https://docs.docker.com/engine/install/).

also a **DNS hostname** pointing to the server.

---

## Variables

```yaml
# install_sftpgo/vars/main.yml
---
install_sftpgo:
  base_path: "/opt/sftpgo"
  fqdn: "ftp.example.com"

  images:
    sftpgo: "drakkan/sftpgo:v2.7.5"
    nginx: "nginx:1.30.1"
```

| Variable | Description |
|---|---|
| `base_path` | Installation/data directory |
| `fqdn` | Hostname used by Nginx and SSL |
| `images.sftpgo` | SFTPGo Docker image |
| `images.nginx` | Nginx Docker image |

---
## Deploy

Create `inventory` and `site.yml`:

```yaml
# site.yml
---
- name: Deploy SFTPGo
  hosts: sftpgo
  become: true
  roles:
    - install_sftpgo
```

Run:

```bash
ansible-playbook -i inventory site.yml
```

After deployment and **Initial Setup**:

```text
WebAdmin:  https://ftp.example.com/web/admin
WebClient: https://ftp.example.com/web/client
SFTP:      sftp -P 2022 user@ftp.example.com
```
---
## Firewall

| Service | Protocol | Port(s) | Required |
|---|---|---:|---|
| HTTP → HTTPS redirect | TCP | 80 | Yes  |
| HTTPS / WebAdmin / WebClient | TCP | 443 | Yes |
| SFTP | TCP | 2022 | Yes |
| FTP | TCP | 21 | Only if FTP is required |
| FTP passive mode | TCP | 50000–50100 | Only if FTP is required |

**SFTP is recommended instead of FTP whenever possible.**

---
## SSL

To replace the certificate and key:

```bash
cp fullchain.pem /opt/sftpgo/ssl/sftpgo.crt
cp privkey.pem /opt/sftpgo/ssl/sftpgo.key
```

Restart Nginx:

```bash
cd /opt/sftpgo/docker-compose
docker compose restart nginx
```

---

## Data

Persistent SFTPGo data is stored under:

```text
{{ install_sftpgo.base_path }}/data/
```

**Back up this directory before upgrades or major configuration changes.**

---
