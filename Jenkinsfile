pipeline {
    agent any
    environment {
        pomVersion = readMavenPom().getVersion()
        BUILD_VERSION =  "${pomVersion}-${currentBuild.number}-SNAPSHOT"
    }
    tools {
        maven 'M3'

    } 
    stages{
        
        stage('Create Release Branch') {
            steps {
                echo "Starting Create Release Branch..."
                sh "git checkout -b '${env.BUILD_VERSION}'"
                sh "mvn versions:set -DnewVersion='${env.BUILD_VERSION}'"
                echo "Create Release Branch: ${currentBuild.currentResult}"
            }
            post {
                success {
                    echo "...Create Release Branch Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                } 
                unsuccessful {
                    echo "...Create Release Branch Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
            }
        }
   
        stage('Build and Test') {
            steps {
                echo "Starting Build and Test..."
                //sh "mvn -Dmaven.test.failure.ignore clean verify"
                echo "Build and Test: ${currentBuild.currentResult}"
            }
            post {
                success {
                    echo "...Build and Test Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                } 
                unsuccessful {
                    echo "...Build and Test Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                    script {
        	            dropLocalReleaseBranch()
                    }
                }
            }
        }
        
        stage('Push Release Branch') {
            steps {
                script {
                    echo "Starting Push Release Branch..."
                    withCredentials([usernamePassword(credentialsId: 'Github', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                      // available as an env variable, but will be masked if you try to print it out any which way
                      // note: single quotes prevent Groovy interpolation; expansion is by Bourne Shell, which is what you want
                      sh 'echo $PASSWORD'
                      // also available as a Groovy variable
                      echo USERNAME
                      // or inside double quotes for string interpolation
                      echo "username is $USERNAME"
                      sh "git add ."
                      sh 'git commit -m "Committing Branch"'
                      sh 'git push https://$USERNAME:$PASSWORD@github.com/pkurimella/hello-world-mule.git'
                        //sh "git push -u origin '${env.BUILD_VERSION}'"
                    }
                } 
            }
            post {
                success {
                    echo "...Push Release Branch Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                } 
                unsuccessful {
                    echo "...Push Release Branch Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                    script {
        	            dropLocalReleaseBranch()
                    }
                }
            }
        }
        
        stage('Deploy Artifact') {
            steps {
                script {
                    echo "Starting Deploy Artifact..."
                    //configFileProvider([configFile(fileId: '107138b1-fdbe-45c5-8eb3-64a520257c07', targetLocation: 'settings.xml', variable: 'MAVEN_SETTINGS_XML')]) {
                        //sh 'mvn -U --batch-mode -s $MAVEN_SETTINGS_XML clean package install deploy'
                    //}
                    sh 'mvn -U --batch-mode clean package install deploy -DskipTests'
                    //sh 'mvn deploy -DskipMunitTests'
                    echo "Artifact Deployed: ${currentBuild.currentResult}"
                }
            }
            post {
                success {
                    echo "...Deploy Artifact Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                } 
                unsuccessful {
                    echo "...Deploy Artifact Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                    script {
        	            dropLocalReleaseBranch()
                        // Drop Remote Release Branch
                    }
                }
            }
        }
        
        stage('Deploy to Development') {
            steps {
                    //applicationName = readMavenPom().getArtifactId()
                    //echo "applicationName=${applicationName}"
                    withCredentials([file(credentialsId: 'Hello-Props', variable: 'FILE')]) {
                        echo "Getting Env Variables"
                        script{
                        //use $FILE
                            def props = readProperties file: FILE
                            env['muleVersion'] = props['muleVersion']
                            env['environment'] = props['environment']
                            env['businessGroup'] = props['businessGroup']
                        }
                        echo "this is me '${env.muleVersion}'"
                        echo "this is me '${env.environment}'"
                        echo "this is me '${env.businessGroup}'"
                    }
                    sh """ mvn clean deploy -U --batch-mode \
                        -DmuleDeploy \
                        -DskipMunitTests \
                        -Dapp.runtime=${env.muleVersion}  \
                        -Denvironment=${env.environment} \
                        -DbusinessGroup="${env.businessGroup}" \
                        -Dmule.env=dev"""
            }
            post {
                success {
                    echo "...Deploy to Development Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                } 
                unsuccessful {
                    echo "...Deploy to Development Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                    script {
        	            dropLocalReleaseBranch()
                        // Drop Remote Branch
                        // Undeploy from Artifact Management Repo
                        // Rollback to previous version
                    }
                }
            }
        }
        
    }

   post {
       success {
           echo "All Good: ${env.BUILD_VERSION}"
            dropLocalReleaseBranch()

       }
       unsuccessful {
           echo "Not So Good: ${env.BUILD_VERSION}"
                    dropLocalReleaseBranch()

       }      
       always {
           echo "Pipeline result: ${currentBuild.result}"
           echo "Pipeline currentResult: ${currentBuild.currentResult}"
       }
   }
   
}

void dropLocalReleaseBranch() {
    echo "Starting Drop Local Release Branch: ${env.BUILD_VERSION} ..."
    sh "git checkout develop"
    sh "git branch -D '${env.BUILD_VERSION}'"
    echo "...Local Release Branch ${env.BUILD_VERSION} Dropped"
}