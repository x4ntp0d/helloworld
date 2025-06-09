pipeline {
    agent any

    environment {
        PYTHONPATH = "${env.WORKSPACE}"
    }

    stages {
        stage('Get Code') {
            steps {
                echo 'Get code'
                git 'https://github.com/x4ntp0d/helloworld.git'
                sh 'ls'
            }
        }

        stage('Build') {
            steps {
                echo 'Python cannot be compiled, it is interpreted'
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh '''
                                pytest --junitxml=result-unit.xml test/unit
                            '''
                        }
                    }
                }

                stage('Rest') {
                    environment {
                        FLASK_APP = 'app/api.py'
                        FLASK_ENV = 'development'
                    }
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh '''
                                java -jar test/wiremock/wiremock-standalone-3.13.0.jar \
                                  --port 9090 \
                                  --root-dir test/wiremock &

                                flask run --host=0.0.0.0 --port=5000 &
                                ping -n 10 127.0.0.1             
                                pytest --junitxml=result-rest.xml test/rest
                            '''
                        }
                    }
                }
            }
        }

        stage('Results') {
            steps {
                junit 'result-*.xml'
            }
        }
    }
}
