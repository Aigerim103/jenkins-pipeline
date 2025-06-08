
# Interactive Flask App CI/CD Project

This project demonstrates a CI/CD pipeline using **Jenkins**, **Docker**, and **GitHub** with an interactive Flask web application.

## 🔧 Components

- ✅ Jenkinsfile
- ✅ Dockerfile & Docker Compose
- ✅ GitHub integration with external repository
- ✅ Email notifications
- ✅ Automatic versioning
- ✅ Health checks
- ✅ Rollback on failure
- ✅ Custom interactive Flask application

---

## 🚀 Pipeline Stages

1. **Clone Project**: Jenkins pulls the Flask project from GitHub.
2. **Set Version**: Pipeline generates a version based on build number and date, stored in `version.txt`.
3. **Build Docker Image**: Docker Compose builds the image.
4. **Run Containers**: Launch the application using `docker-compose up -d`.
5. **Health Check**: Performs an HTTP request to `http://localhost:5000`.
6. **Notification**: Sends email notifications on success or failure.
7. **Rollback**: Docker containers are stopped on failure.

---

## 📁 File Structure

- `/Jenkinsfile`: Jenkins pipeline definition.
- `/interactive-site`: Flask web app with:
  - `app.py`: main application
  - `templates/`: HTML templates
  - `static/`: optional CSS/JS
  - `Dockerfile`: image build
  - `docker-compose.yml`: multi-container setup
  - `requirements.txt`: Python dependencies
  - `version.txt`: generated during pipeline

---

## 🌐 Custom Web App

The app includes an interactive HTML form that allows users to submit messages, which are printed in console.

```python
@app.route('/', methods=['GET', 'POST'])
def home():
    if request.method == 'POST':
        message = request.form.get('message')
        print("Received:", message)
        return render_template('response.html', message=message)
    return render_template('form.html')
```

---

## 📬 Email Configuration

The Jenkins pipeline uses email notifications via Gmail SMTP.  
You need to set up an **App Password** in your Gmail account and configure Jenkins accordingly.

---

## 📌 How to Run Locally

```bash
git clone https://github.com/Aigerim103/interactive-site.git
cd interactive-site
docker-compose up --build
```

Visit `http://localhost:5000`

---

## ✅ Achievements & Points

| Criteria                                                      | Points |
|---------------------------------------------------------------|--------|
| Jenkins installation and configuration                        | 1.0    |
| Use of Jenkinsfile in GitHub repository                       | 1.0    |
| Creating Dockerfile and building Docker images                | 1.0    |
| Defining full application lifecycle in Jenkinsfile            | 0.25   |
| Docker Compose for multi-container setup                      | 0.25   |
| Rollback on failure                                           | 0.25   |
| Email notifications                                           | 0.5    |
| Automatic versioning                                          | 0.25   |
| Custom interactive Flask application                          | 0.25   |
| **Total**                                                     | **4.75** |

---

## 📝 Author

Created by Aigerim Kairatova as part of the DevOps CI/CD project.
