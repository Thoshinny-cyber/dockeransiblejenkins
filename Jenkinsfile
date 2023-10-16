pipeline{
    agent any
    tools {
      maven 'maven'
    }
    environment {
      DOCKER_TAG = getVersion()
      SNYK_API= env.SNYK_API_TOKEN
    }
    stages{
        stage('SCM'){
            steps{
                git credentialsId: 'github', 
                    url: 'https://github.com/Thoshinny-cyber/dockeransiblejenkins.git'
            }
        }
        stage ('Check-Git-Secrets') {
      steps {
        sh "rm trufflehog || true"
        sh "docker pull gesellix/trufflehog"
        sh "docker run gesellix/trufflehog --json https://github.com/Thoshinny-cyber/dockeransiblejenkins.git > trufflehog"
        sh "cat trufflehog"
      }
    }
    //      stage ('Source Composition Analysis') {
    //   steps {
    //      sh 'rm owasp* || true'
    //      sh 'wget "https://raw.githubusercontent.com/Thoshinny-cyber/dockeransiblejenkins/master/owasp-dependency-check.sh" '
    //      sh 'chmod +x owasp-dependency-check.sh'
    //      sh 'bash owasp-dependency-check.sh'
    //      sh 'cat /var/lib/jenkins/Java_pipeline/reports/dependency-check-report.xml'
        
    //   }
    // }
        stage('SAST') {
      steps {
        echo 'Testing...'
        snykSecurity(
          snykInstallation: 'Snyk',
          snykTokenId: SNYK_API
          // place other parameters here
        )
      }
    }        
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
        }
        
        stage('Docker Build'){
            steps{
                sh "docker build . -t thoshinny/javaapp:${DOCKER_TAG} "
            }
        }
        
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker_hub1', variable: 'dockerHubPwd')]) {
                    sh "docker login -u thoshinny -p ${dockerHubPwd}"
                }
                
                sh "docker push thoshinny/javaapp:${DOCKER_TAG} "
            }
        }
        
        stage('Docker Deploy'){
            steps{
              ansiblePlaybook credentialsId: 'dev-server', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
