def registry = 'https://nithyasree.jfrog.io'
// def imageName = 'nithyasree.jfrog.io/namg-docker-local/namtrend'
// def version   = '2.1.3'

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
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"eyJ2ZXIiOiIyIiwidHlwIjoiSldUIiwiYWxnIjoiUlMyNTYiLCJraWQiOiJudTZOS2VUUkFRTmR2S05LbjdOOVpudkk4OG9yNERPdzN5REhGcmNyTVdFIn0.eyJzdWIiOiJqZmFjQDAxaGV5cmZ2dHM0bTdzMHhydmVqdGcxeGQwL3VzZXJzL25pdGh5YXNyZWU3MzczQGdtYWlsLmNvbSIsInNjcCI6ImFwcGxpZWQtcGVybWlzc2lvbnMvYWRtaW4iLCJhdWQiOiIqQCoiLCJpc3MiOiJqZmZlQDAxaGV5cmZ2dHM0bTdzMHhydmVqdGcxeGQwIiwiaWF0IjoxNjk5ODkyNjQ4LCJqdGkiOiJhMjA0YTBkNy0xYzYxLTRiMGMtYmMyOC1hNmFkMjcyM2JhZWQifQ.RAk4BltsU8Qb7eovxjaJRsuAX6fOTxyfDPya0JJXHDn_R6kt0x2hC7B5ArpWETLukaYp5RW467R_LT2XdbjFzzJteY_MIr4PkqrIL5DDpjAZwOl6pJWLAUgqn_rNOpvbHxf4X7-K-U-FmwMdfiFi8oRu3NoJ4n22SjaPs1rSdVeQglKOgSm9BX59Ta-NRZqR4uKXJ_Et_5DVhuE51T6lukWmY2cJPex1fOo1mmIVejCB2aJsl99oh9DjNnf2WxbDy4ERdWItxkszx8aA4yFTOiBu7lmDSTt5E00fh3I9SsxR8Wn94LB51voYh4bYzuBf0ZRn7Sn1M1K2kagd3hSWeg"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "namg-libs-release-local/{1}",
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
