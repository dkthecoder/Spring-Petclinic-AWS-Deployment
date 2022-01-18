pipeline {
    agent any

    // add timestamps to the console log
    options {
        timestamps()
    }

    environment {
        DOCKER_USERNAME = credentials('docker_username')
        DOCKER_PASSWORD = credentials('docker_password')
    }

    stages {

        stage('Clone repo') {
            steps {
                sh "git clone https://github.com/spring-petclinic/spring-petclinic-angular.git"
                sh "git clone https://github.com/spring-petclinic/spring-petclinic-rest.git"
            }
        }

        stage('Build and push images') {
            steps {
                sh "docker-compose build --parallel"
                sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                sh "docker-compose push"
            }
        } //(richard) could potentially need removing, along with docker-compose //


        //(richard) potentially look at setting up angular and maven in the jenkinsfile so that jenkins can run tests (idk if this is necessary)//

        stage('Config and deploy') {
            steps {
                sh "scp -o StrictHostKeyChecking=no docker-compose.yaml docker-swarm-manager:/home/jenkins/docker-compose.yaml"
                sh "scp -o StrictHostKeyChecking=no nginx.conf docker-swarm-manager:/home/jenkins/nginx.conf"
                sh "ansible-playbook -i ansible/inventory.yaml ansible/playbook.yaml"
                
                //Then clean the workspace after deployment ignoring node_modules directory
                cleanWs notFailBuild: true, patterns: [[pattern: 'node_modules', type: 'EXCLUDE']]
            }
        }

    }

    //post {
        //always {
            //junit '**/*.xml'
            //cobertura coberturaReportFile: 'frontend/coverage.xml', failNoReports: false
            //cobertura coberturaReportFile: 'blackcards/coverage.xml', failNoReports: false
            //cobertura coberturaReportFile: 'whitecards/coverage.xml', failNoReports: false
            //cobertura coberturaReportFile: 'magicmaker/coverage.xml', failNoReports: false
            //sh "docker-compose down || true"
        //}
    //}
}