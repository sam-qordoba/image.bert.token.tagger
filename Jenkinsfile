pipeline {
    agent any
    parameters {
        choice(name: 'CONTAINER_REGISTRY', choices: ['devel', 'test', 'prod'], description: 'What container registry should this code, when built into a docker image, be sent to?')
    }

    environment {
        SERVICE = "bert_token_tagger"
        TODAY = '${BUILD_DATE_FORMATTED, "yyyyMMdd"}'
        GIT_HASH = GIT_COMMIT.take(7)
        TAG = VersionNumber (versionNumberString: "date_${TODAY}.git_${BRANCH_NAME}-${GIT_HASH}.buildNo_${BUILD_NUMBER}")
    }

    stages {
        stage('develop') {
            when {
                anyOf {
                    // feature branches go straight to dev
                    branch "eng-*"
                    branch "iq-*"
                    branch "IQ-*"
                    branch "CLAR-*"
                    branch "clar-*"
                    branch "NQ-*"
                    branch "nq-*"
                    branch "no-ticket-*"
                    expression { params.CONTAINER_REGISTRY == "devel" }
                }
            }

            environment {
                PROJECT = "qordoba-devel"
                IMAGE = "gcr.io/${PROJECT}/${SERVICE}:dev-${TAG}"
            }

            steps {
                checkout scm
                sh '''
                docker build -t ${IMAGE} .
                docker push ${IMAGE}
                docker rmi ${IMAGE}
                '''
            }
    }

        stage('test') {
            when {
                allOf {
                    branch "master"
                    expression { params.CONTAINER_REGISTRY == "test" }
                }
            }
            environment {
                PROJECT = "qordoba-test"
                IMAGE = "gcr.io/${PROJECT}/${SERVICE}:tst-${TAG}"
            }
            steps {
                checkout scm
                sh '''
                    docker build -t ${IMAGE} .
                    docker push ${IMAGE}
                    docker rmi ${IMAGE}
                '''
            }
        }
        stage('production') {
            when {
                allOf {
                    branch "master"
                    expression { params.CONTAINER_REGISTRY == "prod" }
                }
            }
            environment {
                PROJECT = "qordoba-prod"
                IMAGE = "gcr.io/${PROJECT}/${SERVICE}:prd-${TAG}"
            }
            steps {
                checkout scm
                sh '''
                    docker build -t ${IMAGE} .
                    docker push ${IMAGE}
                    docker rmi ${IMAGE}
                '''
            }
        }
    }
}