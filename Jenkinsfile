pipeline{
    agent {label 'docker'} 
    
    parameters {
        string(name: 'EmailRecipients', defaultValue: 'sandeep.darsan@qburst.com', description: 'Email Recipients for sharing the test report')
    }

    environment {
        version = ''
    }

    stages{
        stage('Prepare') {
            steps {
                script {
                    version = prepare()
                    createBuild(version)
                }
            }
        }
        stage('Maven Build') {
            agent {
                    dockerfile {
                        reuseNode true
                        additionalBuildArgs '--build-arg IMAGE_SRC=artifactory.zii.aero/dockerhub-docker-prod-remote/maven:3.8.6-jdk-8 --shm-size 3g'
                        args '-v /tmp:/tmp --shm-size 3g'
                } 
            }
            environment {
                MAVEN_HOME = '/usr/share/maven'
            }
            steps {
                sh 'which firefox'
                sh 'firefox -v'
                mavenBuild(version)
                publishBuild()
            }

        }
        stage('Release') {
            when {
                beforeAgent true
                expression { return !pipelineConfig.libraries.semverGit.skipRelease }
            }
            steps {
                release()
                // Promotes maven packages from the dev/snapshot repo to prod
                promote()
            }
        }
    }
    
    post {
        always {
            emailext mimeType: 'text/html',
                body: '${FILE, path="'+"${WORKSPACE}"+'/target/html-email-report.html"}',
                subject: currentBuild.currentResult + " : " + env.JOB_NAME + " : " + env.BUILD_NUMBER,
                to: "${params.EmailRecipients}",
                recipientProviders: [[$class: 'RequesterRecipientProvider']],
                attachmentsPattern: '**/extent-report.html',
                attachLog: true,
                compressLog: true
            
            cleanWs(deleteDirs: true)
            }
        }
}