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
              //sh "${sonarqubeScannerHome}/bin/sonar-scanner -e -Dsonar.host.url=http://84.39.39.38:9000 -Dsonar.login=${sonarLogin} -Dsonar.projectName=phoneBook -Dsonar.projectVersion=${env.BUILD_NUMBER} -Dsonar.projectKey=GS -Dsonar.sources=/var/lib/jenkins/workspace/RD/src/main/ -Dsonar.tests=/var/lib/jenkins/workspace/RD/src/test/ -Dsonar.language=java -Dsonar.java"
            //}
          //}

           stage('Jacoco fail') {
       jacoco changeBuildStatus: true, classPattern: '**/target/classes', deltaBranchCoverage: '50', deltaClassCoverage: '50', deltaComplexityCoverage: '50', deltaInstructionCoverage: '50', deltaLineCoverage: '50', deltaMethodCoverage: '50', execPattern: '**/target/jacoco.exec', maximumBranchCoverage: '100', maximumClassCoverage: '100', maximumComplexityCoverage: '100', maximumInstructionCoverage: '100', maximumLineCoverage: '100', maximumMethodCoverage: '100', minimumBranchCoverage: '40', minimumClassCoverage: '40', minimumComplexityCoverage: '40', minimumInstructionCoverage: '40', minimumLineCoverage: '40', minimumMethodCoverage: '40'
   }

     stage('docker build/push') {
       docker.withRegistry('https://index.docker.io/v1/', 'docker-hub') {
       def app = docker.build("omarewedah/build-test:${commit_id}", '.').push()
       input message: "Ready to deploy to QA server??"
     }
   }
     
     node('ansbile') {
      stage('Deploy to test QA server') {
      sh 'ansible-playbook /home/jenkins/jenkins-ansible-deploy/docker.yml -i /home/jenkins/jenkins-ansible-deploy/inventory --user=ansible --extra-vars "ansible_sudo_pass=12345"'
      }
     }

     node('slave1') {
      stage('runnung tests with RobotFramework') {
        git url: 'https://olc.orange-labs.fr/gitblit/git/CRD/functional-tests.git', credentialsId: 'orange-gitblit'
        sh 'pybot -d output Tests/'
    }
      step([$class: 'RobotPublisher',
        outputPath: 'output',
        passThreshold: 20,
        unstableThreshold: 80,
        otherFiles: ""])
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
