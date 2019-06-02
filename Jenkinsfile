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

    stage('build vote docker image') {
        voteImage = docker.build("roitev/vote", "./src/vote") 
    }
    stage('test vote image') {
        voteImage.inside {
            sh 'echo vote container'
        }
    }
    stage('Push vote image') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            voteImage.push("${env.BUILD_NUMBER}")
            voteImage.push("latest")
        }
    }

    stage('build worker docker image') {
        workerImage = docker.build("roitev/worker", "./src/worker") 
    }
    stage('test worker image') {
        workerImage.inside {
            sh 'echo worker container'
        }
    }
    stage('Push worker image') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            workerImage.push("${env.BUILD_NUMBER}")
            workerImage.push("latest")
        }
    }


}