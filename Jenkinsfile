node {
    checkout scm
    def resultImage = docker.build("test-image", "./src/result") 

    resultImage.inside {
        sh 'make test'
    }
}