pipeline {
  agent any

  triggers {
    cron('H/5 * * * 1')
  }

  options { timestamps() }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build and Test with JaCoCo') {
      steps {
        bat 'gradlew.bat clean test jacocoTestReport'
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: '**\\build\\test-results\\test\\*.xml'
          publishHTML(target: [
            allowMissing: true,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'build/reports/jacoco/test/html',
            reportFiles: 'index.html',
            reportName: 'JaCoCo Report'
          ])
        }
      }
    }

    stage('Package Artifact') {
      steps {
        bat 'gradlew.bat build -x test'
      }
      post {
        success {
          archiveArtifacts artifacts: 'build\\libs\\*.jar', fingerprint: true
        }
      }
    }
  }
}
