/**
 * This declarative pipeline automates the full CI process:
 * 1. Git Checkout
 * 2. Maven Build, Test, and SonarQube Analysis
 * 3. Docker Image Build with dynamic release tagging
 * 4. Push the image to Nexus Registry
 * * NOTE: This pipeline requires the Docker Pipeline Plugin on Jenkins.
 */
pipeline {
    agent any

    // --- Global Configuration Variables ---
    environment {
        // --- Registry & Image ---
        REGISTRY = "localhost:8082"
        IMAGE = "java-maven-app"
        
        // --- Nexus Credentials ID ---
        NEXUS_CREDENTIALS_ID = "nexus-docker-cred" // User's requested credential ID
        
        // --- SonarQube Configuration ---
        SONARQUBE = "sonarqube-server" // SonarQube Server Installation Name in Jenkins
        SONAR_CREDENTIALS_ID = "sonar-analysis-token" // User's requested credential ID
        
        // Placeholder for the dynamically generated tag
        RELEASE_TAG = "" 
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                // User's requested Git repository checkout
                git branch: 'master', url: 'https://github.com/rachanonsomtha/simple-java-maven-app.git'
            }
        }

        stage('2. Build Java Artifact') {
            steps {
                // Compile and package the application
                sh 'mvn clean package -DskipTests=true' 
            }
        }
        
        stage('3. Unit Tests') {
            steps {
                // Execute unit tests
                sh 'mvn test'
            }
        }
        
        stage('4. Static Code Analysis (SonarQube)') {
            steps {
                // Execute SonarQube analysis using the configured environment
                withSonarQubeEnv(
                    installationName: "${SONARQUBE}",
                    credentialsId: "${SONAR_CREDENTIALS_ID}"
                ) { 
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('5. Docker Image Build') {
            steps {
                script {
                    // Generate dynamic release tag (e.g., release_20251109)
                    def DATE_STAMP = sh(script: "date +%Y%m%d", returnStdout: true).trim()
                    env.RELEASE_TAG = "release_${DATE_STAMP}"
                    
                    def FULL_IMAGE_TAG = "${REGISTRY}/${IMAGE}:${env.RELEASE_TAG}"
                    echo "Generated Release Tag: ${env.RELEASE_TAG}"
                    echo "Building Docker image: ${FULL_IMAGE_TAG}"
                    
                    // --- USING JENKINS DOCKER DSL ---
                    // The '-f Dockerfile .' parameter tells Docker to use the Dockerfile in the root context.
                    def customImage = docker.build(FULL_IMAGE_TAG, '-f Dockerfile .')
                    
                    // Store the image object for potential future Groovy steps
                    env.DOCKER_IMAGE = customImage
                }
            }
        }

        stage('6. Docker Push to Nexus') {
            steps {
                script {
                    // 1. Securely retrieve Nexus credentials
                    withCredentials([usernamePassword(
                        credentialsId: "${NEXUS_CREDENTIALS_ID}", 
                        passwordVariable: 'Rachano123', 
                        usernameVariable: 'gladoss'
                    )]) {
                        echo "Logging into Nexus registry: ${REGISTRY}"
                        
                        // 2. Log in using the injected credentials (DOCKER_USER/DOCKER_PASSWORD)
                        sh "docker login ${REGISTRY} -u ${gladoss} -p ${Rachano123}"
                        
                        // 3. Push the image using the generated tag
                        def FULL_TAG = "${REGISTRY}/${IMAGE}:${env.RELEASE_TAG}"
                        echo "Pushing image: ${FULL_TAG}"
                        sh "docker push ${FULL_TAG}"
                        
                        // 4. Log out (best practice)
                        sh "docker logout ${REGISTRY}"
                    }
                }
            }
        }     
        
        stage('7. Output Release Tag'){
            steps{
                echo "RELEASE TAG: ${env.RELEASE_TAG}"
            }
        }
    }
    
    post {
        always {
            cleanWs() // Clean up the workspace
        }
        failure {
            echo 'Pipeline failed. Check previous stages for errors.'
        }
    }
}