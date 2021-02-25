pipeline {
    agent any
	tools {
        maven 'maven_3.6.3'
    }
    stages {
        stage('Checkout') {
            steps {
				checkout([$class: 'GitSCM', branches: [[name: 'master']], extensions: [[$class: 'CheckoutOption', timeout: 5], [$class: 'CloneOption', noTags: false, reference: '', shallow: false, timeout: 5]], userRemoteConfigs: [[url: 'https://github.com/dilipshukla21/devopsTraining.git']]])
                echo 'Checkout source code from git'
            }
        }
	    parallel {
	   	 stage('Quality Check') {
		    steps {
			sh "mvn clean test"    
			echo 'QA verified'
		    }
		}
		stage('Security Check') {
		    steps {
			dependencyCheck additionalArguments: '--scan=. --format=HTML', odcInstallation: 'OWASP-Dependency-Check'
			echo 'All security checks done'
		    }
		}
	}
        stage('Build') {
            steps {
				sh "mvn clean install"
                echo 'Building App with maven'
            }
        }
	stage('Kill previous deploy ment') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    sh "fuser -k 8083/tcp"
                }
            }
        }
        stage('Deploy') {
            steps {
                sh "JENKINS_NODE_COOKIE=dontKillMe nohup java -jar ./target/spring-boot-rest-2-0.0.1-SNAPSHOT.jar &"
		echo 'Deployment done'
            }
        }
        stage('Post Deployment Check') {
            steps {
                sh "/usr/local/bin/newman run Student_Training.postman_collection.json -r html,cli"
		echo 'All deployment check done'
            }
        }
    }
}
