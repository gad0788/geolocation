pipeline {
   agent {
        docker { image 'maven:3.8.6-openjdk-11-slim' }
    }
   tools{
     maven 'M2_HOME'
   }
   environment {
        registry = '257070983248.dkr.ecr.us-east-1.amazonaws.com/utc-app-qa'
        registryCredential ='aws_ecr_id'
        dockerimage = ''
   }
   stages{
        stage('maven build & SonarQube analysis'){
            steps{
                withSonarQubeEnv('SonarServer'){
                sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=gad0788_geo-app'
                }
                }
            }
        }
            stage('Check Quality Gate') {
            steps {
                echo 'Checking quality gate...'
                script {
                    timeout(time: 20, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline stopped because of quality gate status: ${qg.status}"
                        }
                    }
                    stage('maven package') {
                        steps {
                          sh 'mvn clean'
                          sh 'mvn install -DskipTests'
                          sh 'mvn package -DskipTests'
                          }
                     }
                  }
         }
        stage('upload artifacts Nexus'){
            steps{
                script{
                def mavenPom = readMavenPom file: 'pom.xml'
                nexusArtifactUploader artifacts: [[artifactId: "${mavenPom.artifactId}",
                classifier: '',
                file: "target/${mavenPom.artifactId}-${mavenPom.version}.${mavenPom.packaging}",
                type: "${mavenPom.packaging}"]], credentialsId: 'NexusID', groupId: "${mavenPom.groupId}",
                nexusUrl: '72.14.182.149:8081', nexusVersion: 'nexus3', protocol: 'http',
                repository: 'biom', version: "${mavenPom.version}"
                }
            }
        }
        stage('Build Image'){
            steps{
                script{
                    def mavenPom = readMavenPom file: 'pom.xml'
                    dockerImage = docker.build registry + ":${mavenPom.version}"
                }
            }
            stage('Deploy Image'){
              steps{
                script{
                    docker.withRegistry("https://"+registry,"ecr:us-east-1:"+registryCredential){
                        dockerImage.push()
                    }
                }
            }
            
        }
     }
}