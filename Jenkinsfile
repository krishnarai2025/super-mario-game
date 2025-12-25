pipeline {
  agent any

  environment {
    PROJECT_ID = "terraform-gcp-lab-2025"
    REGION = "us-central1"
    AR_REPO = "flask-docker-repo"
    IMAGE_NAME = "super-mario-mimic"
  }

  stages {

    stage('Checkout Source') {
      steps {
        checkout scm
      }
    }

    stage('Build Image') {
      steps {
        script {
          IMAGE_TAG = sh(
            script: 'echo $(git rev-parse --short HEAD)-$(date +%Y%m%d%H%M%S)',
            returnStdout: true
).trim()

          env.FULL_IMAGE = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${AR_REPO}/${IMAGE_NAME}:${IMAGE_TAG}"

          sh """
            docker build -t ${FULL_IMAGE} .
          """
        }
      }
    }

    stage('Authenticate Docker to Artifact Registry') {
      steps {
        sh """
          gcloud auth configure-docker ${REGION}-docker.pkg.dev --quiet
        """
      }
    }

    stage('Push Image to Artifact Registry') {
      steps {
        sh "docker push ${FULL_IMAGE}"
      }
    }

    stage('Trigger CD Pipeline') {
      steps {
        script {
          build job: 'gcp-mig-cd-pipeline',
          parameters: [
            string(name: 'IMAGE_TAG', value: IMAGE_TAG),
            string(name: 'AR_REPO', value: AR_REPO),
            string(name: 'IMAGE_NAME', value: IMAGE_NAME),
            string(name: 'PROJECT_ID', value: PROJECT_ID)
          ],
          wait: false
        }
      }
    }
  }

  post {
    success {
      echo "CI Success — Image Built & Pushed. CD Triggered Automatically."
    }
    failure {
      echo "CI Failed"
    }
  }
}
