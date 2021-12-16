node {
    def mvnHome
    stage('Preparation') {
        git 'https://github.com/dwsaker/configserver.git'
        mvnHome = tool 'M2_HOME'
    }
    stage('Build') {
        // Run the maven build
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
            } else {
                bat(/"%MVN_HOME%\bin\mvn" -Dmaven.test.failure.ignore clean package/)
            }
        }
    }
    stage('Results') {
        junit '**/target/surefire-reports/TEST-*.xml'
        archiveArtifacts 'configserver/target/*.jar'
    }
    stage('Build image') {
        sh "'${mvnHome}/bin/mvn' -Ddocker.image.prefix=us-central1-docker.pkg.dev/sensor-project-334918/ostock -Dproject.artifactId=configserver -Ddocker.image.version=latest dockerfile:build"
    }
    stage('Push image') {
        sh "docker push us-central1-docker.pkg.dev/sensor-project-334918/ostock/configserver:latest"
    }
    stage('Kubernetes deploy') {
        sh "kubectl apply -f configserver-service.yaml,configserver-deployment.yaml"
    }
}