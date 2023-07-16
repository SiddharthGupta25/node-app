def commitHash


pipeline {

    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('Dockehub-Credentials')
    }

    stages {

        stage('Prepare Build') {
            
            steps {

                    sh "echo [+] The Branch: ${GIT_BRANCH}"                    
                script {
                    commitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    branchName = env.GIT_BRANCH

                    if(branchName.toLowerCase().equals('origin/test')) {
                        echo "[+] Setting up candidate on Test Environment"
                        currentBuild.displayName = "TC" + "-" + "${BUILD_NUMBER}-${commitHash.take(9)}"
                    } 
                    
                    else if(branchName.toLowerCase().equals('origin/staging')) {
                        echo "[+] Setting up release candidate on Staging Environment"
                        currentBuild.displayName = "RC" + "-" + "${BUILD_NUMBER}-${commitHash.take(9)}"
                    }

                    else {
                        echo "[+] Setting up candidate on Development Environment"
                        currentBuild.displayName = "DC" + "-" + "${BUILD_NUMBER}-${commitHash.take(9)}"
                    }
                 }
            }
        } // Stage ends

        stage('Building Image') {
            
            steps {
                sh "echo [+] Building the container image"
                sh "docker build -t node-app:${commitHash.take(9)} ."
                sh "echo [+] The Tag for the built image: ${commitHash}"
            }

        }

        stage('Pushing Image') {

            steps {
                sh "echo DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                sh "docker push node-app:${commitHash.take(9)}"
            }
        }        

    } // stages end

    post {
        always {
            sh "docker logout"
        }
    }
    
}