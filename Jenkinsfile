pipeline {
    agent {
        docker {
            image 'maven:3-alpine'
            args '-v /root/.m2:/root/.m2'
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
                    dir ('test') {
                        sh 'mvn -B -DskipTests clean package'
                    }
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
                    mvn "install -Dmaven.test.failure.ignore=true"
                } catch (e) {
                    currentBuild.result = 'FAILURE'
                  throw e
                }

              }

              //step([$class: 'JUnitResultArchiver', testResults: 'target/surefire-reports/*.xml'])
              //parallel (
              //  "Firefox" : {
                //  junit 'target/surefire-reports/*.xml'
              //  },
              //  "Chrome" : {
              //    junit 'target/surefire-reports/*.xml'
                }
              //  )

            post {
                always {
                        //junit 'target/surefire-reports/*.xml'
                        script {
                            try {
                              step([$class: 'JUnitResultArchiver', testResults: 'target/surefire-reports/*.xml'])
                              step([$class: 'XUnitBuilder', thresholds: [[$class: 'FailedThreshold',
                              unstableThreshold: '1']],tools: [[$class: 'JUnitType', pattern: '/target/surefire-reports/**']]])
                            } catch (e) {
                                currentBuild.result = 'FAILED'
                              throw e
                            } finally {
                              emailext attachLog: true, body: 'Unit Test has failed ', subject: 'FAILED', to: 'sprasad.tech812@gmail.com'
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
        /*  stage('Deploy to Tomcat') {
                steps {
                  script {
                        echo "Details of the ${env.tomcat_hostname}"
                      //  sshagent(['7c315e58-66b9-47bd-a49d-dc7c2cae1d98']) {
                        //sh 'cp ${WORKSPACE}/target/my-app-1.0-SNAPSHOT.jar http://ec2-54-159-172-184.compute-1.amazonaws.com:8080/'
                        //deploy(war: ${WORKSPACE}/target/my-app-1.0-SNAPSHOT.jar, url: 'http://ec2-54-159-172-184.compute-1.amazonaws.com:8080',
                      //    path: '/opt/tomcat/webapps', username: 'admin', password: 'admin')
                        //curl --upload-file '${WORKSPACE}/target/my-app-1.0-SNAPSHOT.jar' http://admin:admin@ec2-54-159-172-184.compute-1.amazonaws.com:8080/manager/text/deploy?path=/opt/tomcat/webapps&update=true

                  //}
                                                   }
                      }
                                    }*/
}
}
