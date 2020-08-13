pipeline {
    
    agent none
    
    stages {
        stage('build-worker') {
        	agent {
        		docker {
            		image 'maven:3.6.1-jdk-8-alpine'
            		args '-v $HOME/.m2:/root/.m2'
        		}
    		}
    		
        	when {
        	    changeset "**/worker/**"
        	}
        	
            steps {
                echo 'Compiling worker app'
                dir('worker') {
                    sh 'mvn compile'   
                }
            }
        }
        
        stage('test-worker') {
        	agent {
        		docker {
            		image 'maven:3.6.1-jdk-8-alpine'
            		args '-v $HOME/.m2:/root/.m2'
        		}
    		}
    		
        	when {
        	    changeset "**/worker/**"
        	}
        	
            steps {
                echo 'Running Unit Tests on worker App'
                dir('worker') {
                    sh 'mvn clean test'   
                }
            }
        }
        
        stage('package-worker') {
        	agent {
        		docker {
            		image 'maven:3.6.1-jdk-8-alpine'
            		args '-v $HOME/.m2:/root/.m2'
        		}
    		}
    		
        	when {
        	    branch 'master'
        	    changeset "**/worker/**"
        	}

            steps {
                echo 'Packaging worker app'
                dir('worker') {
                    sh 'mvn package -DskipTests'  
					archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true 
                }
            }
        }
        
        stage('docker-package-worker') {
        	agent any
        	
        	when {
        		branch 'master'
        	    changeset "**/worker/**"
        	}

            steps {
                echo 'Packaging worker app with Docker'
				script {
				    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {

        				def workerImage = docker.build("visdan/worker:v${env.BUILD_ID}", "./worker")

        				/* Push the container to the custom Registry */
        				workerImage.push()
        				workerImage.push("${env.BRANCH_NAME}")
    				}
				    
				}

            }
        }
        
        stage('build-result') {
        	agent {
        		docker {
            		image 'node:8.16.0-alpine'
        		}
    		}
    
        	when {
        	    changeset "**/result/**"
        	}
        	
            steps {
                echo 'Compiling result app'
                dir('result') {
                    sh 'npm install'
                    sh 'npm ls'   
                }
            }
        }
        
        stage('test-result') {
        	agent {
        		docker {
            		image 'node:8.16.0-alpine'
        		}
    		}
    		
        	when {
        	    changeset "**/result/**"
        	}
        	
            steps {
                echo 'Running Unit Tests on result App'
                dir('result') {
                	sh 'npm install'
                    sh 'npm test'   
                }
            }
        }
        
        stage('docker-package-result') {
        	agent any
        	
        	when {
        		branch 'master'
        	    changeset "**/result/**"
        	}

            steps {
                echo 'Packaging result app with Docker'
				script {
				    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {

        				def workerImage = docker.build("visdan/result:v${env.BUILD_ID}", "./result")

        				/* Push the container to the custom Registry */
        				workerImage.push()
        				workerImage.push("${env.BRANCH_NAME}")
    				}
				    
				}

            }
        }
        
        stage('build-vote') {
        	agent {
        		docker {
            		image 'python:2.7.16-slim'
            		args '--user root'
        		}
    		}
    
        	when {
        	    changeset "**/vote/**"
        	}
        	
            steps {
                echo 'Compiling vote app'
                dir('vote') {
                    sh 'pip install -r requirements.txt' 
                }
            }
        }
        
        stage('test-vote') {
        	agent {
        		docker {
            		image 'python:2.7.16-slim'
            		args '--user root'
        		}
    		}
    		
        	when {
        	    changeset "**/vote/**"
        	}
        	
            steps {
                echo 'Running Unit Tests on vote App'
                dir('vote') {
                	sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'   
                }
            }
        }
        
        stage('docker-package-vote') {
        	agent any
        	
        	when {
        		branch 'master'
        	    changeset "**/vote/**"
        	}

            steps {
                echo 'Packaging vote app with Docker'
				script {
				    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {

        				def workerImage = docker.build("visdan/vote:v${env.BUILD_ID}", "./vote")

        				/* Push the container to the custom Registry */
        				workerImage.push()
        				workerImage.push("${env.BRANCH_NAME}")
    				}
				    
				}

            }
        }
        
        stage('SonarQube') {
            


			agent any
			
			environment {
			    sonarpath = tool 'SonarScanner'
			}

			steps {
			    
			    echo 'Running SonarQube Analysis'
			    withSonarQubeEnv('sonar') {
			        sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
			    }

			}

        }

        
        stage('deploy to dev') {
        	agent any
        	
        	when {
				branch 'master'
        	}
                     
            steps {
            	echo 'Deploy instavote with docker compose'
            	sh 'docker-compose up -d'
            }   
        }
        
    }
    
    post {
        always {
            echo 'Pipeline for instavote app is complete...'
        }
        failure {
            slackSend (channel: "instavote-cd", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success {
			slackSend (channel: "instavote-cd", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
    }
}