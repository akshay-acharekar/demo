This GitHub Actions pipeline automates deployment to a remote server (likely an **AWS EC2 instance**) when code is pushed to the `main` branch or triggered manually.

---

## **Breaking Down the Workflow**
### **1. Workflow Name & Trigger**
```yaml
name: GitHub Actions Demo
run-name: Deployment initialized by ${{ github.actor }}
```
- The workflow is named **"GitHub Actions Demo."**
- `run-name` dynamically displays **who triggered the deployment** (`github.actor`).

```yaml
on:
  workflow_dispatch:
  push:
    branches:
      - main
```
- **`workflow_dispatch`** allows **manual trigger** via the GitHub Actions UI.
- **`push` to `main`** automatically triggers deployment.

---

### **2. Job: `deploy`**
```yaml
jobs:
    deploy:
        runs-on: ubuntu-latest
```
- The **`deploy`** job runs on an **Ubuntu-based** virtual machine.

---

### **3. Steps in the Deployment Process**
#### **Step 1: Checkout Repository**
```yaml
- name: Checkout repository
  uses: actions/checkout@v2
```
- **Clones** the repository onto the GitHub runner.

#### **Step 2: Securely Copy Files to the Remote Server**
```yaml
- name: Copying files to remote server
  uses: appleboy/scp-action@v0.1.7
  with:
    host: ${{ secrets.EC2_HOST }}
    username: ${{ secrets.USERNAME }}
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    source: "index.html"
    target: "/home/ubuntu"
```
- Uses **`appleboy/scp-action`** to securely copy `index.html` to the remote EC2 instance.
- **Credentials (`host`, `username`, `key`) are stored as GitHub Secrets**.

#### **Step 3: Deploying Application**
```yaml
- name: Deploying Application
  uses: appleboy/ssh-action@v1
  with:
    host: ${{ secrets.EC2_HOST }}
    username: ${{ secrets.USERNAME }}
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    script: |
      sudo mv index.html /var/www/html
      sudo systemctl restart nginx
```
- Uses **`appleboy/ssh-action`** to:
  1. **Move `index.html`** to the **Nginx web root (`/var/www/html`)**.
  2. **Restart Nginx** to apply changes.

#### **Step 4: Print Success Message**
```yaml
- run: echo "Deployment successful"
```
- **Prints a message** after successful deployment.

---

## **Secrets Used**
Stored in **GitHub Secrets** for security:
| Secret Name         | Purpose |
|---------------------|---------|
| `EC2_HOST`         | Public IP or hostname of the EC2 instance |
| `USERNAME`         | SSH username (e.g., `ubuntu` for AWS) |
| `SSH_PRIVATE_KEY`  | Private SSH key for authentication |

---

## **How This Workflow Works**
✅ **Triggered by**:  
- **Push to `main`** OR **Manual Trigger via GitHub UI**  
✅ **Performs the following actions**:  
1. **Clones repository**  
2. **Copies `index.html` to EC2**  
3. **Moves the file to Nginx web root**  
4. **Restarts Nginx** to apply changes  
5. **Confirms successful deployment**  

---

## **Final Thoughts**
This GitHub Actions workflow provides an **automated deployment pipeline** for updating a remote **EC2-hosted** Nginx web server.
