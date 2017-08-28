node {
   //try {
     def commit_id
     stage('Source') {
       git url: 'https://olc.orange-labs.fr/gitblit/git/CRD/phoneBook.git', credentialsId: 'orange-gitblit'  
       sh "git rev-parse --short HEAD > .git/commit-id"                        
      commit_id = readFile('.git/commit-id').trim()                      
   }
   
     stage('SonarQube') {
       sh "mvn clean compile test sonar:sonar"
      //sh "mvn clean compile sona"
       //input message: "Ready for Docker"
   }

   stage('build and package') {
       sh "mvn clean package"
       //input message: "Ready for Docker"
   }
   
     
      //stage('sonar-scanner') {
            //def sonarqubeScannerHome = tool name: 'sonar', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            //withCredentials([string(credentialsId: 'sonar', variable: 'sonarLogin')]) {
              //sh "${sonarqubeScannerHome}/bin/sonar-scanner -e -Dsonar.host.url=http://84.39.39.38:9000 -Dsonar.login=${sonarLogin} -Dsonar.projectName=phoneBook -Dsonar.projectVersion=${env.BUILD_NUMBER} -Dsonar.projectKey=GS -Dsonar.sources=/var/lib/jenkins/workspace/RD/src/main/ -Dsonar.tests=/var/lib/jenkins/workspace/RD/src/test/ -Dsonar.language=java"
            //}
          //}

     stage('docker build/push') {
       docker.withRegistry('https://index.docker.io/v1/', 'docker-hub') {
       def app = docker.build("omarewedah/build-test:${commit_id}", '.').push()
     }
   }
     
     node('ansbile') {
      stage('Deploy to test QA server') {
      ansiblePlaybook installation: 'ansible', inventory: '/home/jenkins/jenkins-ansible-deploy/inventory', playbook: '/home/jenkins/jenkins-ansible-deploy/docker.yml', sudo: true, sudoUser: 'ansible'

      }
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
