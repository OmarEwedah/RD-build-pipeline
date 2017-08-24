node {
   //try {
     def commit_id
     stage('Source') {
       git url: 'https://olc.orange-labs.fr/gitblit/git/CRD/SonarTest.git', credentialsId: 'orange-gitblit'  
       sh "git rev-parse --short HEAD > .git/commit-id"                        
      commit_id = readFile('.git/commit-id').trim()                      
   }
   
     stage('build, package') {
       //sh "mvn clean package"
       sh "mvn clean install sonar:sonar -Dmaven.test.skip=true"
       input message: "Ready for Docker"
   }
   
     
      stage('sonar-scanner') {
            def sonarqubeScannerHome = tool name: 'sonar', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            withCredentials([string(credentialsId: 'sonar', variable: 'sonarLogin')]) {
              sh "${sonarqubeScannerHome}/bin/sonar-scanner -e -Dsonar.host.url=http://84.39.39.127:9000 -Dsonar.login=${sonarLogin} -Dsonar.projectName=SonarTest -Dsonar.projectVersion=${env.BUILD_NUMBER} -Dsonar.projectKey=GS -Dsonar.sources=/var/lib/jenkins/workspace/RD -Dsonar.tests=/var/lib/jenkins/workspace/RD -Dsonar.language=java"
            }
          }

     stage('docker build/push') {
       docker.withRegistry('https://index.docker.io/v1/', 'docker-hub') {
       def app = docker.build("omarewedah/build-test:${commit_id}", '.').push()
     }
   }
     
     stage('Deploy to test QA server') {
     }




    //} catch(e) {
    // mark build as failed
    //currentBuild.result = "FAILURE";
    // set variables
    //def content = '${JELLY_SCRIPT,template="html"}'

    // send email
     //emailext(body: content, mimeType: 'text/html',
         //subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} ${currentBuild.result}",
         //to: 'ewedah88@gmail.com', attachLog: true )

       // send slack notification
    //slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")

    // mark current build as a failure and throw the error
    //throw e;
  //}
}
