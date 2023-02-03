pipeline {
    agent any
        DOCKER_IMAGE_NAME = "csouvik187/train-schedule"
    }
    tools {
        jdk 'java11'
    }
    stages {
        stage('compile') {
	         steps {
                // step1 
                echo 'compiling..'
		            git url: 'https://github.com/S-o-u-v-i-k/cicd-pipeline-train-schedule-autodeploy'
		            sh script: '/opt/maven/bin/mvn compile'
           }
        }
        stage('codereview-pmd') {
	         steps {
                // step2
                echo 'codereview..'
		            sh script: '/opt/maven/bin/mvn -P metrics pmd:pmd'
           }
	         post {
               success {
		             recordIssues enabledForFailure: true, tool: pmdParser(pattern: '**/target/pmd.xml')
               }
           }		
        }
        stage('unit-test') {
	          steps {
                // step3
                echo 'unittest..'
	               sh script: '/opt/maven/bin/mvn test'
            }
	          post {
               success {
                   junit 'target/surefire-reports/*.xml'
               }
            }			
        }
        stage('package/build-war') {
	         steps {
                // step5
                echo 'package......'
		            sh script: '/opt/maven/bin/mvn package'	
           }		
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
