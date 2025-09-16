pipeline {
  agent any

  parameters {
    string(name: 'REPO_URL', defaultValue: 'https://github.com/BenjamanTran/jenkins-cicd.git', description: 'Application repository URL')
    string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch to build')
    choice(name: 'DEPLOY_ENV', choices: ['local', 'remote', 'both'], description: 'Where to deploy')
    string(name: 'KEEP_RELEASES', defaultValue: '5', description: 'How many releases to keep')
    string(name: 'FIREBASE_PROJECT_ID', defaultValue: 'tantt-jenkins-2592e', description: 'Firebase project ID (optional)')
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
            if [ -f package.json ] && npm run -s | grep -q ' test:ci'; then
              npm run test:ci
            else
              echo "No test:ci script, skip tests"
            fi
          '''
        }
      }
    }

    stage('Trigger classic job') {
      steps {
        script {
          // Trigger classic job after pipeline stages succeed
          build job: 'deploy-application',
                wait: true,
                parameters: [
                  string(name: 'REPO_URL', value: params.REPO_URL),
                  string(name: 'BRANCH', value: params.BRANCH),
                  string(name: 'DEPLOY_ENV', value: params.DEPLOY_ENV),
                  string(name: 'KEEP_RELEASES', value: params.KEEP_RELEASES),
                  string(name: 'FIREBASE_PROJECT_ID', value: params.FIREBASE_PROJECT_ID)
                ]
        }
      }
    }
  }
}
