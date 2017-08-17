node {
   //try {
     def commit_id
     stage('Source') {
       git url: 'https://olc.orange-labs.fr/gitblit/git/CRD/phoneBook.git', credentialsId: 'orange-gitblit'  
       sh "git rev-parse --short HEAD > .git/commit-id"                        
      commit_id = readFile('.git/commit-id').trim()                      
   }
   
     stage('build, package') {
       sh "mvn clean package"
       input message: "Ready for Docker"
   }
   
     stage('docker build/push') {
       docker.withRegistry('https://index.docker.io/v1/', 'docker-hub') {
       def app = docker.build("omarewedah/build-test:${commit_id}", '.').push()
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
