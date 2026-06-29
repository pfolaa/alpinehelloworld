pipeline {
	environment {
		IMAGE_NAME = "alpinehelloworld"
		IMAGE_TAG = "latest"
		STAGING = "eazytraining-staging"
		PRODUCTION = "eazytraining-production"
	}
	agent none
	stages {
		stage('Build image') {
			agent any
			steps {
				script {
					sh 'docker build -t paulin84/$IMAGE_NAME:$IMAGE_TAG .'
				}
			}
		}
		stage('Run container based on builded image') {
			agent any
			steps {
				script {
					sh '''
						docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 paulin84/$IMAGE_NAME:$IMAGE_TAG
						sleep 5
					'''
				}
			}
		}
		stage('Test image') {
			agent any
			step {
				script {
					sh '''
						curl http://192.168.56.100 | grep -q "Hello world!"
					'''
				}
			}
		}

		stage ('Login and Push Image on docker hub') {
          	agent any
	        environment {
	           DOCKERHUB_PASSWORD  = credentials('dockerhub')
	        }            
		  steps {
			 script {
			   sh '''
				   echo $DOCKERHUB_PASSWORD_PSW | docker login -u $ID_DOCKER --password-stdin
				   docker push ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
			   '''
			 }
		  }
	  }    
		
		stage('Clean Container') {
			agent any
			steps {
				script {
					sh '''
						docker stop $IMAGE_NAME
						docker rm $IMAGE_NAME
					'''
				}
			}
		}
		stage('Push image in staging and deploy it') {
			when {
				expression {GIT_BRANCH == 'origin/master'}
			}
			agent any
			environment {
				HEROKU_API_KEY = credentials('heroku_api_key')
			}
			steps {
				script {
					sh '''
						heroku container:login
						heroku create $STAGING || echo "project already exist"
						heroku container:push -a $STAGING web
						heroku container:release -a $STAGING web
					'''
				}
			}
		}
		stage('Push image in production and deploy it') {
			when {
					expression {GIT_BRANCH == 'origin/master'}
				}
			agent any
			environment {
				  HEROKU_API_KEY = credentials('heroku_api_key')
			}
			steps {
				script {
					sh '''
						heroku container:login
						heroku create $PRODUCTION || echo "project already exist"
						heroku container:push -a $PRODUCTION web
						heroku container:release -a $PRODUCTION web
					'''
				}
			}
		}
	}
}
