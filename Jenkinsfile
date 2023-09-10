rtMaven = null
server= null
pipeline {
  agent any

  tools {
    maven 'Maven3'
  }
  stages {
    stage ('Build') {
      steps {
      sh 'mvn clean install -f MyWebApp/pom.xml'
      }
    }
    stage ('Code Quality') {
      steps {
        withSonarQubeEnv('SonarQube') {
        sh 'mvn -f MyWebApp/pom.xml sonar:sonar'
        }
      }
    }
    stage ('JaCoCo') {
      steps {
      jacoco()
      }
    }
    stage ('Artifactory Upload') {
      steps {
        script {
          server = Artifactory.server('My_Artifactory')
          rtMaven = Artifactory.newMavenBuild()
          rtMaven.tool = 'Maven3'
          rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local', server: server
          rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot', server: server
          rtMaven.deployer.deployArtifacts = false // Disable artifacts deployment during Maven run
          buildInfo = Artifactory.newBuildInfo()
          rtMaven.run pom: 'MyWebApp/pom.xml', goals: 'install', buildInfo: buildInfo
          rtMaven.deployer.deployArtifacts buildInfo
          server.publishBuildInfo buildInfo
        }
      }
    }
    stage ('DEV Deploy') {
      steps {
      echo "deploying to DEV Env "
      deploy adapters: [tomcat8(credentialsId: '268c42f6-f2f5-488f-b2aa-f2374d229b2e', path: '', url: 'http://localhost:8090')], contextPath: null, war: '**/*.war'
      }
    }
    stage ('Slack Dev Notification') {
      steps {
        echo "deployed to DEV Env successfully"
        slackSend(channel:'your slack channel_name', message: "Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
    }
    stage ('QA Approve') {
      steps {
        echo "Taking approval from QA manager"
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you want to proceed to QA Deploy?', submitter: 'admin,manager_userid'
        }
      }
    }
    stage ('QA Deploy') {
      steps {
        echo "deploying to QA Env "
        deploy adapters: [tomcat8(credentialsId: '268c42f6-f2f5-488f-b2aa-f2374d229b2e', path: '', url: 'http://your_dns_name:8090')], contextPath: null, war: '**/*.war'
        }
    }
    stage ('Slack QA Notification') {
      steps {
        echo "Deployed to QA Env successfully"
        slackSend(channel:'your slack channel_name', message: "Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
    }
  }
}
