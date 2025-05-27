# Flask App with GitHub Actions CI/CD Pipeline
This repository demonstrates a complete CI/CD pipeline setup for a Python Flask application using **GitHub Actions**. It supports automated deployment to **Amazon EC2** instances for both **staging** and **production** environments.
---

## Project Overview
- **app.py**: Core Flask application.
- **test_app.py**: Test cases using `pytest`.
- **.github/workflows/ci-cd.yml**: GitHub Actions workflow file.
- **.env**: Environment variable file (created during deployment).
- **requirements.txt**: Lists all Python dependencies.
---

## CI/CD Pipeline
### Triggers
- **On push** to `main` → Deploy to **production**
- **On push** to `staging` → Deploy to **staging**
- **On release** creation → Deploy to **production**

### Workflow Jobs
| Step              | Description                                               |
|-------------------|-----------------------------------------------------------|
|  Checkout         | Clones the repository code                                |
|  Set up Python    | Initializes Python environment                            |
|  Install Deps     | Installs dependencies via pip                             |
|  Run Tests        | Executes test suite using `pytest`                        |
|  Decode SSH Key   | Converts base64 key into `key.pem` for EC2 SSH access     |
|  Deploy           | SSH into EC2 and deploys Flask app                        |
---

## Deployment Targets
| Environment | Trigger                 | EC2 Host Secret        |
|-------------|--------------------------|------------------------|
| Staging     | Push to `staging` branch | `EC2_HOST_STAGING`     |
| Production  | Release tag or `main`    | `EC2_HOST_PRODUCTION`  |
---

## GitHub Secrets
Configure these secrets in **GitHub > Settings > Secrets and Variables > Actions**:
| Secret Name            | Purpose                                      |
|------------------------|----------------------------------------------|
| `MONGO_URI`            | MongoDB connection string                    |
| `EC2_USER`             | SSH user for EC2 (e.g., `ec2-user`)          |
| `EC2_KEY`              | Base64-encoded private SSH key               |
| `EC2_HOST_STAGING`     | Staging EC2 public IP                        |
| `EC2_HOST_PRODUCTION`  | Production EC2 public IP                     |
---

## .env File Handling
During deployment, the CI/CD pipeline automatically writes the `.env` file to the EC2 instance using the `MONGO_URI` secret.
**Sample `.env` format:**
```env
MONGO_URI=mongodb+srv://user:pass@cluster.mongodb.net/db
```
**CI Script Snippet (already in your workflow):**
```bash
echo "MONGO_URI=${{ secrets.MONGO_URI }}" > .env
```
Your Flask app loads this file using `python-dotenv`. Make sure the following is present in your `app.py`:
```python
from dotenv import load_dotenv
load_dotenv()
```
---

## Flask App with systemd (on EC2)
Create `/etc/systemd/system/flaskapp.service`:

```ini
[Unit]
Description=Flask App
After=network.target

[Service]
User=ec2-user
WorkingDirectory=/home/ec2-user/GithubActions_FlaskApp
EnvironmentFile=/home/ec2-user/GithubActions_FlaskApp/.env
ExecStart=/home/ec2-user/GithubActions_FlaskApp/venv/bin/python app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

Then run:
```bash
sudo systemctl daemon-reexec
sudo systemctl enable flaskapp
sudo systemctl start flaskapp
```
---

## 📸 Screenshots (Add These)
- Successful GitHub Actions run<br>
  <img width="398" alt="image" src="https://github.com/user-attachments/assets/2281bb55-6d8f-4d2a-ab70-885b59035b44" /><br>
  ![image](https://github.com/user-attachments/assets/ce563d02-7d51-409c-a24d-b599db5fb67a)<br>

- Tests passing log<br>
  <img width="578" alt="image" src="https://github.com/user-attachments/assets/6e7f2a02-db5e-47ec-b321-7526cb50765f" /><br>

- Deployment logs (from EC2)<br>
  PROD<br>
  ![image](https://github.com/user-attachments/assets/2feb51e5-ede4-4e02-9f89-5d26322dd6b0)<br>
  ![image](https://github.com/user-attachments/assets/e5a333d4-66ba-45f1-a0e1-ae83e5df4539)<br>
  
  STAGE<br>
  ![image](https://github.com/user-attachments/assets/746ef014-afff-4f05-b554-7df9ce890c12)<br>
  ![image](https://github.com/user-attachments/assets/8db13e4e-afdf-4fe2-90b3-112b7cd3dc4e)<br>

- Running Flask app in browser<br>
  PROD<br>
  <img width="200" alt="image" src="https://github.com/user-attachments/assets/5e06e8ba-a17f-4387-95e0-72c6efb62668" /><br>
  <img width="215" alt="image" src="https://github.com/user-attachments/assets/0f309be4-2f05-48ff-abae-b938871481c8" /><br>
  
  STAGE<br>
  <img width="247" alt="image" src="https://github.com/user-attachments/assets/cbe50025-b48f-49d4-8430-938334ac93f9" /><br>
  <img width="232" alt="image" src="https://github.com/user-attachments/assets/1cbb6e41-7869-4fa6-a7c8-205c7625ae85" /><br>

---

## 📌 Git Commands

```bash
# Show commit history
git log --oneline

# Create release tag
git tag -a v1.0 -m "Release v1.0"
git push origin v1.0
```

---

## 🙋 Need Help?

For queries, raise an issue or contact [Tanuj Bhatia](mailto:tanujbhatia24@gmail.com).
