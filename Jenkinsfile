def registry = 'https://nithyasree.jfrog.io'
def imageName = 'nithyasree.jfrog.io/namg-docker-local/namtrend'
def version   = '2.1.3'

pipeline {
    agent {label 'maven'}


environment {
    PATH = "/opt/apache-maven-3.9.4/bin:$PATH"
    }
    stages {

       stage("build"){
        steps {
            echo "------- build started --------"
            sh 'mvn clean deploy -Dmaven.test.skip=true'
            echo "---------build completed ----------"
        }
       
    }
    stage("test"){
        steps{
            echo "--------unit test started ---------"
            sh 'mvn surefire-report:report'
            echo "---------unit test completed -------"
        }
    }

    stage('SonarQube analysis'){
    environment {
      scannerHome = tool 'namg-sonar-scanner' //sonar scanner name should be same as what we have defined in the tools
    }
    steps {                                 // in the steps we are adding our sonar cube server that is with Sonar Cube environment.
    withSonarQubeEnv('namg-sonarqube-server') {
       sh "${scannerHome}/bin/sonar-scanner" // This is going to communicate with our sonar cube server and send the analysis report.
        }
      }
    }
    
    stage("Quality Gate") {
        steps {
            script {
            timeout(time: 1, unit: 'HOURS') {
                def qg = waitForQualityGate()
                if (qg.status != 'OK') {
                    error " Pipeline aborted due to qualitygate failure: ${qg.status}"
                }
            }
        }
    }
}
    stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrogartifact-credentials"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "sunny-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
            }
        }   
    }   

    stage(" Docker Build ") {
        steps {
            script {
               echo '<--------------- Docker Build Started --------------->'
               app = docker.build(imageName+":"+version)
               echo '<--------------- Docker Build Ends --------------->'
            }
          }
        }
    
    stage (" Docker Publish "){
        steps {
            script {
                echo '<--------------- Docker Publish Started --------------->'  
                docker.withRegistry(registry, 'jfrogartifact-credentials'){
                app.push()
                }    
                echo '<--------------- Docker Publish Ended --------------->'  
                }
            }
        }

    stage ("Deploy"){
        steps {
          script {
            sh './deploy.sh'
          }  
        }
    }

}
}
