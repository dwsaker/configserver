pipeline {
    agent any
    environment {
        PROJECT_ID = 'sensor-project-334918'
        CLUSTER_NAME = 'sensor-project-dev-cluster'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'gke-service-account'
        mvnHome = tool 'M2_HOME'
        REGISTRY = "us-central1-docker.pkg.dev"
        PREFIX = "us-central1-docker.pkg.dev/sensor-project-334918/ostock"
        ARTIFACT_ID = "configserver"
        VERSION = "latest"
    }
    stages  {
        stage('Preparation') {
            steps {
                git 'https://github.com/dwsaker/configserver.git'
            }
        }
        stage('Build') {
            steps {
                script {
                // Run the maven build
                    if (isUnix()) {
                        sh "'${env.mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
                    } else {
                        bat(/"%MVN_HOME%\bin\mvn" -Dmaven.test.failure.ignore clean package/)
                    }
                }
            }
        }
        stage('Results') {
            steps {
                junit '**/target/surefire-reports/TEST-*.xml'
                archiveArtifacts 'configserver/target/*.jar'
            }
        }
        stage('Build image') {
            steps {
                sh "'${env.mvnHome}/bin/mvn' -Ddocker.image.prefix=${env.PREFIX} -Dproject.artifactId=${env.ARTIFACTS_ID} -Ddocker.image.version=${env.VERSION} dockerfile:build"
            }
        }
        stage('Push image') {
            steps {
                withCredentials([file(credentialsId: 'google-service-account', variable: 'GC_KEY')]) {
                    sh('gcloud auth activate-service-account --key-file=${GC_KEY}')
                    sh("gcloud auth configure-docker ${env.REGISTRY}")
                    sh("docker push ${env.PREFIX}/${env.ARTIFACT_ID}:${env.VERSION}")
                }
            }
        }
        stage('Deploy to GKE') {
            steps{
                //sh "sed -i 's/hello:latest/hello:${env.BUILD_ID}/g' deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'configserver-deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }
    }
}