Setup guide to automate deployment process to dev and prod server. 
With this setup, every time you push to the `prod` branch on either the frontend or backend repositories, the code will automatically get deployed to your GCE instance.
Here's a step-by-step guide:

### 1. **Prerequisites**:

- **Google Cloud SDK**: Make sure you've installed and configured the Google Cloud SDK on your local machine.
  
- **Server Configuration**: Ensure that your Google Compute Engine (GCE) instances have been set up correctly and are accessible via SSH. Make sure your MySQL is also appropriately configured and secure.

### 2. **Setting Up CI/CD Tool**:
   
I'll use GitHub Actions as the CI/CD tool for this example because it's integrated with GitHub, but similar steps would apply if you use GitLab CI/CD, Jenkins, or others.

#### 2.1. **GitHub Actions Setup**:

- On your Vue.js (frontend) and PHP Laravel (backend) repositories, create a `.github/workflows` directory.
  
- Inside this directory, create a YAML file, say `deploy.yml`, in both repos.

### 3. **Frontend - Vue.js Deployment**:

#### 3.1. **Building**:

In the `deploy.yml` file of your frontend repo:

```yaml
name: Deploy Vue.js Frontend

on:
  push:
    branches:
      - prod

jobs:
  build_and_deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      
      - name: Install dependencies and build
        run: |
          npm install
          npm run build
```

#### 3.2. **Deployment**:

After building the frontend, push the build files to your GCE instance:

```yaml
      - name: Deploy to GCE
        run: |
          gcloud compute scp --recurse dist/* <YOUR-GCE-INSTANCE-NAME>:<PATH-TO-WEB-DIRECTORY> --zone=<YOUR-GCE-ZONE>
```

Remember to replace placeholders with your actual GCE instance name, path to your web directory, and zone.

### 4. **Backend - Laravel Deployment**:

#### 4.1. **Deployment Script**:

In the `deploy.yml` file of your backend repo:

```yaml
name: Deploy Laravel Backend

on:
  push:
    branches:
      - prod

jobs:
  deploy_backend:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Deploy to GCE
        run: |
          gcloud compute scp --recurse . <YOUR-GCE-INSTANCE-NAME>:<PATH-TO-LARAVEL-DIRECTORY> --zone=<YOUR-GCE-ZONE>
          ssh <YOUR-GCE-INSTANCE-NAME> 'cd <PATH-TO-LARAVEL-DIRECTORY> && composer install && php artisan migrate'
```

Replace placeholders accordingly.

### 5. **Database**:

For the MySQL database:

- Ensure your database credentials are stored securely, preferably as encrypted environment variables or secrets in GitHub.
  
- Your Laravel backend should use these credentials, and the `php artisan migrate` command (included in the Laravel deployment script) will handle any database schema changes.

### 6. **Final Steps**:

- Store GCE SSH keys and other credentials securely as GitHub secrets. Reference them in the GitHub Actions script so that your CI/CD can authenticate and deploy to the GCE instances.

- Add necessary error handling and notification mechanisms in your GitHub Actions scripts to get alerted in case of deployment failures.

- Ensure proper firewall and security group configurations on GCE to keep your server and database secure.

