pipeline {
    agent {  label params.JENKINS_AGENT_LABEL }
    environment {
        def BUILD_DATE =new Date().format("dd-MMM")
        def IMAGE_VERSION = "IMAGE_VERSION=${BULID_DATE}-${BUILD_NUMBER}" 
      }
    
    stages {
        
         stage('Defining environemnt speific variables') {
            steps{
                script{
                        /*withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'acr-credentials', usernameVariable: 'ACR_USERNAME', passwordVariable: 'ACR_PASSWORD']]) {*/
                    withCredentials([
                            [$class: 'UsernamePasswordMultiBinding', credentialsId: 'ovr_credentials', usernameVariable: 'OVR_USERNAME', passwordVariable: 'OVR_PASSWORD'],
                                    ])
                        {   
              
                    // if (params.ENV_TYPE == 'production') {
                        OVR_REGISTRY = "9h4f263p.c1.va1.container-registry.ovh.us"
                        REGISTRY_REPO = "maigha/qrclient"
                        //RM_SLACK_CHANNEL = "ft-svc-deployments"
                        //RM_SLACK_TOKEN = "08db7e7a-e27d-4760-8295-06fb022bfe05"
                        DOCKER_IMAGE_TAG = "${BUILD_DATE}-${BUILD_NUMBER}"
                        DOCKER_COMMAND = "docker login ${OVR_REGISTRY} --username ${OVR_USERNAME} --password ${OVR_PASSWORD}"
                        //TAG = "${ACR_REGISTRY}/${REGISTRY_REPO}:${DOCKER_IMAGE_TAG}"
                        TAG = "${DOCKER_IMAGE_TAG}"
                    // } 
                    commitId = sh(script: "git rev-parse --short HEAD",returnStdout: true)
                    commitMsg = sh(script: """git rev-list --format=%B --max-count=1 ${commitId}""",returnStdout: true) 
                }
            }
        }}
     

        stage('Checkout') {
            steps {
                echo "Checking out $BRANCH_NAME"
                /*checkout([$class: 'GitSCM', branches: [[name: '${BRANCH_NAME}']], extensions: [], userRemoteConfigs: [[credentialsId: 'svc_acct_github', url: 'git@github.com:reshamandi/reshamandi-backend-platform.git']]])*/
                checkout([$class: 'GitSCM', branches: [[name: '${BRANCH_NAME}']], extensions: [], userRemoteConfigs: [[credentialsId: 'svc_acct_github',url: 'https://github.com/AppAnySite/QRClient.git']]])
                echo "Checkout is completed" 
            }
        }
        
       
        stage ('Docker Build and Push Image to OVR'){
            steps{
                parallel(
                    maighaserver: {
                        script{
                        if(params.BUILD_FLAG){
                        echo "Building docker image for Maigha Backend"
                        sh """
                            docker version
                            docker build --no-cache --tag ${OVR_REGISTRY}/${REGISTRY_REPO}:${DOCKER_IMAGE_TAG} .
                            echo "Executing Docker Commands"
                            set +x; ${DOCKER_COMMAND}&> /dev/null
                            docker push ${OVR_REGISTRY}/${REGISTRY_REPO}:${DOCKER_IMAGE_TAG}
                            
                            """ 
                        echo "Building docker image for Maigha Backend is completed"
                        }
                     }
                  } 
                ) 
            }
        }
      
        stage("Deploy Maigha Server") {
            steps{
                script{
                if(params.DEPLOY_FLAG){ 
                    echo "Triggering Spinnaker Pipeline for Deployment "
                     kubernetesDeploy(configs: "deployment.yaml", "service.yaml")
                    echo "Please Check Spinnaker Pipeline for Deployment Progress"
                    }
                }
            }
        } 
        
        // stage("Deploy Maigha Server") {
        //     steps{
        //         script{
        //         if(params.DEPLOY_FLAG){ 
        //             echo "Triggering Spinnaker Pipeline for Deployment "
        //              sh """
        //                     echo 'TAG=${TAG}' > build.properties
        //                 """  
        //             echo "Please Check Spinnaker Pipeline for Deployment Progress"
        //                }
        //         }
        //     }
        // }    
        
        }
    post { 
           always { 
               emailext body: '''$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS:
               Check console output at $BUILD_URL to view the results.''', subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!', to: '${BUILD_USER_EMAIL}'
               archiveArtifacts artifacts: 'build.properties', onlyIfSuccessful: true
            }
    }   
}
