pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes'
            yaml """
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: jnlp
                    image: jenkins/inbound-agent:3107.v665000b_51092-15
                    resources:
                      limits:
                        cpu: '1'
                        memory: '2Gi'
                      requests:
                        cpu: '900m'
                        memory: '1Gi'
                    tty: true
            """
        }
    }

    tools {
        nodejs "node"
    }


    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // stage('Install Dependencies') {
        //     steps {
        //         sh 'npm install'
        //     }
        // }
        stage('Install SBOM tool') {
            steps {
                sh 'npm install -g @cyclonedx/cdxgen'
            }
        }

        stage('Generate SBOM') {
            steps {
                sh 'export FETCH_LICENSE=true && cdxgen -r -o bom.json'
                script {
                    def sbom = readFile('bom.json')
                    echo "Generated SBOM:\n$sbom"
                }
            }
        }

        stage('Upload SBOM to Dependency-Track') {
            steps {
                withCredentials([string(credentialsId: 'apikey', variable: 'X_API_KEY')]) {
                    sh """
                    curl -k -X POST "https://dt-api-jenkins-test.staging.cryptosoft.com/api/v1/bom" \
                    -H "Content-Type:multipart/form-data" \
                    -H "X-Api-Key:${X_API_KEY}" \
                    -F "autoCreate=true" \
                    -F "projectName=Jenkinsnodejs" \
                    -F "projectVersion=1.22" \
                    -F "bom=@bom.json"
                    """
                }
            }
        }
    }
}
