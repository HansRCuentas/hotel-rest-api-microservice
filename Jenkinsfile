pipeline {
    agent none
    stages {
        // stage('Build') {
        //     steps {
        //         sh 'mvn clean compile -B -ntp'
        //     }
        // }
        // stage('Junit-Test') {
        //     steps {
        //         sh 'mvn test -Dmaven.test.failure.ignore=true -B -ntp'
        //     }
        //     post {
        //         always {
        //             junit 'target/surefire-reports/*.xml'
        //         }
        //     }
        // }
        // stage('Jacoco-Coverage') {
        //     steps {
        //         sh 'mvn jacoco:report -B -ntp'
        //     }
        //     post {
        //         success {
        //             recordCoverage(tools: [[parser: 'JACOCO']])
        //         }
        //     }
        // }
        // stage('Package') {
        //     steps {
        //         sh 'mvn package -B -ntp -DskipTests'
        //     }
        // }
        // stage('SonarQube') {
        //     steps {
        //         withSonarQubeEnv('sonarqube'){
        //             sh 'env | sort'
        //             script {
        //                 if (env.CHANGE_ID) {
        //                     sh """
        //                         mvn sonar:sonar -B -ntp \
        //                         -Dsonar.pullrequest.key=${env.CHANGE_ID} \
        //                         -Dsonar.pullrequest.branch=${env.CHANGE_BRANCH} \
        //                         -Dsonar.pullrequest.base=${env.CHANGE_TARGET}
        //                     """
        //                 } else {
        //                     def branchName = GIT_BRANCH.replaceFirst('^origin/', '')
        //                     println "Branch name: ${branchName}"
        //                     sh "mvn sonar:sonar -B -ntp -Dsonar.branch.name=${branchName} -Dsonar.branch.target=${branchName}"
        //                 }
        //             }
        //         }
        //     }
        // }
        stage('DockerHub') {
            agent any
            options { skipDefaultCheckout() }
            steps {
                checkout scm
                sh 'docker --version'
                script {

                    def pom = readMavenPom file: 'pom.xml'
                    sh 'docker run --privileged --rm tonistiigi/binfmt --install all'
                    sh 'docker buildx create --use'
                    sh 'docker buildx inspect --bootstrap'

                    sh 'docker buildx version'

                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        sh """
                            docker buildx build \
                                -t danycenas/${pom.artifactId}:${pom.version} \
                                -t danycenas/${pom.artifactId}:latest \
                                --platform linux/amd64,linux/arm64 --push .
                        """
                    }

                }
            }
        }
    }
    // post {
    //     success {
    //         archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
    //     }
    //     cleanup {
    //         cleanWs()
    //     }
    // }
}