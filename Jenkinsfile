pipeline {
    agent { // slave jenkins
        label 'maven'
    }
    /**
     * git clone
     * maven compilação (package) -> jar/war
     * maven test 
     * startBuild -> (jar) --> is
     * newApp
     **/
     
    stages {
        stage('Git Clone') {
            steps {
                git '${APP_GIT_URL}'
                sh 'ls -lah'
            }
        }
        
        stage('Maven Package') {
            steps {
                sh 'mvn clean package -DskipTests=true -f ${APP_CONTEXT_DIR}/pom.xml'
                sh 'ls -lah ${APP_CONTEXT_DIR}/target'
            }
        }
        
        stage('Maven Test') {
            steps {
                sh 'mvn test -f ${APP_CONTEXT_DIR}/pom.xml'
            }
        }
        
        stage('startBuild') {
            steps {
                // apagar a imagem
                sh  '''
                    oc -n ${APP_NOME_PROJETO} delete is/${APP_NAME} --ignore-not-found=true
                '''
                // apagar o build
                sh  '''
                    oc -n ${APP_NOME_PROJETO} delete bc/${APP_NAME} --ignore-not-found=true
                '''
                
                sh '''
                    oc -n ${APP_NOME_PROJETO} \
                        new-build \
                        --name=${APP_NAME} \
                        --image-stream=openshift/jboss-eap71-openshift:latest \
                        --binary --labels="app=${APP_NAME}"
                '''
                
                sh  '''
                    oc -n ${APP_NOME_PROJETO} start-build ${APP_NAME} --from-dir=${APP_CONTEXT_DIR}/target --wait -F
                '''
            }
        }
        
        stage('newApp') {
            steps {
                sh 'oc -n ${APP_NOME_PROJETO} new-app ${APP_NAME}'
                sh 'oc -n ${APP_NOME_PROJETO} expose svc/${APP_NAME}'
            }
        }
    }
    
    
}
