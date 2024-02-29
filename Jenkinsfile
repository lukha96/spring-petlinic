pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_S3_BUCKET = 'bucketjfrog'
    }

    stages {
        stage('snyk scan') {
            steps {
                snykSecurity(
                    snykInstallation: 'Snyk',
                    snykTokenId: 'Snyk',
                    monitorProjectOnBuild: false,
                    failOnIssues: false,
                    additionalArguments: '--json-file-output=all-vulnerabilities.json'
                )
            }
        }

        stage('maven build artifact') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage('building a docker image') {
            steps {
               sh "docker build -t spring-clinic/petapp:${BUILD_NUMBER} ."

            }
        }

        stage("TRIVY") {
            steps {
                sh "trivy image  spring-clinic/petapp:${BUILD_NUMBER} --scanners vuln > trivyimage.txt"
            }
        }

        stage('docker image push') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/']) {
                sh "docker push spring-clinic/petapp:${BUILD_NUMBER}"
                }
            }
        }

stage('Push to AWS S3') {
    steps {
        script {
            def jarFile = 'target/spring-petclinic-3.2.0-SNAPSHOT.jar'

            sh "aws s3 cp ${jarFile} s3://bucketjfrog/"
               }
             }
         }
      }
   }
