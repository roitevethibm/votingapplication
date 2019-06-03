def helmPackage (path) {
    echo "Packaging ${path}"

    script {
        sh "helm package ${path}"
    }
}

def uploadChart (file) {
    echo "uploading chart to chartmuseum ${file}"

    script {
        sh 'curl --data-binary "@' + file + '" http://9.98.171.136:30552/api/charts'
    }
}

def helmInstall (chart, release) {
    echo "Installing ${chart} release ${release}"

    script {
        sh """
            helm upgrade --install ${release} ${chart} --tls
        """
        sh "sleep 5"
    }
}



node {
    def resultImage
    def voteImage
    def workerImage

    stage('Clone repository') {
        checkout scm
    }
    stage('build docker images') {
        resultImage = docker.build("roitev/result", "./src/result")
        voteImage = docker.build("roitev/vote", "./src/vote")
        workerImage = docker.build("roitev/worker", "./src/worker")

    }
    stage('test docker images') {
        resultImage.inside {
            sh 'echo result container'
        }

        voteImage.inside {
            sh 'echo vote container'
        }

        workerImage.inside {
            sh 'echo worker container'
        }
    }

    stage('Push docker images') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            resultImage.push("${env.BUILD_NUMBER}")
            resultImage.push("latest")

            voteImage.push("${env.BUILD_NUMBER}")
            voteImage.push("latest")

            workerImage.push("${env.BUILD_NUMBER}")
            workerImage.push("latest")


        }
    }

    stage('cloudctl login') {
        script {
        sh "cloudctl login -a https://9.98.171.136:8443 --skip-ssl-validation -u admin -p admin -n voting"
        }
    }

    stage('helm init') {
        script {
        sh "helm init --client-only --skip-refresh"
        }
    }
    
    stage('tailor helm charts') {
        script {
        sh "sed -i -- 's/BUILD_VERSION/${env.BUILD_NUMBER}/g' ./helm_charts/*/values.yaml"
        }
    }
    stage('helm package charts') {
        helmPackage('helm_charts/db')
        helmPackage('helm_charts/redis')
        helmPackage('helm_charts/result')
        helmPackage('helm_charts/vote')
        helmPackage('helm_charts/worker')
        helmPackage('helm_charts/votingapplication')

    }
    stage('upload helm charts to chartmuseum') {
        uploadChart('db-0.1.0.tgz')
        uploadChart('redis-0.1.0.tgz')
        uploadChart('result-0.1.0.tgz')
        uploadChart('vote-0.1.0.tgz')
        uploadChart('worker-0.1.0.tgz')
    }

    stage('helm install votingapplication') {
        helmInstall('helm_charts/votingapplication', 'votingapp')
    }

}