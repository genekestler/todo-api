pipeline {
   options {              
      timeout(time: 5, unit: 'MINUTES')
      buildDiscarder(logRotator(numToKeepStr: '5'))
      skipDefaultCheckout()
   }
	agent {label 'docker-cloud'}
//	script {artifactName = *.war'}
	stages {
		stage('Checkout') {
			steps {echo 'INFO - Retrieving Source'
			//	git 'https://github.com/jglick/simple-maven-project-with-tests.git'
				checkout scm		// create a multi-branch project that only checks out this branch
				gitShortCommit(7)
			}
		}
		stage('Build') {
			agent {label 'maven-jdk-8'}
			steps {echo 'INFO - Starting Build phase'
			//     sh 'mvn validate'
			//     sh 'mvn compile'
			       sh 'mvn -Dmaven.test.failure.ignore clean install' // clean install does a compile, so no reason to do compile, also runs unit tests
			}
		}
		stage('Test') {
			agent {label 'maven-jdk-8'}
			steps {
				parallel (
					"unit test": {
						echo 'unit tests'
						sh 'mvn test'
						// generate a maven unit test report using surefire
						sh 'mvn surefire-report:report'
						// this is the path to your unit test report: your-project/target/site/surefire-report.html
					},
					"integration tests": {
						echo 'integration tests'
						sh 'mvn verify -fn'	// generate a maven integration test report
						junit '**/target/surefire-reports/TEST-*.xml'	// generate a junit test report
					}, failFast: true
				)
			}
		}
		stage('Package') {
			agent {label 'maven-jdk-8'}
			steps {echo 'INFO - Starting Package phase'
				sh 'mvn clean package'
				stash name: 'artifactName', includes: '*.xml'
			}
		}
		stage('Deploy Archive Artifacts') {
			agent {label 'maven-jdk-8'}
			steps {echo 'INFO - Starting Deploy phase'
	//-------------------------------------------------------------
	//		sh 'mvn deploy' // this won't work until the todo-api pom is not configured to deploy
	//		sh """
	//   		rm -rf ${somevalue}/webapps/ROOT                 // remove a directory and files without comfirmation
	//			mv -f ${somevalue}                               // move files from one directory to another without prompting
	//			cp -rf ${war} ${somevalue}/webapps/ROOT.war      // copy files from one location to another without prompting
	//			${somevalue}/bin/startup.sh
	//		"""
	//-------------------------------------------------------------
			}
		}
	}
	post {
		success {
			unstash 'artifactName'
			archive "target/${artifactName}"
		}
	}
}
