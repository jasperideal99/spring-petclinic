pipeline {
  agent any

  triggers {
    cron('H/5 * * * 1')
  }

  options { timestamps() }

  stages {

    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build + Test + JaCoCo') {
      steps {
        bat 'gradlew.bat clean test jacocoTestReport --no-daemon'
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: '**\\build\\test-results\\test\\*.xml'

          script {
            def reportPath = 'build/reports/jacoco/test/html'
            if (fileExists(reportPath)) {
              publishHTML(target: [
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: reportPath,
                reportFiles: 'index.html',
                reportName: 'JaCoCo Report'
              ])
            } else {
              echo "JaCoCo HTML report not found at ${reportPath} (skipping publishHTML)."
            }
          }
        }
      }
    }

    stage('Package Artifact') {
      steps {
        bat 'gradlew.bat build -x test --no-daemon'
      }
      post {
        success {
          archiveArtifacts artifacts: 'build\\libs\\*.jar', fingerprint: true
        }
      }
    }
  }
}
