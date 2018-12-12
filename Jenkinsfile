pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  parameters {
    //https://github.com/Kornil/simple-react-app.git
    string(name: 'GIT_IMAGE_REPO_NAME', defaultValue: 'mat-orz/basic-react', description: '')
    string(name: 'GIT_REACT_APP_PROJECT_NAME', defaultValue: 'simple-react-app', description: '')
    string(name: 'GIT_REACT_APP_ROOT', defaultValue: 'https://github.com/Kornil', description: '')
    string(name: 'DOCKER_IMAGE_REPO_NAME', defaultValue: 'matorz/basic-react', description: '')
    string(name: 'LATEST_BUILD_TAG', defaultValue: 'build-latest', description: '')
    string(name: 'DOCKERFILE_REACT', defaultValue: 'Dockerfile-react', description: '')
    string(name: 'DOCKER_COMPOSE_FILENAME', defaultValue: 'docker-compose-react.yml', description: '')
    string(name: 'DOCKER_STACK_NAME', defaultValue: 'react_stack', description: '')
    booleanParam(name: 'NPM_RUN_TEST', defaultValue: true, description: '')
    booleanParam(name: 'PUSH_DOCKER_IMAGES', defaultValue: false, description: '')
    booleanParam(name: 'DOCKER_STACK_RM', defaultValue: true, description: 'Remove previous stack.  This is required if you have updated any secrets or configs as these cannot be updated. ')
  }
  stages {
/*
    stage('clone app repo'){
      environment {
        GIT_REACT_APP_URL = "${params.GIT_REACT_APP_ROOT}/${params.GIT_REACT_APP_PROJECT_NAME}.git"
      }
      steps{
        dir("$JENKINS_HOME/workspace/$BUILD_TAG"){
         sh "hostname"
         sh "git clone $GIT_REACT_APP_URL"
         sh "pwd"
         sh "ls -lathr"      
        }
      }
    }
*/

    stage('project mkdir'){
      
      steps{
          sh "mkdir $JENKINS_HOME/workspace/$BUILD_TAG/${params.GIT_REACT_APP_PROJECT_NAME}"
       
      }
    }

    stage('clone app repo'){
      agent {
          docker {
             image 'node:latest'
             customWorkspace "$JENKINS_HOME/workspace/$BUILD_TAG/${params.GIT_REACT_APP_PROJECT_NAME}"
          }
      }
      environment {
        GIT_REACT_APP_URL = "${params.GIT_REACT_APP_ROOT}/${params.GIT_REACT_APP_PROJECT_NAME}.git"
      }
      steps{
         sh "hostname"
         sh "git clone $GIT_REACT_APP_URL"
         sh "pwd"
         sh "ls -lathr"  
         
      }
    }


    stage('npm install'){
      agent {
          docker {
             image 'node:latest'
             customWorkspace "$JENKINS_HOME/workspace/$BUILD_TAG/${params.GIT_REACT_APP_PROJECT_NAME}"
          }
      }
      steps{
         sh "hostname"
         sh "pwd"
         sh "npm install"
         
      }
    }
    
    stage('npm test'){
      agent {
          docker {
             image 'node:latest'
             customWorkspace "$JENKINS_HOME/workspace/$BUILD_TAG/${params.GIT_REACT_APP_PROJECT_NAME}"
          }
      }
      steps{
         sh "hostname"
         sh "pwd"
         sh "ls -lathr"
         sh "npm test -- --coverage"
      }
    }
    stage('npm build'){
      agent {
          docker {
             image 'node:latest'
             customWorkspace "$JENKINS_HOME/workspace/$BUILD_TAG/${params.GIT_REACT_APP_PROJECT_NAME}"
          }
      }
      steps{
         sh "pwd"
         sh "npm run build"
      }
    }
    stage('docker build'){
      environment {
        COMMIT_TAG = sh(returnStdout: true, script: 'git rev-parse HEAD').trim().take(7)
        BUILD_IMAGE_REPO_TAG = "${params.GIT_IMAGE_REPO_NAME}:${env.BUILD_TAG}"
      }
      steps{
        dir("$JENKINS_HOME/workspace/$BUILD_TAG"){
          sh "pwd"
          sh "ls -lathr"
          sh "cat Dockerfile-react"
          sh "docker build -f ${params.DOCKERFILE_REACT} -t $BUILD_IMAGE_REPO_TAG ."
          sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.DOCKER_IMAGE_REPO_NAME}:$COMMIT_TAG"
          sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.DOCKER_IMAGE_REPO_NAME}:${readJSON(file: 'package.json').version}"
          sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.DOCKER_IMAGE_REPO_NAME}:${params.LATEST_BUILD_TAG}"
          sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.DOCKER_IMAGE_REPO_NAME}:$BRANCH_NAME-latest"
        }
      }
    }
    stage('docker push'){
      when{
        expression {
          return params.PUSH_DOCKER_IMAGES
        }
      }
      environment {
        COMMIT_TAG = sh(returnStdout: true, script: 'git rev-parse HEAD').trim().take(7)
        BUILD_IMAGE_REPO_TAG = "${params.DOCKER_IMAGE_REPO_NAME}:${env.BUILD_TAG}"
      }
      steps{
          sh "docker login -u \$(cat /run/secrets/DOCKER_LOGIN) -p \$(cat /run/secrets/DOCKER_PASSWORD)"
        //sh "docker push $BUILD_IMAGE_REPO_TAG"
        //sh "docker push ${params.DOCKER_IMAGE_REPO_NAME}:$COMMIT_TAG"
        //sh "docker push ${params.DOCKER_IMAGE_REPO_NAME}:${readJSON(file: 'package.json').version}"
        //sh "docker push ${params.DOCKER_IMAGE_REPO_NAME}:${params.LATEST_BUILD_TAG}"
        //sh "docker push ${params.DOCKER_IMAGE_REPO_NAME}:$BRANCH_NAME-latest"
      }
    }
    stage('Remove Previous Stack'){
      when{
        expression {
          return params.DOCKER_STACK_RM
        }
      }
      steps{
        sh "docker stack rm ${params.DOCKER_STACK_NAME}"


      }
    }
    stage('Docker Stack Deploy'){
      steps{
        sh "docker stack deploy -c ${params.DOCKER_COMPOSE_FILENAME} ${params.DOCKER_STACK_NAME}"
      }
    }
  }
  post {
    always {
      sh 'echo "This will always run"'
    }
  }
}


