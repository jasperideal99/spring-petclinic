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

    stage('JaCoCo Code Coverage') {
      steps {
        bat '''
          @echo off
          set INIT_FILE=%WORKSPACE%\\jenkins-jacoco.gradle

          (
            echo allprojects ^{
            echo   apply plugin: 'jacoco'
            echo   tasks.withType(Test).configureEach ^{
            echo     finalizedBy 'jacocoTestReport'
            echo   ^}
            echo   tasks.register('jacocoTestReport', JacocoReport) ^{
            echo     dependsOn tasks.withType(Test)
            echo     reports ^{
            echo       xml.required = true
            echo       html.required = true
            echo     ^}
            echo     def mainSourceSets = project.hasProperty('sourceSets') ? sourceSets : null
            echo     if (mainSourceSets != null) ^{
            echo       sourceDirectories.setFrom files(mainSourceSets.main.allSource.srcDirs)
            echo       classDirectories.setFrom files(mainSourceSets.main.output)
            echo     ^}
            echo     executionData.setFrom fileTree(project.buildDir).include("jacoco/*.exec","outputs/unit_test_code_coverage/*.exec")
            echo   ^}
            echo ^}
          ) > "%INIT_FILE%"

          gradlew.bat clean test --no-daemon --init-script "%INIT_FILE%"
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
