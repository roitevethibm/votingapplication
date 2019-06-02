node {
    checkout scm
    def resultImage = docker.build("roitev/result", "./src/result") 

    resultImage.inside {
        sh 'echo result container'
    }
}