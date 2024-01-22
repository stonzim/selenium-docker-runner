pipeline {

    agent any

    parameters {
        choice choices: ['chrome', 'firefox', 'edge'], description: 'Select the browser', name: 'BROWSER'
        choice choices: ['1', '2', '3', '4'], description: 'Select the number of browser containers to run. Recommend using the same number as that of suites being run.', name: 'NUMBER_OF_BROWSERS'
        choice choices: ['1', '2', '3', '4'], description: 'Select the number of TestNG threads to run.', name: 'THREAD_COUNT'
        choice choices: ['vendor-portal.xml', 'flight-reservation.xml', 'all-suites.xml'], description: 'Select the single suite you would like to run, or select "All Suites".', name: 'TEST_SUITE'
    }

    stages {

        stage('Start Grid') {
            steps {
                sh "THREAD_COUNT=${params.THREAD_COUNT} TEST_SUITE=${params.TEST_SUITE} docker compose -f docker-grid.yaml up --scale ${params.BROWSER}=${params.NUMBER_OF_BROWSERS} -d"
            }
        }

        stage('Run Test') {
            steps {
                sh "docker compose -f test-suites.yaml up --pull=always"
                script {
                    if(fileExists("output/flight-reservation/testng-failed.xml") || fileExists("output/vendor-portal/testng-failed.xml")) {
                        error("Failed test/s found")
                    }
                }
            }
        }

    }

    post {
        always {
            sh "docker compose -f docker-grid.yaml down"
            sh "docker compose -f test-suites.yaml down"
            archiveArtifacts artifacts: "output/${params.TEST_SUITE}/emailable-report.html", followSymlinks: false
        }
    }

}
