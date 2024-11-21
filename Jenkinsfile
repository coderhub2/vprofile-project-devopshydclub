pipeline {
    
	agent any
	
	tools {
        maven "maven3"
        jdk "JDK11"
	
    }
	
    environment {
       registry = "coderhub1/vproapp"
       registryCredential = "dockerhub"
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

	stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

	stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
		
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool 'sonarscanner4'
          }

          steps {
            withSonarQubeEnv('sonar-pro') {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }

            timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
            }
          }
        }

        stage('build app image'){
            steps{
                script{
                  dockkerImage = docker.build registry + ":$BUILD_NUMBER"
            }

            }
            
        }

        stage('upload image to docker'){
            steps{
                script{
                    docker.withRegistry('', registryCredential)
                    dockkerImage.push("$BUILD_NUMBER")
                    dockkerImage.push("latest")
                }
            }
        }

        stage('remove unused images'){
            steps{
                sh "docker rmi registry + :$BUILD_NUMBER"
            }
        }

        stage('kube deploy'){
            agent{label 'KOPS'}
              steps{
                sh "helm upgrade --install --force vpro-stack helm/vprofilecharts --set appimage=${registry}:${$BUILD_NUMBER} --namespace prod"
              }
        }
        
    }


}
