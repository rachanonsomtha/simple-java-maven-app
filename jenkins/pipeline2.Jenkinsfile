pipeline {
    agent any
    parameters {
        string(name: 'RELEASE_TAG')
        choice(name:'ACTION', choices:['DEPLOY','ROLLBACK'])
    }
    stages {
        stage('Determine Target'){
            steps{
                script{
                    ACTIVE = sh(script:"kubectl get svc java-maven-app-service -o=jsonpath='{.spec.selector.version}'",returnStdout:true).trim()
                    TARGET = (ACTIVE=="blue") ? "green" : "blue"
                    echo "ACTIVE=${ACTIVE}, TARGET=${TARGET}"
                }
            }
        }
        stage('Deploy if DEPLOY'){
            when{expression{params.ACTION=='DEPLOY'}}
            steps{
                sh "sed 's/{{TAG}}/${RELEASE_TAG}/' k8s/deployment-${TARGET}.yaml > k8s/render-${TARGET}.yaml"
                sh "kubectl apply -f k8s/render-${TARGET}.yaml"
                sh "kubectl rollout status deployment/java-maven-app-${TARGET} --timeout=60s"
            }
        }
        stage('Switch Traffic'){
            steps{
                script{
                    NEW_ACTIVE = (params.ACTION=="DEPLOY") ? TARGET : ACTIVE
                    sh "kubectl patch service java-maven-app-service -p '{"spec":{"selector":{"app":"java-maven-app","version":"${NEW_ACTIVE}"}}}'"
                }
            }
        }
    }
}
