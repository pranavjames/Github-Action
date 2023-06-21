pipeline {

	agent {
		label 'Linux_Node1'
	      }

	options {
		buildDiscarder(logRotator(numToKeepStr: '5'))
	}

    environment {

      

       DOCKER_REGISTRYY = credentials("hosted_jfrog_ip");
	   DOCKER_REGISTRY = "${DOCKER_REGISTRYY}/docker-pranav/spring"
       DATE = new Date().format('yy.M')
       TAG = "${DATE}.${BUILD_NUMBER}"
    }

	tools {

		maven "Maven"

		jdk "JDK17"

	}


	stages {

		stage('Begin') {
			steps {
				sh 'echo "Helo"'
				sh 'echo "Docker registry here : ${DOCKER_REGISTRY}"'
			}
		}

		stage('Initialize') {
			steps {
				sh "echo 'Initializing'"
			}
		}

		stage('Clean') {
			steps {
				echo "Cleaning"
				sh "mvn clean"
			}
		}

		stage('Install') {
			steps {
				echo "Install"
				sh "mvn install"
			}
		}

		stage('Sonarqube Analysis') {
			steps {

				withSonarQubeEnv('Sonarqube_Server') {
					sh 'mvn sonar:sonar'
				}

				timeout(time: 10, unit: 'MINUTES') {
					waitForQualityGate abortPipeline: true
					echo 'inside sonar environment'
				}

			}

		}

		stage('Deploy to Artifact') {
			steps {
				script {
					def server = Artifactory.server 'Jfrog'
					def uploadSpec = """{
					"files": [{
						"pattern": "target/*.jar",
						"target": "CICD/"
					}]
				}
				"""
				server.upload(uploadSpec)
			}

		}

		post {
			always {
				jiraSendBuildInfo()
			}

		}

	}

    stage('Building Docker Image') {
			steps {



                 sh "docker build -t ${DOCKER_REGISTRY}/docker-pranav/deccan_pranav:${TAG} -t ${DOCKER_REGISTRY}/docker-pranav/deccan_pranav:latest ."
		  				

			}

		}

        stage('Push Image To Registry') {
			steps {
				sh "docker push ${DOCKER_REGISTRY}/docker-pranav/deccan_pranav:${TAG}"
				sh "docker push ${DOCKER_REGISTRY}/docker-pranav/deccan_pranav:latest"
			}
		}



}

}
