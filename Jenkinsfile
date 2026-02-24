pipeline {
  agent any

  triggers {
    cron('H/5 * * * 1')
  }

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build + Test') {
      steps {
        script {
          if (isUnix()) {
            sh 'chmod +x mvnw'
            sh './mvnw -B clean test'
          } else {
            bat 'mvnw.cmd -B clean test'
          }
        }
      }
    }

    stage('Jacoco Report') {
      steps {
        script {

          if (isUnix()) {
            sh './mvnw -B jacoco:report'
          } else {
            bat 'mvnw.cmd -B jacoco:report'
          }
        }

        publishHTML(target: [
          allowMissing: false,
          alwaysLinkToLastBuild: true,
          keepAll: true,
          reportDir: 'target/site/jacoco',
          reportFiles: 'index.html',
          reportName: 'JaCoCo Coverage Report'
        ])
      }
    }

    stage('Package Artifact') {
      steps {
        script {
          if (isUnix()) {
            sh './mvnw -B -DskipTests package'
          } else {
            bat 'mvnw.cmd -B -DskipTests package'
          }
        }
      }
    }
  }

  post {
    always {
      junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'

      archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
    }
  }
}
