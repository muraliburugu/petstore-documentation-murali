pipeline{
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages{
        stage ('clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage ('checkout scm') {
            steps {
                git 'https://github.com/muraliburugu/jpetstore-6.git'
            }
        }
        stage('Prepare Maven Wrapper') {
            steps {
                script {
                    // Regenerate the Maven Wrapper
                    sh 'mvn -N io.takari:maven:wrapper'
                }
            }
        }
        stage ('maven compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage ('maven Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petshop \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petshop '''
                }
            }
        }
        stage("quality gate"){
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
           }
        }
        stage ('Build war file'){
            steps{
                sh 'mvn clean install -DskipTests=true'
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage ('Build and push to docker hub'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t petshop ."
                        sh "docker tag petshop muraliburugu507/petshop:latest"
                        sh "docker push muraliburugu507/petshop:latest"
                   }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image muraliburugu507/petshop:latest > trivy.txt"
            }
        }
        stage("Trigger deployment"){
            steps{
                  // Trigger the deployment pipeline and wait for it to complete
                  build job: 'CD-petshop', wait: true
             }
         }
    }
    post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'muraliburugu507@gmail.com',
            attachmentsPattern: 'trivy.txt'
        }
    }
}
