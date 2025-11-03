# 🚨 Important Notice

This repository is published as a **reference implementation** for educational purposes. Before using this code:

1. **Configure repository secrets** in your Gitea instance:
   - `REGISTRY_USER`: Your Gitea username for container registry
   - `REGISTRY_PASSWORD`: Your Gitea container registry token/password

2. **Configure repository variables**:
   - `REGISTRY_HOSTNAME`: Your Gitea instance hostname (e.g., `gitea.example.com`)

3. **Update workflow files** if using different registry or naming conventions

4. **Customize Dockerfiles** for your specific tool versions or requirements

5. **Configure act_runner** with appropriate labels to use these images

All sensitive configuration should be managed through CI/CD variables and repository secrets, not committed to the repository.

---

# 🛠️ Custom Docker Images for Gitea Runners

This repository contains Docker images designed for specialized CI/CD tasks in **Gitea Actions**. Each image is automatically built and pushed to your **Gitea container registry** with both `latest` and date-based tags.

---

## 📦 Available Images

### 🔧 Ansible Runner
**File:** `Dockerfile.ansible`  
**Purpose:** Automation and configuration management tasks  
**Registry:** `${REGISTRY_HOSTNAME}/${REGISTRY_USER}/ansible-runner`

### 💾 Backup Runner  
**File:** `Dockerfile.backup`  
**Purpose:** Kubernetes backup operations with restic, kubectl, and database tools  
**Registry:** `${REGISTRY_HOSTNAME}/${REGISTRY_USER}/backup-runner`  
**Includes:** Latest restic, kubectl, PostgreSQL client, Ubuntu 22.04 base

---

## 📅 Build Schedule

Both workflows run:
- ⏰ **Weekly**: Every **Monday at 3:00 AM UTC**
- 🚀 **Manual**: Via Gitea Actions UI
- 🔄 **On push**: When Dockerfiles are modified

---

## 🏷️ Image Tags

All images are tagged as:
- `latest` - Most recent build
- `YYYY-MM-DD` - Date-based versioning (e.g., `2025-10-17`)

Example registry paths:
```
${REGISTRY_HOSTNAME}/${REGISTRY_USER}/ansible-runner:latest
${REGISTRY_HOSTNAME}/${REGISTRY_USER}/ansible-runner:2025-10-17
${REGISTRY_HOSTNAME}/${REGISTRY_USER}/backup-runner:latest
${REGISTRY_HOSTNAME}/${REGISTRY_USER}/backup-runner:2025-10-17
```

---

## 🔐 Configuration

### 🔑 Required Secrets
| Name                | Description                          |
|---------------------|--------------------------------------|
| `REGISTRY_USER`     | Gitea username for registry login    |
| `REGISTRY_PASSWORD` | Gitea container registry token/password |

### ⚙️ Required Variables
| Name                | Description                                 |
|---------------------|---------------------------------------------|
| `REGISTRY_HOSTNAME` | Gitea instance hostname (e.g., `${REGISTRY_HOSTNAME}`) |

---

## 🏃‍♂️ Usage in Gitea Runners

Configure your `act_runner` with custom labels:

```yaml
# act_runner config
labels:
  - "ansible-runner:docker://${REGISTRY_HOSTNAME}/${REGISTRY_USER}/ansible-runner:latest"
  - "backup-runner:docker://${REGISTRY_HOSTNAME}/${REGISTRY_USER}/backup-runner:latest"
```

Use in workflows:

```yaml
# Ansible automation job
jobs:
  deploy:
    runs-on: [ansible-runner]
    steps:
      - uses: actions/checkout@v3
      - name: Run Ansible Playbook
        run: ansible-playbook deploy.yml

  # Backup job  
  backup:
    runs-on: [backup-runner]
    steps:
      - uses: actions/checkout@v3
      - name: Backup Database
        run: |
          kubectl get pods -n production
          restic backup /data
```

---

## 🐳 Image Specifications

### Ansible Runner
- **Base:** Ubuntu 22.04
- **Tools:** Ansible, Python, SSH, Git
- **Use cases:** Infrastructure automation, deployment pipelines

### Backup Runner
- **Base:** Ubuntu 22.04  
- **Tools:** Restic (latest), kubectl, PostgreSQL client, jq, curl
- **User:** Non-root `backup` user (UID 1001)
- **Use cases:** Kubernetes backups, database dumps, file archiving

---

## 🔧 Development

### Local Testing
```bash
# Build images locally
docker build -f Dockerfile.ansible -t ansible-runner:test .
docker build -f Dockerfile.backup -t backup-runner:test .

# Test functionality
docker run --rm ansible-runner:test ansible --version
docker run --rm backup-runner:test restic version
```

### Manual Workflow Trigger
1. Navigate to **Actions** tab in Gitea
2. Select desired workflow:
   - **Build and Push Ansible Runner**
   - **Build and Push Backup Runner**
3. Click **Run workflow**

---

## 🛟 Troubleshooting

### Common Issues

**Build failures:**
- Check Dockerfile syntax
- Verify package availability in Ubuntu repositories
- Review build logs in Actions tab

**Registry authentication:**
- Confirm `REGISTRY_USER` and `REGISTRY_PASSWORD` secrets
- Verify `REGISTRY_HOSTNAME` variable matches your Gitea instance

**Runner usage:**
- Ensure act_runner configuration includes custom labels
- Check image pull permissions in your namespace
- Verify workflow `runs-on` labels match act_runner config

### Debugging Commands
```bash
# Check runner logs
docker logs gitea-runner

# Test image locally
docker run --rm -it <image-name> /bin/bash

# Verify restic version compatibility
docker run --rm backup-runner:latest restic version
```

---

## 🧼 Maintenance

### Image Cleanup
- Old images can be managed via Gitea's container registry UI
- Consider implementing retention policies for date-tagged images
- Monitor registry storage usage

### Updates
- Images rebuild weekly to incorporate security updates
- Monitor upstream tool releases (restic, kubectl, ansible)
- Update Dockerfiles for new tool versions as needed

---

## 📚 Related Documentation

- [Gitea Actions Overview](https://docs.gitea.com/usage/actions/overview)
- [act_runner Configuration](https://docs.gitea.com/usage/actions/act_runner)
- [Restic Documentation](https://restic.readthedocs.io/)
- [Ansible Documentation](https://docs.ansible.com/)