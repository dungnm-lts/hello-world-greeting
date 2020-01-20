pipeline {
  agent {
    node {
      label 'docker'
    }

  }
  tools {
      nodejs 'node'
      maven 'M3'
  }

  stages {
    stage('Build') {
      steps {
        sh 'mvn clean verify  -DskipITs=true';
        junit '**/target/surefire-reports/TEST-*.xml'
        archive 'target/*.jar'
      }
    }
    stage('Scan') {
      steps {

          withSonarQubeEnv('sonarqube'){
            sh 'mvn clean verify sonar:sonar
            -Dsonar.projectName=hello-world-greeting
            -Dsonar.projectKey=hello-world-greeting
            -Dsonar.sources=src/main
            -Dsonar.tests=src/test
            -Dsonar.projectVersion=$BUILD_NUMBER';
          }
        }

    }
    stage('Integration Test') {
      steps {
         sh 'mvn clean verify -Dsurefire.skip=true';    
         junit '**/target/failsafe-reports/TEST-*.xml'           
         archive 'target/*.jar'
      }
    }

    stage ('Publish'){
      steps {
         script {
        def server = Artifactory.server 'my-artifactory'
        def uploadSpec = """{  
          "files": [    
            {       
              "pattern": "target/hello-0.0.1.war",       
              "target": "example-project/${BUILD_NUMBER}/",       
              "props": "Integration-Tested=Yes;Performance-Tested=No"    
            }  
          ]
        }"""
        server.upload(uploadSpec)
      }
      }
    }

  }
}
