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
}