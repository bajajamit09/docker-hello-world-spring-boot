pipeline {
  agent any 
     environment {
          def regUrl = "k8workshopregistry.azurecr.io"
          def appImage = "spring-demo-api";
          def apiImage = "angular-ui"
          def dockerRepo = "angular-ui"
          def latestTag = "latest";
          buildNumber = "${env.BUILD_ID}"
          branchName = "${env.GIT_BRANCH}"
          def buildTag = "build-${BUILD_NUMBER}";
          def releaseTag = "qa";
          def pullSecret = "acr-secret"
          def environment = "dev"
          def namespace = "jenkins"
          def acr = "k8workshopregistry"
          def AKS_SRVC_USER = "e544388b-8114-4c6b-bf63-622229700801"
          def AKS_SRVC_PASSWORD = ""
          def TENANT_ID = "5f9d8183-ac49-417b-95c3-f12d0b218595"
          def RESOURCE_GROUP = "RSG-AKSDemo"
	  def CLUSTER_NAME = "DemoMicroservices"
     }
      stages {
        stage('checkout'){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/bajajamit09/docker-hello-world-spring-boot.git']]])
            }
        }
    stages {
        stage('Get a Maven project') {
           steps {
            sh 'mvn -Dmaven.test.failure.ignore clean package'
            } 
          }
      
 
            stage('Build Docker Image') {
                steps {
                sh """ 
                echo "Build tag is ${buildTag} "
                docker build -t ${regUrl}/${appImage}:${buildNumber} . 
                docker build -t ${regUrl}/${apiImage}:${buildNumber}  ${dockerRepo}/
                docker push $regUrl/$appImage:${buildNumber}
                docker push $regUrl/$apiImage:${buildNumber}
                """
                    }
            }
	    /*  stage('Vulnerability Scan w/Twistlock') {
		      steps {
                twistlock.scanImage("k8workshopregistry.azurecr.io/hello-world-java:latest")
    }
	      }	      

        
        stage('Scan') {
            steps {
                // Scan the image
                prismaCloudScanImage ca: '',
                cert: '',
                dockerAddress: 'unix:///var/run/docker.sock',
                image: 'k8workshopregistry.azurecr.io/hello-world-java:latest',
                key: '',
                logLevel: 'info',
                podmanPath: '',
                project: '',
                resultsFile: 'prisma-cloud-scan-results.json',
                ignoreImageBuildTime:true
            }
        }*/
           stage("Deploy Spring Boot Demo API") {
              steps {
          //  def ticketId = mozart.openAksRfc(buildProdMozartRequest())
          //  withCredentials([prodAzureSecretRepo]) {
              sh "ls -l"
              sh "bin/spring-demo-api-deployment.sh " +
                "${pullSecret} " + //repo
                "${environment} " + //environment
                "${namespace} " + //namespace
              //  "${IMAGE_NAME} " + //image name
                "${env.BUILD_ID} " + //image version
             //   "${DOCKER_REPO} " + //docker repo
                "${acr} " + //azure registry
                "1" // replica count
               }
            }
            stage("Authenticate Service Account") {
              steps {
	       // withCredentials([azureServiceAccount, azureTenantId, devSixClusterName, resourceGroup]) {
	       //   sh 'chmod -R 777 ./bin/aks'
        	  sh "./bin/authenticate-az-service-account.sh " +
	            "${env.AKS_SRVC_USER} " +
        	    "${env.AKS_SRVC_PASSWORD} " +
	            "${env.TENANT_ID} " +
        	    "${RESOURCE_GROUP} " +
	            "${CLUSTER_NAME}"
		        }
	      }
           stage("Deploy Angular-UI") {
              steps {
          //  def ticketId = mozart.openAksRfc(buildProdMozartRequest())
          //  withCredentials([prodAzureSecretRepo]) {
              sh "./bin/angular-ui.sh " +
                "${pullSecret} " + //repo
                "${environment} " + //environment
                "${namespace} " + //namespace
              //  "${IMAGE_NAME} " + //image name
                "${env.BUILD_ID} " + //image version
             //   "${DOCKER_REPO} " + //docker repo
                "${acr} " + //azure registry
                "1" // replica count
               }
	   }
        	 stage("Unauthenticate Service Account") {
	           steps {
		        sh "./bin/unauthenticate-service-account.sh"
      			}
	}
    }
 }

