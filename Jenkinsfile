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

                dependencyCheck additionalArguments: '--scan ./',
                odcInstallation: 'DP-Check'

                dependencyCheckPublisher pattern:
                '**/dependency-check-report.xml'

            }
        }

        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt || true'
            }
        }

        stage('Docker Build') {
            steps {

                sh '''
                docker build \
                --build-arg TMDB_V3_API_KEY=$TMDB_V3_API_KEY \
                -t netflix .
                '''

            }
        }

        stage('Docker Push') {
            steps {

                script {

                    docker.withRegistry(
                    '',
                    'docker'
                    )

                    {

                        sh '''
                        docker tag netflix \
                        22i0832/netflix:latest

                        docker push \
                        22i0832/netflix:latest
                        '''

                    }

                }

            }
        }

        stage('Trivy Image Scan') {
            steps {

                sh '''
                trivy image \
                22i0832/netflix:latest \
                > trivyimage.txt || true
                '''

            }
        }

        stage('Deploy Container') {
            steps {

                sh '''
                docker rm -f netflix-app || true

                docker run -d \
                --name netflix-app \
                -p 8081:80 \
                22i0832/netflix:latest
                '''

            }
        }

    }
}
