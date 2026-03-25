pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = '22i0832/netflix'
        TMDB_V3_API_KEY = credentials('tmdb-api-key')
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/AyaanKhan1576/Netflix-DevSecOps.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=netflix \
                    -Dsonar.projectKey=netflix \
                    -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install || true'
            }
        }

        stage('OWASP Dependency Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format XML', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt || true'
            }
        }

        stage('Docker Build Image') {
            steps {
                sh '''
                docker build \
                --build-arg TMDB_V3_API_KEY=$TMDB_V3_API_KEY \
                -t netflix .
                '''
            }
        }

        stage('Docker Tag Image') {
            steps {
                sh '''
                docker tag netflix 22i0832/netflix:latest
                '''
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                    docker.withRegistry('', 'docker') {
                        sh 'docker push 22i0832/netflix:latest'
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image 22i0832/netflix:latest \
                > trivyimage.txt || true
                '''
            }
        }

        stage('Remove Old Container') {
            steps {
                sh '''
                docker stop netflix-app || true
                docker rm netflix-app || true
                docker stop netflix || true
                docker rm netflix || true
                '''
            }
        }

        stage('Free Port 8081') {
            steps {
                sh '''
                CONTAINER_ID=$(docker ps -q --filter "publish=8081")
                if [ ! -z "$CONTAINER_ID" ]; then
                    docker rm -f $CONTAINER_ID
                fi
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker run -d \
                --name netflix-app \
                -p 8081:80 \
                22i0832/netflix:latest
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                docker ps
                '''
            }
        }

    }
}
