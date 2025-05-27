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

- Successful GitHub Actions run
- Tests passing log
- Deployment logs (from EC2)
- Running Flask app in browser

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
