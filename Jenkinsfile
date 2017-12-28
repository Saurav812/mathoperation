pipeline {
    agent {
        node {
           label 'master'
        }
    }

    stages {
        stage('Build') {
            steps {
                //sh 'mvn -B -DskipTests clean package'
                script {
                try {
                    // run tests in the same workspace that the project was built
                    echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
                        sh 'mvn -B -DskipTests clean package'

                } catch (e) {
                    // if any exception occurs, mark the build as failed
                    currentBuild.result = 'FAILURE'
                    throw e
                } finally {
                    // perform workspace cleanup only if the build have passed
                    // if the build has failed, the workspace will be kept
                    //cleanWs cleanWhenFailure: false
                    emailext attachLog: true, body: 'Build has failed ', subject: 'FAILED', to: 'sprasad.tech812@gmail.com'
                }
        }
      }
		}

        stage('Test') {
            steps {
              script{
                  try {
                    echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
                  sh "mvn install -Dmaven.test.failure.ignore=true"
                } catch (e) {
                    currentBuild.result = 'FAILURE'
                  throw e
                }

              }
            }
            post {
                always {
                        //junit 'target/surefire-reports/*.xml'
                        script {
                            try {
                              step([$class: 'JUnitResultArchiver', testResults: 'target/surefire-reports/*.xml'])
                              step([$class: 'XUnitBuilder', thresholds: [[$class: 'FailedThreshold',
                              unstableThreshold: '1']],tools: [[$class: 'JUnitType', pattern: '/target/surefire-reports/**']]])
                            } catch (e) {
                                currentBuild.result = 'SUCCESS'
                              throw e
                            } finally {
                              emailext attachLog: true, body: 'Unit Test has passed ', subject: 'SUCCESS', to: 'sprasad.tech812@gmail.com'
                            }
                        }
                        publishHTML([
                          allowMissing: true,
                          alwaysLinkToLastBuild: true,
                          keepAll: true, reportDir: 'reports/',
                          reportFiles: '*.xml',
                          reportName: 'Coverage Report',
                          reportTitles: '']
                                    )

                        }
                  }

          }


          stage('JaCoCo Code Coverage') {
              steps {
                  script {
                    try {
                          //sh "${mvnHome}/bin/mvn -B -f ./product-api -Dspring.profiles.active=integration -Dmaven.test.failure.ignore=true prepare-package"
                          jacoco(classPattern: './product-api/target/**/classes', execPattern: './product-api/target/jacoco.exec', sourcePattern: './product-api/src/main/java')
                          // TODO_1st : if code coverage  is below threshold stop build & fail , is there a way show these numbers on UI  or record and send at end of email
                    } catch (e) {
                          currentBuild.result = 'FAILED'
                      throw e
                    } finally {
                          // Email triggered if the build is failed
                          emailext attachLog: true, body: 'Unit Test has failed ', subject: 'FAILED', to: 'sprasad.tech812@gmail.com'
                              }

                          }

                      }
                    }


}
}
