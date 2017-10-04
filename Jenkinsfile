import groovy.json.JsonSlurperClassic
@NonCPS
def getMicroServicesConfig(def url){

  def result = new URL(url).getText()


  def microConfigDict = new JsonSlurperClassic().parseText(result)
  def keys = microConfigDict.keySet() as List
  def microConfigs = []

  for (key in keys){
    microConfigs << microConfigDict[key]

  }
  return microConfigs
}
node{
    try {


  
    def repos = getMicroServicesConfig("http://18.221.3.67:8083/jenkinsbuild-all.json")
    

    for(repo in repos){
      
        
      def commit_id
        dir (repo["serviceName"]){
            
         stage('Checkout') {
           git url: repo["gitRepo"], credentialsId: 'orange-gitblit'  //param
           sh "git rev-parse --short HEAD > .git/commit-id"                        
          commit_id = readFile('.git/commit-id').trim()  
          repo."commitId" = commit_id
          }
        }
       


      dir (repo["serviceName"]){
        stage('SonarQube Clean, Compile, Test') {
  
               withSonarQubeEnv('sonar') {
              sh "mvn clean compile test sonar:sonar"
           }
           
                  timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
                    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                    print  qg
                  }
                
        }
          
       stage('Maven Cleaning, Package') {
           sh "mvn clean package -Dclass.lines.covered.ratio=${repo["classLinesCoveredRatio"]}"
           archiveArtifacts 'target/*.jar'
       
       }
       
        stage('Jacoco') {
           jacoco classPattern: '**/target/classes', execPattern: '**/target/jacoco.exec'
          }
      

       
     }
      }
      
      for(repo in repos){
    
            dir (repo["serviceName"]){
                stage('Docker Build, Push') {
                     docker.withRegistry('https://index.docker.io/v1/', 'docker-hub') {
                     def app = docker.build("${repo["dockerImageName"]}:${repo["commitId"]}", '.').push() //param
                   }
                }
            }
            
      }
     
     input message: "Ready to deploy to QA server??"
      node('ansbile') {
         
                  stage('Deploy To  QA Server') {
                    // def image_name
                    for(repo in repos){
                       def image_name = "${repo["dockerImageName"]}:${repo["commitId"]}"
                       sh "ansible-playbook /home/jenkins/jenkins-ansible-deploy/docker.yml -i /home/jenkins/jenkins-ansible-deploy/inventory --user=ansible --extra-vars ansible_sudo_pass=12345 --extra-vars CONTAINER_NAME=${repo["serviceName"]} --extra-vars CONTAINER_PORT=${repo["containerPort"]} --extra-vars IMAGE_NAME=${image_name} --extra-vars ACTIVE_PROFILE=${repo["activeProfile"]} --extra-vars CONFIG_SERVER_URL=${repo["configServerUrl"]} --extra-vars EUREKA_INSTANCE_HOSTNAME=${repo["eurekaInstanceHostname"]} --extra-vars EUREKA_INSTANCE_NONSECUREPORT=${repo["eurekaInstanceNonsecureport"]}"
                  
                    } 
                      
                  }
   }
      input message: "Do you want run Robot ?"
      node('slave1'){
          git url: 'https://olc.orange-labs.fr/gitblit/git/CRD/functional-tests.git', credentialsId: 'orange-gitblit'
             sh 'pybot -d out Tests/AddPhoneBook.robot'
      }
     
  } catch (err) {
        //notify("ERROR!: ${err}")
        currentBuild.result = 'FAILURE'
    }

      finally {
        node('slave1'){
        step([$class: 'RobotPublisher', 
    outputPath:'out', 
    passThreshold: 0, 
    unstableThreshold: 0, 
    otherFiles: "", 
    logFileName: 'log.html', 
    outputFileName: 'output.xml', 
    reportFileName: 'report.html'])
        }
      
  }

}  

    } catch(e) {
    // mark build as failed
    currentBuild.result = "FAILURE";
    // set variables
    def content = '${JELLY_SCRIPT,template="html"}'

    // send email
     emailext(body: content, mimeType: 'text/html',
         subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} ${currentBuild.result}",
         to: 'oewedah.ext@orange.com', attachLog: true )

       // send slack notification
    //slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")

    // mark current build as a failure and throw the error
    //throw e;
  //}
}
