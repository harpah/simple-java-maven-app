pipeline {
    agent {
        docker {
            image 'maven:3.9.0'
            args '-v /root/.m2:/root/.m2'
        }
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
                junit 'target/surefire-reports/*.xml'
            }
        }

        stage('Manual Approval') {
            steps {
                script {
                    userInput = input(
                        id: 'manual-approval-input',
                        message: 'Lanjutkan ke tahap Deploy?',
                        parameters: [
                            choice(choices: ['Proceed', 'Abort'], description: 'Pilih tindakan', name: 'ACTION')
                        ]
                    )
                    
                    if (userInput == 'Abort') {
                        error("Pipeline dihentikan oleh pengguna.")
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh './jenkins/scripts/deliver.sh'
            }
        }

        stage('Stop Application') {
            steps {
                // Menjeda eksekusi selama 1 menit setelah deploy
                sleep(time: 60, unit: 'SECONDS')
                // run skrip kill.sh setelah 1 menit
                sh './jenkins/scripts/kill.sh'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
