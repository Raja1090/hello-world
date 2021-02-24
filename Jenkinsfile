node {
    agent any
    environment {
       JENKINS_NODE_COOKIE = "dontKillMe /usr/bin/dotnet"
   }

    stages {
        stage('CleanWorkspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout SCM') {
            steps {
                git branch: '${BRANCH_NAME}', credentialsId: 'TFS-Security', url: 'https://tfs.ercbpo.local/tfs/ERC/ERC%20New/_git/core-unity-web'
            }
        }
        
        stage('env varaible') {
            steps {
               sh "printenv | sort"
               sh "echo $JENKINS_NODE_COOKIE"
            }
        }
        
        stage("Build Dotnet Package") {
            steps {
                script {
                  dir ("$WORKSPACE") {
                     sh '''
                        rm -rf package-lock.json node_modules/
                        npm install
                        npm run ng build
                        
                     '''
                 }
                }
            }
            
        }
        
        
        stage("Build Docker image") {
            steps {
                script {
                  dir ("/var/lib/jenkins/workspace/core-unity-web-build") {
                     cicd_docker_app = docker.build("10.215.97.155:5000/${env.APP_NAME}:${env.Docker_Version}.${env.BUILD_NUMBER}")
                 }
                }
            }
            
        }
        stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://10.215.97.155:5000') {
                            cicd_docker_app.push("${env.Docker_Version}.${env.BUILD_NUMBER}")
                            
                    }
                }
            }
        }
      stage("Deploy to kubernetes Test Cluster") {
            steps {
                script {
                    dir ("$WORKSPACE") {
                     withKubeConfig([credentialsId: 'k8s-testcluster-token', serverUrl: 'https://10.215.97.102:6443']) {
                         
      sh "sed -i 's/v1.0.0/${Docker_Version}.${BUILD_NUMBER}/g' unity-*.yml"
      sh 'cat unity-*.yml'
      sh 'kubectl create --dry-run=true -o unity-*.yml | kubectl apply -f unity-*.yml  -n ${Namespace}'
      //echo 'waiting 15 seconds for deployment to complete prior checking'
      //sleep 15 // seconds
      //sh 'kubectl get all -n ${Namespace} | grep ${APP_NAME}'
    }

}   
            }
            }
            
        }
    
        
    }
}
