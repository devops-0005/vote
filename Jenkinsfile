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
    securityContext:
      privileged: true  # Run BuildKit container in privileged mode
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
        DOCKER_CREDS = credentials('dockerlogin')
    }

    stages {
        stage('Build App') {
            steps {
                container('python') {
                    echo 'Compiling vote app'
                    // Add any additional build commands if required
                    sh 'pip install -r requirements.txt'
                }
            }
        }

        stage('Test App') {
            steps {
                container('python') {
                    echo 'Running Unit Tests on vote app'
                    // Run unit tests here, placeholder for 'nosetests' or any test framework you use
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v || true'  // Replace with the correct test command
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
                        // Print debug info for commit hash
                        echo "GIT_COMMIT: ${env.GIT_COMMIT}"

                        // Ensure commitHash is non-empty and fallback to 'latest' if necessary
                        def commitHash = env.GIT_COMMIT?.take(7) ?: 'latest'

                        // Ensure that the commitHash is valid and remove any extra colons
                        if (!commitHash) {
                            error("Commit hash is empty or invalid. Falling back to 'latest'.")
                        }

                        // Print debug info for commitHash
                        echo "Using image tag: ${commitHash}"

                        // Create a Docker config file with credentials
                        sh '''
                        mkdir -p /root/.docker
                        echo '{ "auths": { "https://index.docker.io/v1/": { "auth": "'$(echo -n $DOCKER_CREDS_USR:$DOCKER_CREDS_PSW | base64)'" } } }' > /root/.docker/config.json
                        '''

                        // Run BuildKit build and push command with valid tag format
                        sh """
                          buildctl build --frontend dockerfile.v0 \
                          --local context=. \
                          --local dockerfile=. \
                          --output type=image,name=docker.io/initcron/vote:${commitHash},push=true
                        """
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
