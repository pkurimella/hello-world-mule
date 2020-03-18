pipeline {
    agent any
    environment {
        //BUILD_VERSION = "v${currentBuild.number}.RELEASE"
        //BUILD_VERSION = "1.0.0-SNAPSHOT.RELEASE"
        pomVersion = readMavenPom().getVersion()
        BUILD_VERSION =  "${pomVersion}-${currentBuild.number}-SNAPSHOT"
        //echo "${parsedVersion.majorVersion}.${parsedVersion.minorVersion}.${parsedVersion.nextIncrementalVersion}-${parsedVersion.qualifier}"
    }
    tools {
        maven 'M3'
        java 'J191'
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
                sh "mvn -Dmaven.test.failure.ignore clean verify"
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
        
        // stage('Push Release Branch') {
        //     steps {
        //         script {
        //             echo "Starting Push Release Branch..."
        //             // Success
        //             //sh "git add pom.xml"
        //             //sh "ssh -vT -o StrictHostKeyChecking=no git@github.com"
        //             //sh "ssh -vvv -o StrictHostKeyChecking=no git@github.com"
        //             sh "ssh -v -T -i /var/lib/jenkins/.ssh/id_rsa git@github.com:"
        //             sh "git add ."
        //             sh 'git commit -m "Committing Branch"'
        //             sh "git remote set-url origin git@github.com:pkurimella/employee-system-api.git"
        //             sh "git push -u origin '${env.BUILD_VERSION}'"
        //         } 
        //     }
        //     post {
        //         success {
        //             echo "...Push Release Branch Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
        //         } 
        //         unsuccessful {
        //             echo "...Push Release Branch Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
        //             script {
        // 	            dropLocalReleaseBranch()
        //             }
        //         }
        //     }
        // }

        stage('Clean up Local Release Branch') {
            steps {
                script {
                    echo "Starting Clean Release Branch..."
                    // Success
                    sh "git add ."
                    sh 'git commit -m "Committing Branch"'
                    //dropLocalReleaseBranch()
                } 
            }
            post {
                success {
                    echo "...Clean Release Branch Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                } 
                unsuccessful {
                    echo "...Clean Release Branch Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
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
                script {
                    echo "Starting Deploy to Development"
                        
                    //applicationName = readMavenPom().getArtifactId()
                    //echo "applicationName=${applicationName}"
                    
                    props = readProperties(file: '/tmp/Jenkins/employee-system-api/deploy.properties')
                    muleVersion = props['muleVersion']
                    environment = props['environment']
                    applicationName = props['cloudhub.applicationName']
                    //anypointUsername = props['anypoint.username']
                    //anypointPassword = props['anypoint.password']
                    businessGroup = props['businessGroup']
                    cloudhubRegion = props['cloudhub.region']
                    cloudhubWorkerType = props['cloudhub.workerType']
                    cloudhubWorkers = props['cloudhub.workers']
                    cloudhubEnvClientID = props['cloudhub.env.client_id']
                    cloudhubEnvSecretID = props['cloudhub.env.client_secret']
        
                    // echo "anypointMuleVersion=${anypointMuleVersion}"
                    // echo "anypointMuleEnvironment=${anypointMuleEnvironment}"
                    // echo "anypointUsername=${anypointUsername}"
                    // echo "anypointPassword=${anypointPassword}"
                    // echo "cloudhubEnv=${cloudhubEnv}"
                    // echo "cloudhubRegion=${cloudhubRegion}"
                    // echo "cloudhubWorkerType=${cloudhubWorkerType}"
                    // echo "cloudhubWorkers=${cloudhubWorkers}"
                    // echo "cloudhubEnvClientID=${cloudhubEnvClientID}"
                    // echo "cloudhubEnvSecretID=${cloudhubEnvSecretID}"
                        
                    echo "Deploy to Development: ${currentBuild.currentResult}"
                }
                
                //configFileProvider([configFile(fileId: '107138b1-fdbe-45c5-8eb3-64a520257c07', targetLocation: 'settings.xml', variable: 'MAVEN_SETTINGS_XML')]) {
                    // Run the maven build
                    // sh """ mvn -U --batch-mode -s $MAVEN_SETTINGS_XML \
                    //     -Dmule.version=${anypointMuleVersion}  \
                    //     -Danypoint.applicationName=${applicationName} \
                    //     -Danypoint.mule.environment=${anypointMuleEnvironment} \
                    //     -Danypoint.username=${anypointUsername} \
                    //     -Danypoint.password=${anypointPassword} \
                    //     -Dcloudhub.env=${cloudhubEnv} \
                    //     -Dcloudhub.region=${cloudhubRegion} \
                    //     -Dcloudhub.workerType=${cloudhubWorkerType} \
                    //     -Dcloudhub.workers=${cloudhubWorkers} \
                    //     -Dcloudhub.env.client_id=${cloudhubEnvClientID} \
                    //     -Dcloudhub.env.client_secret=${cloudhubEnvSecretID} \
                    //     clean install mule:deploy -P cloudhub """
                    sh """ mvn deploy -U --batch-mode \
                        -DmuleDeploy \
                        -DskipMunitTests \
                        -Dmule.version=${muleVersion}  \
                        -Danypoint.applicationName=${applicationName} \
                        -Denvironment=${environment} \
                        -DworkerType=${cloudhubWorkerType} \
                        -Dworkers=${cloudhubWorkers} \
                        -DbusinessGroup="${businessGroup}" \
                        -Dcloudhub.env.client_id=${cloudhubEnvClientID} \
                        -Dcloudhub.env.client_secret=${cloudhubEnvSecretID} \
                        -Dmule.env=dev"""

                //}
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