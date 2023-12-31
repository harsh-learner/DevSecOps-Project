pipeline {
    agent any

    tools{
        jdk  'jdk17'
        nodejs 'node16'
        
    }
    
    environment{
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('CleanWorkspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout'){
            steps{
                git branch: 'devops', url: "https://github.com/harsh-learner/DevSecOps-Project.git"
            }
        }
        
        stage('SonarQubeScanning'){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
    
                }
            }
        }
        
        stage('QualityGateCheck'){
            steps{
                waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
            }
        }
        
        stage('Install Dependencies'){
            steps{
                sh 'npm install'
            }
        }
        
        stage('OwaspDependencyCheck'){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy FS'){
            steps{
                sh 'trivy fs . > trivyfs.txt'
            }
        }
        
        stage('Docker Build & Push'){
            steps{
                script{
                      withDockerRegistry(credentialsId: 'docker', toolName:'docker')  {
                         sh "docker build --build-arg TMDB_V3_API_KEY=77cac3265a3814777acf10119a8a3116 -t netflix ."
                         sh "docker tag netflix learnerharsh/netflix:latest"
                         sh "docker push learnerharsh/netflix:latest"
                    
                        }
              
                }
            }
        }
        
        stage('Trivy'){
            steps{
                sh 'trivy image learnerharsh/netflix:latest > trivyimage.txt'
            }
        }
    }
    
}
