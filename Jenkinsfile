pipeline {
    agent {
        kubernetes {
            label 'python-docker-agent'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: python
    image: python:alpine3.17
    command:
    - cat
    tty: true
    securityContext:
      runAsUser: 0  # Run container as root
  - name: docker
    image: docker:20.10-dind
    securityContext:
      privileged: true  # Docker-in-Docker requires privileged mode
    volumeMounts:
      - name: docker-sock
        mountPath: /var/run/docker.sock
  volumes:
  - name: docker-sock
    emptyDir: {}
"""
        }
    }

    stages {
        stage('Build') {
            steps {
                container('python') {
                    echo 'Compiling vote app'
                    sh 'pip install -r requirements.txt'
                }
            }
        }

        stage('Unit Test') {
            steps {
                container('python') {
                    echo 'Running Unit Tests on vote app'
                    sh 'pip install -r requirements.txt'
                    echo 'Placeholder to run nosetests -v'
                }
            }
        }

        stage('Docker Build and Push') {
            when {
                branch "main"
            }
            steps {
                container('docker') {
                    script {
                        def commitHash = env.GIT_COMMIT.take(7)
                        docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                            def dockerImage = docker.build("initcron/vote:${commitHash}", "./")
                            dockerImage.push()
                            dockerImage.push("latest")
                            dockerImage.push("dev")
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline for vote is complete.'
        }
    }
}
