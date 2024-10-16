pipeline {
    agent {
        kubernetes {
            label 'python-buildkit-agent'
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
  - name: buildkit
    image: moby/buildkit:latest
    env:
      - name: DOCKER_BUILDKIT
        value: "1"
    volumeMounts:
      - name: buildkit-cache
        mountPath: /var/lib/buildkit
  volumes:
  - name: buildkit-cache
    emptyDir: {}
"""
        }
    }

    environment {
        DOCKER_CREDS = credentials('dockerlogin')  // Retrieve Docker credentials from Jenkins credentials store
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

        stage('Docker Build and Push with BuildKit') {
            when {
                branch "main"
            }
            steps {
                container('buildkit') {
                    script {
                        def commitHash = env.GIT_COMMIT.take(7)

                        // Create a Docker config file with credentials
                        sh '''
                        mkdir -p /root/.docker
                        echo '{ "auths": { "https://index.docker.io/v1/": { "auth": "'$(echo -n $DOCKER_CREDS_USR:$DOCKER_CREDS_PSW | base64)'" } } }' > /root/.docker/config.json
                        '''

                        // Run BuildKit build and push command
                        sh '''
                          buildctl build --frontend dockerfile.v0 \
                          --local context=. \
                          --local dockerfile=. \
                          --output type=image,name=docker.io/initcron/vote:${commitHash},push=true
                        '''
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
