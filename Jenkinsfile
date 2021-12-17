node {
    def mvnHome
    def project = "sensor-project-334918"
    def registry = "us-central1-docker.pkg.dev"
    def prefix = "us-central1-docker.pkg.dev/sensor-project-334918/ostock"
    def artifactId = "configserver"
    def version = "latest"
    def REGISTRY_USER = credentials('sensor-project-334918')
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
        sh "'${mvnHome}/bin/mvn' -Ddocker.image.prefix=${prefix} -Dproject.artifactId=${artifactId} -Ddocker.image.version=${version} dockerfile:build"
    }
    stage('Push image') {
        withCredentials([file(credentialsId: 'sensor-project-334918', variable: 'GC_KEY')]) {
            sh("gcloud auth activate-service-account --key-file=${GC_KEY}")
            sh("gcloud auth configure-docker ${registry}")
            sh("docker push ${prefix}/${artifactId}:${version}")
        }
    }
    stage('Kubernetes deploy') {
        sh "kubectl apply -f configserver-service.yaml,configserver-deployment.yaml"
    }
}