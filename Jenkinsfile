import hudson.tasks.test.AbstractTestResultAction

node {
    def testEnvs = TEST_ENVIRONMENTS.replace("\\n", "\n")
    allure includeProperties: false, jdk: '', results: [[path: 'allure-results']]

    properties(
            [
                    disableConcurrentBuilds(),
                    parameters([choice(choices: testEnvs, description: '', name: 'SPRING_PROFILES_ACTIVE'),
                                booleanParam(defaultValue: true, description: '', name: 'GRID_ENABLED')]),
                    [$class              : 'ThrottleJobProperty', categories: [], limitOneJobWithMatchingParams: false,
                     maxConcurrentPerNode: 0, maxConcurrentTotal: 0, paramsToUseForLimit: '', throttleEnabled: false,
                     throttleOption      : 'project'], pipelineTriggers([cron('00 07 * * *')])
            ]
    )

    try {
        stage('Checkout') {
            git credentialsId: '', url: ''
        }

        stage('Create network and containers') {
            sh 'docker-compose up --scale chrome-setpace=8 -d'

            docker.image(JAVA_MAVEN_DOCKER_IMAGE).inside('-it --net "selenium_grid_dockerized_selenium-network-setpace"') {
                stage('Run tests') {
                    configFileProvider([configFile(fileId: MVN_SETTINGS_FILE_ID, variable: 'MAVEN_SETTINGS')]) {
                        sh 'mvn -s $MAVEN_SETTINGS clean test'
                    }
                }
            }
        }
    }

    finally {
        stage('Remove network and containers') {
            sh 'docker-compose down'
        }

        stage('Publish test result') {
            junit 'target/surefire-reports/*.xml'
            archiveArtifacts artifacts: 'target/screenshots/**/*.png', allowEmptyArchive: true

            AbstractTestResultAction testResultAction = currentBuild.rawBuild.getAction(AbstractTestResultAction.class)
            if (testResultAction != null) {
                if (testResultAction.failCount == 0) {
                    currentBuild.rawBuild.@result = hudson.model.Result.SUCCESS
                }
            }
        }
    }
}