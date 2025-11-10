pipeline {
    agent any
    environment {
        REGISTRY = "localhost:8082"
        IMAGE = "java-maven-app"
        SONARQUBE = "sonarqube-server"
    }
    stages {
        stage('Checkout'){steps{git branch: 'master', url: 'https://github.com/rachanonsomtha/simple-java-maven-app.git'}}
        stage('Build'){steps{sh 'mvn clean package -DskipTests=false'}}
        stage('Unit Tests'){steps{sh 'mvn test'}}
        stage('Static Code Analysis'){
            steps {
                withSonarQubeEnv(
                    installationName: "${SONARQUBE}",
                    credentialsId: "sonar-analysis-token"
                ) { sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    // Generate release tag (e.g., release_20251109)
                    DATE_STAMP = sh(script: "date +%Y%m%d", returnStdout: true).trim()
                    RELEASE_TAG = "release_${DATE_STAMP}"
                    echo "Generated Release Tag: ${RELEASE_TAG}"

                    // Build the image using the multi-stage Dockerfile
                    // Ensure docker/Dockerfile is fixed and available
                    sh "docker build -f docker/Dockerfile -t ${REGISTRY}/${IMAGE}:${RELEASE_TAG} ."
                }
            }
        }
        stage('Docker Push to Nexus') {
            steps {
                script {
                    // 1. Securely retrieve Nexus credentials
                    withCredentials([usernamePassword(
                        credentialsId: 'nexus-docker-cred', 
                        passwordVariable: 'user1', 
                        usernameVariable: 'user1'
                    )]) {
                        echo "Logging into Nexus registry: ${REGISTRY}"
                        
                        // 2. Log in using the injected credentials
                        // The $REGISTRY is required because Nexus may use a different path than Docker Hub
                        sh "docker login ${REGISTRY} -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                        
                        // 3. Push the image using the generated tag
                        echo "Pushing image: ${REGISTRY}/${IMAGE}:${RELEASE_TAG}"
                        sh "docker push ${REGISTRY}/${IMAGE}:${RELEASE_TAG}"
                    }
                }
            }
        }     
        stage('Output Release Tag'){
            steps{
                echo "RELEASE TAG: ${RELEASE_TAG}"
            }
        }
    }
}
