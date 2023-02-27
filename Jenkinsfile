//pull config repo ok
// update tag in config repo ok
// not yet commit and push
def appSourceRepo = 'https://github.com/luudinhmac/demo-app.git'
def appSourceBranch = 'master'

def appConfigRepo = 'https://github.com/luudinhmac/app-helmchart.git'
def appConfigBranch = 'master'
def helmRepo = "app-helmchart"
def helmChart = "app-demo"
def helmValueFile = "app-demo/values.yaml" 
// app-demo-value.yaml"

def dockerhubAccount = 'dockerhub'
def githubAccount = 'github'

dockerBuildCommand = './'
def version = "v1.${BUILD_NUMBER}"

pipeline {
    agent any    
   
    environment {
        DOCKER_REGISTRY = 'https://registry.hub.docker.com'
        DOCKER_IMAGE_NAME = "luudinhmac/demo-app"
        DOCKER_IMAGE = "registry.hub.docker.com/${DOCKER_IMAGE_NAME}"
    }

    stages {        
        stage('Checkout project') {
          steps {
            git branch: appSourceBranch,
                credentialsId: githubAccount,
                url: appSourceRepo
          }
        }
        stage('Build And Push Docker Image') {
            steps {
                script {
                    sh "git reset --hard"
                    sh "git clean -f"                    
					app = docker.build(DOCKER_IMAGE_NAME, dockerBuildCommand)
                    docker.withRegistry( DOCKER_REGISTRY, dockerhubAccount ) {
                       app.push(version)
                    }

                    sh "docker rmi ${DOCKER_IMAGE_NAME} -f"
                    sh "docker rmi ${DOCKER_IMAGE}:${version} -f"
                }
            }
        }

        stage('Update value in helm-chart') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE'){
                    withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    sh """#!/bin/bash
                        git config --global user.email "nguoiemcuanui@gmail.com"
                        git config --global user.name "luudinhmac"
                        [[ -d ${helmRepo} ]] && rm -r ${helmRepo}
                        git clone ${appConfigRepo} --branch ${appConfigBranch}
                        cd ${helmRepo}
                        sed -i 's|  tag: .*|  tag: "${version}"|' ${helmValueFile}
                        git add . ; git commit -m "Update to version ${version}";git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/luudinhmac/app-helmchart.git
                        cd ..
                        [[ -d ${helmRepo} ]] && rm -r ${helmRepo}
                        """		
                    }	
                }			
            }
        }
    }
}