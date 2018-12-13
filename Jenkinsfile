

pipeline {
  agent {
        docker {
            image 'node:latest' 
            //args  '-v $JENKINS_HOME/workspace/$BUILD_TAG:/project -u root:root'
        }
  }
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
    string(name: 'DOCKER_COMPOSE_FILENAME', defaultValue: 'docker-compose.yml', description: '')
    string(name: 'DOCKER_STACK_NAME', defaultValue: 'react_stack', description: '')
    booleanParam(name: 'NPM_RUN_TEST', defaultValue: true, description: '')
    booleanParam(name: 'PUSH_DOCKER_IMAGES', defaultValue: false, description: '')
    booleanParam(name: 'DOCKER_STACK_RM', defaultValue: true, description: 'Remove previous stack.  This is required if you have updated any secrets or configs as these cannot be updated. ')
  }
  stages {



  stage('who am i?'){
      steps{
        dir("$JENKINS_HOME/workspace/$BUILD_TAG"){
            sh "whoami" 
            sh "id root"
            sh "hostname"
            sh "ls -lathr /"
            sh "pwd"
        }
         
      }
    }
    
    stage('copy docker files from master to workspace'){
      steps{
        
            sh "cp -arv * $JENKINS_HOME/workspace/$BUILD_TAG/" 
            
        
      }
    }

    

    stage('clone app repo'){
      environment {
        GIT_REACT_APP_URL = "${params.GIT_REACT_APP_ROOT}/${params.GIT_REACT_APP_PROJECT_NAME}.git"
      }
      steps{
         dir("$JENKINS_HOME/workspace/$BUILD_TAG"){
           sh "whoami" 
           sh "ls -lathr" 
            sh "hostname"
            sh "git clone $GIT_REACT_APP_URL"
            sh "pwd"
         }
      }
    }


    stage('npm install'){
      
        steps{
          dir("$JENKINS_HOME/workspace/$BUILD_TAG/${params.GIT_REACT_APP_PROJECT_NAME}"){
            sh "hostname"
            sh "ls -lathr"
            sh "pwd"
            sh "npm install"
          
        }
      }
      
    }
    
    stage('npm test'){

      steps{
         dir("$JENKINS_HOME/workspace/$BUILD_TAG/${params.GIT_REACT_APP_PROJECT_NAME}"){
            sh "hostname"
            sh "pwd"
            sh "ls -lathr"
            sh "npm test -- --coverage"
         }
      }

    }


    stage('npm build'){

      steps{
         dir("$JENKINS_HOME/workspace/$BUILD_TAG"){
            sh "hostname"
            sh "pwd"
            sh "ls -lathr"
            sh "npm run build"
         }
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
          sh "docker build -t $BUILD_IMAGE_REPO_TAG ."
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
      sh 'ls -lathr'
      sh 'rm -rf simple-react-app'
      sh 'ls -lathr'
    }
  }
}


