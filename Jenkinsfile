pipeline {
  agent any

  triggers {
    cron('H/5 * * * 1')
  }

  options {
    timestamps()
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('JaCoCo Code Coverage') {
      steps {
        powershell '''
          $initFile = "$env:WORKSPACE\\jenkins-jacoco.gradle"

@"
allprojects {
  apply plugin: 'jacoco'

  tasks.withType(Test) {
    finalizedBy 'jacocoTestReport'
  }

  tasks.register('jacocoTestReport', JacocoReport) {
    dependsOn tasks.withType(Test)

    reports {
      xml.required = true
      html.required = true
    }

    if (project.hasProperty('sourceSets')) {
      sourceDirectories.setFrom files(sourceSets.main.allSource.srcDirs)
      classDirectories.setFrom files(sourceSets.main.output)
    }

    executionData.setFrom fileTree(project.buildDir)
      .include('jacoco/*.exec','outputs/unit_test_code_coverage/*.exec')
  }
}
"@ | Set-Content -Path $initFile -Encoding UTF8

          cmd /c "gradlew.bat clean test --no-daemon --init-script `"$initFile`""
        '''
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
              echo "JaCoCo HTML report not found at ${reportPath}"
            }
          }
        }
      }
    }

    stage('Generate Artifact') {
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
