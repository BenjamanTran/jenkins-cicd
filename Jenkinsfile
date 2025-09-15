pipeline {
  agent any

  parameters {
    string(name: 'REPO_URL', defaultValue: 'https://github.com/BenjamanTran/jenkins-cicd.git', description: 'Application repository URL')
    string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch to build')
    choice(name: 'DEPLOY_ENV', choices: ['local', 'remote', 'both'], description: 'Where to deploy')
    string(name: 'KEEP_RELEASES', defaultValue: '5', description: 'How many releases to keep')
    string(name: 'FIREBASE_PROJECT_ID', defaultValue: '', description: 'Firebase project ID (optional)')
  }

  environment {
    // GOOGLE_APPLICATION_CREDENTIALS or FIREBASE_TOKEN should be provided by Docker/Env
    KEEP_RELEASES = "${params.KEEP_RELEASES}"
    FIREBASE_PROJECT_ID = "${params.FIREBASE_PROJECT_ID}"
    DEPLOY_ENV = "${params.DEPLOY_ENV}"
  }

  stages {
    stage('Checkout (infra)') {
      steps {
        checkout scm
      }
    }

    stage('Checkout (app)') {
      steps {
        dir('source') {
          checkout([$class: 'GitSCM', branches: [[name: params.BRANCH]], userRemoteConfigs: [[url: params.REPO_URL]]])
        }
      }
    }

    stage('Build') {
      steps {
        dir('source') {
          sh '''
            if [ -f package.json ]; then
              (npm ci || npm install)
            else
              echo "No package.json, skip install"
            fi
          '''
        }
      }
    }

    stage('Lint/Test') {
      steps {
        dir('source') {
          sh '''
              npm run test:ci
          '''
        }
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          set -e
          export FIREBASE_PROJECT_ID="$FIREBASE_PROJECT_ID"
          ./scripts/firebase-deploy.sh app

          export DEPLOY_ENV="$DEPLOY_ENV"
          export KEEP_RELEASES="$KEEP_RELEASES"
          ./deploy.sh
        '''
      }
    }
  }
}
