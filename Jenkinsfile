pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node20'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        // Inject from Jenkins credentials — never hardcode.
        TMDB_V3_API_KEY = credentials('tmdb-v3-api-key')
    }
    stages {
        stage('clean workspace') {
            steps { cleanWs() }
        }
        stage('Checkout from Git') {
            steps { git branch: 'main', url: 'https://github.com/ofdengiz/filmapp.git' }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                          -Dsonar.projectName=filmapp \
                          -Dsonar.projectKey=filmapp
                    '''
                }
            }
        }
        stage('Quality gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps { sh 'npm install' }
        }
        stage('OWASP FS Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Trivy FS Scan') {
            steps { sh 'trivy fs . > trivyfs.txt' }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker build --build-arg TMDB_V3_API_KEY=$TMDB_V3_API_KEY -t filmapp .'
                        sh 'docker tag filmapp ofdengiz/filmapp:latest'
                        sh 'docker push ofdengiz/filmapp:latest'
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps { sh 'trivy image ofdengiz/filmapp:latest > trivyimage.txt' }
        }
        stage('Deploy to container') {
            steps { sh 'docker run -d -p 8081:80 ofdengiz/filmapp:latest' }
        }
    }
}

// One-time Jenkins host setup (run manually, not part of the pipeline):
//
//   sudo usermod -aG docker jenkins
//   sudo systemctl restart jenkins
