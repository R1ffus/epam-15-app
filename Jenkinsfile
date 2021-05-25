def tag = ''
def release = false
def prod = false

pipeline {
  agent {
    kubernetes {
      yamlFile 'agent.yaml'
    }
  }
  environment {
    CREDENTIALS = 'epam-15'
    PROJECT_ID = 'epam-15'
    CLUSTER_NAME = 'c1'
    LOCATION = 'europe-north1-a'
  }

  stages {
    stage('Test') {
      steps {

          script {
	    container('docker') {
	      sh 'printenv'
	      tag = env.TAG_NAME ?: env.BUILD_ID
              release = env.TAG_NAME ? true : false
              docker.withRegistry('https://eu.gcr.io', "gcr:${env.CREDENTIALS}") {
              	def image = docker.build("epam-15/cicd_app:${tag}")
              	  image.push()
		  if(env.TAG_NAME) {
		  	image.push('latest') 
		  }
              } 
	    }
      	 }
      }
   }
   stage ('deploy') {
     steps {
       container('kubectl') {
          sh "sed -i 's/__TAG__/${tag}/g' k8s/manifest.yaml"
          step([
            $class: 'KubernetesEngineBuilder',
            projectId: env.PROJECT_ID,
            clusterName: env.CLUSTER_NAME,
            location: env.LOCATION,
            manifestPattern: 'k8s/manifest.yaml',
            namespace: release ? 'testapp-release' : 'testapp-test',
            credentialsId: env.CREDENTIALS,
            verifyDeployments: true
          ])
       }
      script {
        if (release) {
	  prod = input    message: "deploy to prod?",
                  id: 'prodDeploy',
                  parameters: [booleanParam(name: "reviewed", defaultValue: false,
                  	description: "prod deploy")]
          println prod
        }	
     }
    }
   }
   stage ('deploy prod') {
     when {
                expression { prod == true }
     }
     steps {
       container('kubectl') {
          sh "sed -i 's/__TAG__/${tag}/g' k8s/manifest.yaml"
          step([
            $class: 'KubernetesEngineBuilder',
            projectId: env.PROJECT_ID,
            clusterName: env.CLUSTER_NAME,
            location: env.LOCATION,
            manifestPattern: 'k8s/manifest.yaml',
            namespace: 'testapp-prod',
            credentialsId: env.CREDENTIALS,
            verifyDeployments: true
          ])
       }

     }
   }
 }
}
