# Flask Backend – Jenkins CI/CD Deployment

## 1. Project Overview

This project demonstrates Continuous Integration and Continuous Deployment (CI/CD) for a Python Flask backend application using GitHub, Jenkins, and AWS EC2.

Whenever a developer pushes code to the GitHub repository, a GitHub webhook notifies Jenkins. Jenkins executes the configured pipeline, validates the application, runs endpoint tests, and deploys the application to the EC2 instance.

## 2. Technologies Used

- Python
- Flask
- Gunicorn
- Git and GitHub
- Jenkins
- AWS EC2 (Ubuntu)
- Linux systemd
- GitHub Webhooks

## 3. Architecture

Developer → GitHub Repository → GitHub Webhook → Jenkins Pipeline → AWS EC2 → Flask Application

The Flask application runs as a systemd service using Gunicorn on port `5000`.

## 4. Prerequisites

- An AWS EC2 Ubuntu instance
- Python 3 and pip
- Git
- Jenkins installed and running
- An accessible GitHub repository
- A configured GitHub webhook
- Appropriate AWS Security Group rules

## 5. GitHub Repository

Repository: https://github.com/krishna-280792shinde/jenkins-cicd-flask

## 6. Run the Application Locally

Clone the repository:

```bash
git clone https://github.com/krishna-280792shinde/jenkins-cicd-flask.git
cd jenkins-cicd-flask
```

Create and activate a virtual environment.

On Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

On Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Install the dependencies:

```bash
python -m pip install -r requirements.txt
```

Run the application using the project's configured development entry point. If `app.py` exposes the Flask object as `app`, you can run:

```bash
python app.py
```

## 7. Jenkins Pipeline Configuration

Create or configure the Jenkins job:

`Flask-CI-Pipeline`

The pipeline includes the following stages:

1. Checkout source code from GitHub.
2. Check the Python environment.
3. Install application dependencies.
4. Validate Python files.
5. Test Flask endpoints.
6. Deploy the application to EC2.

The deployment script is:

```text
/usr/local/sbin/deploy-flask
```

The script updates the deployment checkout, installs dependencies, validates the application, restarts the Flask service, and checks application health.

## 8. Configure the GitHub Webhook

In the GitHub repository, navigate to:

**Settings → Webhooks → Add webhook**

Configure the webhook as follows:

| Setting | Value |
|---|---|
| Payload URL | `http://xx.xxx.xxx.xxx:8080/github-webhook/` |
| Content type | `application/json` |
| Events | Just the push event |
| Active | Enabled |

In Jenkins, open the `Flask-CI-Pipeline` job and enable **GitHub hook trigger for GITScm polling** under Build Triggers.

The webhook should trigger a new Jenkins build when a change is pushed to the branch configured in the job.

## 9. Deployment and Verification

The deployed Flask service is managed by systemd:

```text
Service: flask-backend
Port: 5000
```

Check the service:

```bash
sudo systemctl status flask-backend --no-pager
```

Check the health endpoint locally on EC2:

```bash
curl -i http://127.0.0.1:5000/health
```

Expected response:

```json
{"status":"healthy"}
```

Public application URL:

http://xx.xxx.xxx.xxx:5000/

Health-check URL:

http://xx.xxx.xxx.xxx:5000/health

The deployed application uses Gunicorn bound to `0.0.0.0:5000` so it can accept external connections when the AWS Security Group permits them.

## 10. Troubleshooting

### Check service logs

```bash
sudo journalctl -u flask-backend -n 100 --no-pager
```

### Restart the application

```bash
sudo systemctl restart flask-backend
```

### Check the listening port

```bash
sudo ss -ltnp | grep ':5000'
```

### Check Jenkins logs

```bash
sudo journalctl -u jenkins -n 100 --no-pager
```

If the application is healthy locally but unavailable externally, verify the Gunicorn bind address, EC2 Security Group inbound rules, and public IP address.

## 11. Security Considerations

- Never commit AWS credentials or private keys to GitHub.
- Restrict SSH access to trusted IP addresses.
- Avoid exposing application ports to all internet addresses unless required.
- Protect Jenkins with authentication and appropriate authorization.
- Prefer HTTPS and secure webhook configurations for production deployments.

## 12. Conclusion

This project demonstrates automated Flask application deployment using Jenkins CI/CD, GitHub webhooks, AWS EC2, Gunicorn, and systemd.

The pipeline was tested successfully, and a GitHub push triggered a new Jenkins build automatically.