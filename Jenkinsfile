pipeline {
    stages{
        stage('check env') {
            parallel {
                stage('check mvn') {
                steps {
                    sh 'mvn -v'
                }
                }
                stage('check java') {
                steps {
                    sh 'java -version'
                }
                }
            }
        }
   
    dir('my-elr-backend') {
        stage("Compilation and Analysis") {
            parallel 'Compilation': {
                sh "mvn clean install -DskipTests"
            }, 'Static Analysis': {
                stage("Checkstyle") {
                    sh "mvn checkstyle:checkstyle"
                     
                    step([$class: 'CheckStylePublisher',
                      canRunOnFailed: true,
                      defaultEncoding: '',
                      healthy: '100',
                      pattern: '**/target/checkstyle-result.xml',
                      unHealthy: '90',
                      useStableBuildAsReference: true
                    ])
                }
            }
        }
         
        stage("Tests and Deployment") {
            parallel 'Unit tests': {
                stage("Runing unit tests") {
                    try {
                        sh "mvn test -Punit"
                    } catch(err) {
                        step([$class: 'JUnitResultArchiver', testResults: 
                          '**/target/surefire-reports/TEST-*UnitTest.xml'])
                        throw err
                    }
                   step([$class: 'JUnitResultArchiver', testResults: 
                     '**/target/surefire-reports/TEST-*UnitTest.xml'])
                }
            }, 'Integration tests': {
                stage("Runing integration tests") {
                    try {
                        sh "mvn test -Pintegration"
                    } catch(err) {
                        step([$class: 'JUnitResultArchiver', testResults: 
                          '**/target/surefire-reports/TEST-'
                            + '*IntegrationTest.xml'])
                        throw err
                    }
                    step([$class: 'JUnitResultArchiver', testResults: 
                      '**/target/surefire-reports/TEST-'
                        + '*IntegrationTest.xml'])
                }
            }
             
            stage("Staging") {
                sh "pid=\$(lsof -i:8989 -t); kill -TERM \$pid "
                  + "|| kill -KILL \$pid"
                withEnv(['JENKINS_NODE_COOKIE=dontkill']) {
                    sh 'nohup mvn spring-boot:run -Dserver.port=8989 &'
                }   
            }
        }
}    }
}
