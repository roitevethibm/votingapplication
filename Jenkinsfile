node {
    def resultImage
    stage('Clone repository') {
        checkout scm
    }
    stage('build result docker image') {
        resultImage = docker.build("roitev/result", "./src/result") 
    }
    stage('test result image') {
        resultImage.inside {
            sh 'echo result container'
        }
    }
    stage('Push result image') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            resultImage.push("${env.BUILD_NUMBER}")
            resultImage.push("latest")
        }
    }
}