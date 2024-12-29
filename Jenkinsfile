pipeline {

    agent any
    tools {
        maven "MAVEN_HOME"
        jdk "OracleJDK11"
    }
    environment {
        registry = "abhi41/vproappdock"
        registryCredential = 'dockerhub'
    }

    stages{

        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

       /*  stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        } */

        /* stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
 */
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

       /*  stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'mysonarscanner4'
            }

            steps {
                 withSonarQubeEnv('sonar-pro') {
                            sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                                -Dsonar.projectName=vprofile-repo \
                                -Dsonar.projectVersion=1.0 \
                                -Dsonar.sources=src/ \
                                -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                                -Dsonar.junit.reportsPath=target/surefire-reports/ \
                                -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                                -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml \
                                -Dsonar.host.url=http://18.208.131.124:80'''
                        }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        } */
        //build our docker image
        stage('Build App Image'){
           steps {
              script {
                dockerImage = docker.build registry + ":V${BUILD_NUMBER}"
              }
           }
        }
        //upload docker image to docker hub
        stage('Upload Image'){
           steps{
             script {
               docker.withRegistry('', registryCredential) {
                  //dockerImage.push ("V$BUILD_NUMBER")
                  dockerImage.push ('latest')
               }
             }
           }
        }
        //remove docker image
        stage('Remove Unsued docker image'){
            steps{
            //  sh "docker rmi $registry:V$BUILD_NUMBER"
              sh "docker rmi $registry:latest"
            }
        }
        // next stage is deploying image to kubernetes cluser and we are going to run helm command from kops

        stage('Kubernetes Deploy'){
          agent {label 'KOPS'}
            steps{
              //sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts -set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
              sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts -set appimage=${registry}:latest --namespace prod"
            }
        }




    }


}