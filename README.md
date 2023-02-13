# CloudComposer
for create GitHub actions and deploy to Cloud Composer in GCP
Here's a basic outline of the steps to deploy to Cloud Composer using GitHub Actions:

- Create a Cloud Composer environment in GCP: If you haven't already, create a Cloud Composer environment in GCP. This environment will be where your DAGs (directed acyclic graphs) will run.

- Create a GitHub Actions workflow: In your GitHub repository, create a new workflow file in the .github/workflows directory. This file will define the steps that GitHub Actions should take when deploying to Cloud Composer.

- Authenticate with GCP: To deploy to Cloud Composer, you'll need to authenticate with GCP using a service account. You can do this by creating a new service account in GCP, downloading its private key, and then using the gcloud CLI in your workflow to authenticate with the service account.

- Deploy to Cloud Composer: In your workflow, you can use the gcloud CLI to interact with the Cloud Composer API and deploy your DAGs. You can either create a new environment or update an existing environment.

Here is an example of a GitHub Actions workflow that deploys to Cloud Composer:

```yaml
name: Deploy to Cloud Composer

on:
  pull_request:
    branches:
      - develop
  push:
    branches:
      - features/*

env:
  GCP_PROJECT: my-gcp-project
  GCP_COMPOSER_LOCATION: us-central1
  GCP_COMPOSER_ENVIRONMENT: my-composer-environment

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Authenticate to GCP
      uses: google-auth/setup-gcloud@v2
      with:
        service_account_email: ${{ secrets.GCP_SA_EMAIL }}
        project_id: ${{ env.GCP_PROJECT }}

    - name: Deploy to Cloud Composer
      run: |
        if [[ "${{ github.ref }}" == "refs/heads/develop" && "${{ github.event_name }}" == "pull_request" ]]; then
          gcloud composer environments update ${GCP_COMPOSER_ENVIRONMENT} \
            --location ${GCP_COMPOSER_LOCATION} \
            --update-dag
        else
          gcloud composer environments update ${GCP_COMPOSER_ENVIRONMENT} \
            --location ${GCP_COMPOSER_LOCATION} \
            --update-dag-run
        fi

```


```yaml
name: Deploy to Cloud Composer

on:
  push:
    branches:
      - main

env:
  PROJECT_ID: my-gcp-project
  COMPOSER_ENVIRONMENT: my-composer-environment

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Authenticate with GCP
      uses: google-auth/setup-gcloud@v1
      with:
        service_account_key: ${{ secrets.GCP_SA_KEY }}

    - name: Deploy to Cloud Composer
      run: |
        gcloud auth activate-service-account --key-file=${{ secrets.GCP_SA_KEY }}
        gcloud config set project $PROJECT_ID
        gcloud composer environments update $COMPOSER_ENVIRONMENT --update-dag-gcs-prefix=gs://my-dag-bucket

```

here's another example of a GitHub Actions workflow that deploys a Cloud Composer environment:

```yamm
name: Deploy to Cloud Composer

on:
  push:
    branches:
      - main

env:
  PROJECT_ID: my-gcp-project
  COMPOSER_ENVIRONMENT: my-composer-environment
  CLOUD_STORAGE_BUCKET: my-cloud-storage-bucket

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Authenticate with GCP
      uses: google-auth/setup-gcloud@v1
      with:
        service_account_key: ${{ secrets.GCP_SA_KEY }}

    - name: Deploy to Cloud Composer
      run: |
        gcloud auth activate-service-account --key-file=${{ secrets.GCP_SA_KEY }}
        gcloud config set project $PROJECT_ID
        gcloud composer environments create $COMPOSER_ENVIRONMENT \
          --location=us-central1 \
          --zone=us-central1-a \
          --machine-type=n1-standard-2 \
          --disk-size=100GB \
          --python-version=3 \
          --environment-variables=GOOGLE_CLOUD_PROJECT=$PROJECT_ID,GOOGLE_CLOUD_BUCKET=$CLOUD_STORAGE_BUCKET

```

deploy to Cloud Composer in Google Cloud Platform (GCP) with manual deployment to the dev environment and automatic deployment to feature branches.

```

name: Deploy to Cloud Composer

on:
  push:
    branches:
      - dev
      - features/*

env:
  GCP_PROJECT: my-gcp-project
  GCP_COMPOSER_LOCATION: us-central1
  GCP_COMPOSER_ENVIRONMENT: my-composer-environment

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Authenticate to GCP
      uses: google-auth/setup-gcloud@v2
      with:
        service_account_email: ${{ secrets.GCP_SA_EMAIL }}
        project_id: ${{ env.GCP_PROJECT }}

    - name: Update Cloud Composer
      if: ${{ startsWith(github.ref, 'refs/heads/features/') }}
      run: |
        gcloud composer environments update ${GCP_COMPOSER_ENVIRONMENT} \
          --location ${GCP_COMPOSER_LOCATION} \
          --update-dag-run

    - name: Deploy to dev environment
      if: ${{ github.ref == 'refs/heads/dev' }}
      run: |
        gcloud composer environments update ${GCP_COMPOSER_ENVIRONMENT} \
          --location ${GCP_COMPOSER_LOCATION} \
          --update-dag-run
```
second way

```
name: Deploy to Cloud Composer

on:
  push:
    branches:
      - dev
      - features/*

env:
  GCP_PROJECT: my-gcp-project
  GCP_COMPOSER_LOCATION: us-central1
  GCP_COMPOSER_ENVIRONMENT: my-composer-environment

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Authenticate to GCP
      uses: google-auth/setup-gcloud@v2
      with:
        service_account_email: ${{ secrets.GCP_SA_EMAIL }}
        project_id: ${{ env.GCP_PROJECT }}

    - name: Deploy to Cloud Composer
      run: |
        if [[ "${{ github.ref }}" == "refs/heads/dev" ]]; then
          gcloud composer environments update ${GCP_COMPOSER_ENVIRONMENT} \
            --location ${GCP_COMPOSER_LOCATION} \
            --update-dag
        else
          gcloud composer environments update ${GCP_COMPOSER_ENVIRONMENT} \
            --location ${GCP_COMPOSER_LOCATION} \
            --update-dag-run
        fi
```
