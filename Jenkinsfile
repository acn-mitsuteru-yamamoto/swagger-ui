pipeline {
    //Dont change this. This agent definition keeps the jobs using up nodes for management.
    agent none

    //defining common variables.
    environment{
    GITHUB_URL = "https://github.com/fastretailing/AR2-Frontend-ref-component.git"
    PROJECT_NAME = "AR2-Frontend-ref-component"
    SLACK_CHANNEL = "#frontend"
    }

    //Tools used by the pipeline go here
    tools{
        maven 'maven 3.5.0'
        nodejs 'nodejs 8.4.0'
        git 'Default'
    }

    stages {
        stage ('Initialize') {
        agent { label 'dynamic-jp' }
            steps {
                sh 'echo "Hello"'
                sh "printenv"
            }
        }


        stage('Building') {
        agent { label 'dynamic-jp' }
            steps {
                cleanWs()
                checkout scm: [
                        $class: 'GitSCM',
                        userRemoteConfigs: [[credentialsId: 'github-deploybot-account', url: "${GIT_URL}"]],
                        branches: [[name: "${GIT_COMMIT}"]]
                ]


                nodejs(nodeJSInstallationName: 'nodejs 8.4.0', configId:null){
                    sh "rm -rf ./node_modules/ && npm install"
                    sh "ionic build --prod"
                }
            }
        }

        stage('Scanning') {
        agent { label 'dynamic-jp' }
        when{
            anyOf { branch 'master'; branch 'hotfix' ; branch 'release' ; branch 'develop'}
        }
            steps {
                cleanWs()
                checkout scm: [
                        $class: 'GitSCM',
                        userRemoteConfigs: [[credentialsId: 'github-deploybot-account', url: "${GIT_URL}"]],
                        branches: [[name: "${GIT_COMMIT}"]]
                ]


                nodejs(nodeJSInstallationName: 'nodejs 8.4.0', configId:null){
                    sh "npm run test-ci || true"
                }
                junit 'config-test/report/junit/*xml'

                echo 'Upload to sonarqube'
                withSonarQubeEnv('sonarqube') {
                    nodejs(nodeJSInstallationName: 'nodejs 8.4.0', configId:null){
                        sh "npm run sonar -- -Dsonar.projectName=${PROJECT_NAME} -Dsonar.branch=$BRANCH_NAME -Dsonar.projectKey=${PROJECT_NAME}"
                    }
                }
            }
        }

        stage("Quality Gate"){
        agent { label 'dynamic-jp' }
        when{
            anyOf { branch 'master'; branch 'hotfix' ; branch 'release' ; branch 'develop'}
        }
            steps{
                script {
                    timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
                    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                        if (qg.status != 'OK') {
                            echo "[FAILURE] Failed to build"
                            currentBuild.result = 'FAILURE'
                            exit
                        }
                    }
                }
            }   
        }

        stage('Packaging') {
        agent{ label 'dynamic-jp'}
        when{
            anyOf { branch 'master'; branch 'hotfix' ; branch 'release' ; branch 'develop'}
        }
            steps {
                echo 'Packaging..AR2 Frontend APP'
                pwd()
                sh "ls -ltra"
                sh "tar czvf AR2-Frontend-ref-component-'$BRANCH_NAME'.tar.gz www/"
                sh "ls -ltra"

            }
        }


        stage('Deploying to production environment') {
        agent { label 'dynamic-jp' }
        when{
            branch 'master'
        }
            steps {
                sh 'echo "deploying to production"'
            }
        }


        stage('Deploying to staging environment') {
        agent { label 'dynamic-jp' }
        when{
            anyOf { branch 'master'; branch 'hotfix' }
        }
            steps {
                sh 'echo "deploying to staging"'
            }
        }


        stage('Deploying to release environment') {
        agent { label 'dynamic-jp' }
        when{
        branch 'release'
        }
            steps {
                sh 'echo "deploying to release"'
            }
        }


        stage('Deploying to development environment') {
        agent { label 'dynamic-jp' }
        when{
        branch 'development'
        }
            steps{
                echo "uploading to S3"
                echo "$BUILD_NUMBER"
                echo "$BRANCH_NAME"
                // sh "curl -O -u sa-jenkins:AKCp5Z2Y3qv6JUGFskdbP3fdUzxZ1LSq8XsvcnLhr49Wy4fVovSAjsujTXgxYRWHhFJnLvd9H -X GET http://10.251.62.20/artifactory/frontend-app/'$BRANCH_NAME'/frontend-app-'$BUILD_NUMBER'.tar.gz"
                // sh "tar -xvzf AR2-Frontend-ref-component-'$BRANCH_NAME'.tar.gz"

                sh "ls -ltra"
                withAWS(credentials: 'bluepoc-exceldemo.com'){
                    s3Upload(file:'www', bucket:'vrf-ariake2.0-dev-s3-website', path:'')
                }
                sh "ls -ltra"

                cleanWs()
            }
        }
    }
    post{
        success{
        slackSend baseUrl: 'https://accenture-ariakepj.slack.com/services/hooks/jenkins-ci/', channel: "${SLACK_CHANNEL}", color: '#00FF00', message: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}] ${env.BRANCH_NAME}' (<${env.BUILD_URL}|Open>)", token: 'P48phroMfwHXmETeILiGEWaP'
        }
        unstable{
        slackSend baseUrl: 'https://accenture-ariakepj.slack.com/services/hooks/jenkins-ci/', channel: "${SLACK_CHANNEL}", color: '#FFFF00', message: "UNSTABLE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}] ${env.BRANCH_NAME}' (<${env.BUILD_URL}|Open>)", token: 'P48phroMfwHXmETeILiGEWaP'
        }
        failure{
        slackSend baseUrl: 'https://accenture-ariakepj.slack.com/services/hooks/jenkins-ci/', channel: "${SLACK_CHANNEL}", color: '#FF0000', message: "FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}] ${env.BRANCH_NAME}' (<${env.BUILD_URL}|Open>)", token: 'P48phroMfwHXmETeILiGEWaP'
        }
    }
    options {
        buildDiscarder(logRotator(numToKeepStr:'10'))
        timeout(time: 60, unit: 'MINUTES')
    }
}
