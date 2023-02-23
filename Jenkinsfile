def registry = 'https://nandana237.jfrog.io'
def imageName = 'nandana237.jfrog.io/val-twitttertrend-doc/valaxy-twittertrend'
def version   = '2.0.2'
pipeline{
    agent {
        node {
            label "jenkins-slave"
        }
    }
    environment {
        PATH = "/opt/apache-maven-3.8.7/bin:$PATH"
    }
    stages {
        stage('cloning the repository from gitHub') {
            steps {
                git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/Nandana237/valaxy-twittertrend.git'
            }
        }
        stage('build') {
            steps{
                echo "------------ build started ---------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "------------ build completed ---------"
            }
        }
        stage('Unit Test') {
            steps {
                echo '<--------------- Unit Testing started  --------------->'
                sh 'mvn surefire-report:report'
                echo '<------------- Unit Testing completed  --------------->'
            }
        }
        stage ("Sonar Analysis") {
            environment {
              scannerHome = tool 'nandana-sonarscanner'
            }
            steps {
                echo '<--------------- Sonar Analysis started  --------------->'
                withSonarQubeEnv('nandana-sonarqube-server') {    
                    sh "${scannerHome}/bin/sonar-scanner"
                echo '<--------------- Sonar Analysis completed  --------------->'
                }    
            }   
        }
        stage("Checking Quality Gate") {
            steps {
                script {
                  echo '<--------------- Sonar Gate Analysis Started --------------->'
                    timeout(time: 1, unit: 'HOURS'){
                      def qg = waitForQualityGate()
                        if(qg.status !='OK') {
                            error "Pipeline failed due to quality gate failures: ${qg.status}"
                        }
                    }  
                  echo '<--------------- Sonar Gate Analysis completed  --------------->'
                }
            }
        }
        stage("Publishing Jar to Jfrog-Maven-Repo") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory", credentialsId:"jfrog-access-credential"
                     def properties = "buildid=${env.BUILD_ID}, commitid=${env.GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "valaxy_twittertrend-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                        ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish completed --------------->'  
                }
            }   
        }
        stage("Building Docker Image ") {
          steps {
            script {
               echo '<--------------- Docker Build Started --------------->'
               app = docker.build(imageName+":"+version)
               echo '<--------------- Docker Image built --------------->'
            }
          }
        }
        stage ("Pushing Docker Image to Jfrog Artifactory"){
            steps {
                script {
                   echo '<--------------- Docker Publish Started --------------->'  
                   docker.withRegistry(registry, 'jfrog-access-credential'){app.push()}    
                   echo '<--------------- Docker Image Pushed --------------->'  
                }
            }
        }
        stage(" Deploy ") {
          steps {
            script {
               echo '<--------------- Deploy Started --------------->'
               sh './deploy.sh'
               echo '<--------------- Deploy Ends --------------->'
            }
          }
        }
    }
}
    
