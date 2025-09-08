pipeline {
    agent any
    tools {
        nodejs 'nodejs-24'
    }
    environment {
        MONGO_URI = "mongodb://admin:secret@localhost:27017/mydb?authSource=admin"
    }

    stages {
        stage ('Install Dependencies') {
            steps {
                sh 'npm install --no-audit'
                sh 'node seed.js' //Add data into DB
            }
        }
        stage ('Dependency Scanning') {
            parallel {
                stage ('NPM Dependency Audit') {
                    steps {
                        sh '''
                            npm audit --audit-level=crirtical
                            echo $?
                        '''
                    }
                }
                stage ('OWASP Dependency Check') {
                    steps {
                        dependencyCheck additionalArguments: '''
                            --scan ./
                            --out ./
                            --format ALL
                            --prettyPrint''', odcInstallation: 'OWASP-DepCheck-12'

                        dependencyCheckPublisher failedTotalCritical: 2, pattern: 'dependency-check-report.xml', stopBuild: true, unstableTotalCritical: 2

                        junit allowEmptyResults: true, keepProperties: true, testResults: 'dependency-check-junit.xml'

                        publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'Dependency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }

            }
        }
        stage ('Unit Testing') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Mongo-DB-Credentials', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                    sh 'npm test'
                }
                junit allowEmptyResults: true, keepProperties: true, testResults: 'test-results.xml'
                
            }
        }
        stage ('Code Coverage') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Mongo-DB-Credentials', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                    catchError(buildResult: 'SUCCESS', message: 'Code Coverage below threshold. Will be fixed soon!', stageResult: 'UNSTABLE') {
                        sh 'npm run coverage'
                    }
                }
            }
        }
        // stage ('Deploy Application') {
        //     steps {
        //         sh 'npm start'
        //     }
        // }
    }
}