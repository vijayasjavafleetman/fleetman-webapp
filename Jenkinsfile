node {

   def mvnHome
   def pom
   def artifactVersion
   def tagVersion
   def retrieveArtifact
   def name="webapp"
   def SERVICE_NAME
   def NREPOSITORY_TAG
   def appname
    
   stage('Prepare') {
      appname='webapp'
   }

   stage('Checkout') {
      checkout scm

      def props = readProperties file: 'webapp.properties'
      artifactVersion = props.version

      REPOSITORY_TAG="${env.DOCKERHUB_USERNAME}/${env.ORGANIZATION_NAME}-${name}:${artifactVersion}.${env.BUILD_ID}"
      echo "${REPOSITORY_TAG}"
      echo "artifact version : ${artifactVersion}"

   }
 
   if(env.BRANCH_NAME ==~ /release.*/){

      artifactVersion = artifactVersion.replace("-SNAPSHOT", "")
      tagVersion = 'v'+artifactVersion
      
      stage("QA Approval"){
         echo "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) is waiting for input. Please go to ${env.BUILD_URL}."
         input 'Approval for QA Deploy?';
      }

      stage('SSH transfer to QA') {
         script {
            sshPublisher(
               continueOnError: false,
               failOnError: true,
               publishers: [
                  sshPublisherDesc(
                     configName: "ansibleserver",
                     verbose: true,
                     transfers: [
                        sshTransfer(
                           sourceFiles: "",
                           removePrefix: "",
                           remoteDirectory: "",
                           execCommand: "cd /home/ansadmin/jenkins/${appname}-qa/workspace;rm -f *;"
                        ),
                        sshTransfer(
                           execTimeout: 999999,
                           sourceFiles: "dist,nginx.conf.j2,Dockerfile,docker-entrypoint.sh,${appname}-build-playbook-qa.yaml",
                           removePrefix: "",
                           remoteDirectory: "${appname}-qa/workspace",
                           execCommand: "cd /home/ansadmin/jenkins/${appname}-qa/workspace;ansible-playbook -i /home/ansadmin/jenkins/${appname}-qa/hostconfig/hosts -u ansadmin -e tag=${REPOSITORY_TAG} /home/ansadmin/jenkins/${appname}-qa/workspace/${appname}-build-playbook-qa.yaml;"
                        )
                     ]
                  )
               ]
            )
         }
      }

      withEnv(["REPOSITORY_TAG=${REPOSITORY_TAG}"]){
         stage('Deploy to QA Cluster') {
            sh 'envsubst < ${WORKSPACE}/deploy.yaml > ${WORKSPACE}/udeploy.yaml'
            sh 'cat ${WORKSPACE}/udeploy.yaml'

            stage('SSH transfer') {
               script {
                  sshPublisher(
                     continueOnError: false,
                     failOnError: true,
                     publishers: [
                        sshPublisherDesc(
                           configName: "ansibleserver",
                           verbose: true,
                           transfers: [
                              sshTransfer(
                                 sourceFiles: "",
                                 removePrefix: "",
                                 remoteDirectory: "",
                                 execCommand: "cd /home/ansadmin/jenkins/${appname}-qa/workspace;rm -f *;"
                              ),
                              sshTransfer(
                                 execTimeout: 999999,
                                 sourceFiles: "udeploy.yaml,${appname}-deployment-playbook-qa.yaml",
                                 removePrefix: "",
                                 remoteDirectory: "${appname}-qa/workspace",
                                 execCommand: "cd /home/ansadmin/jenkins/${appname}-qa/workspace;ansible-playbook -i /home/ansadmin/jenkins/${appname}-qa/hostconfig/hosts -u ansadmin  /home/ansadmin/jenkins/${appname}-qa/workspace/${appname}-deployment-playbook-qa.yaml;"
                              )
                           ]
                        )
                     ]
                  )
               }
            }
         }
      }
   }

   if(env.BRANCH_NAME == 'master'){
   }

   if(env.BRANCH_NAME == 'develop'){

      stage('SSH transfer') {
         script {
            sshPublisher(
               continueOnError: false,
               failOnError: true,
               publishers: [
                  sshPublisherDesc(
                     configName: "ansibleserver",
                     verbose: true,
                     transfers: [
                        sshTransfer(
                           sourceFiles: "",
                           removePrefix: "",
                           remoteDirectory: "",
                           execCommand: "cd /home/ansadmin/jenkins/${appname}/workspace;rm -f *;"
                        ),
                        sshTransfer(
                           execTimeout: 999999,
                           sourceFiles: "dist,nginx.conf.j2,Dockerfile,docker-entrypoint.sh, ${appname}-build-playbook.yaml",
                           removePrefix: "",
                           remoteDirectory: "${appname}/workspace",
                           execCommand: "cd /home/ansadmin/jenkins/${appname}/workspace;ansible-playbook -i /home/ansadmin/jenkins/${appname}/hostconfig/hosts -u ansadmin -e tag=${REPOSITORY_TAG} /home/ansadmin/jenkins/${appname}/workspace/${appname}-build-playbook.yaml;"
                        )
                     ]
                  )
               ]
            )
         }
      }

      withEnv(["REPOSITORY_TAG=${REPOSITORY_TAG}"]) {
         stage('Deploy to Cluster') {
            sh 'envsubst < ${WORKSPACE}/deploy.yaml > ${WORKSPACE}/udeploy.yaml'
            sh 'cat ${WORKSPACE}/udeploy.yaml'

            stage('SSH transfer') {
               script {
                  sshPublisher(
                     continueOnError: false,
                     failOnError: true,
                     publishers: [
                        sshPublisherDesc(
                           configName: "ansibleserver",
                           verbose: true,
                           transfers: [
                              sshTransfer(
                                 sourceFiles: "",
                                 removePrefix: "",
                                 remoteDirectory: "",
                                 execCommand: "cd /home/ansadmin/jenkins/${appname}/workspace;rm -f *;"
                              ),
                              sshTransfer(
                                 execTimeout: 999999,
                                 sourceFiles: "udeploy.yaml,${appname}-deployment-playbook.yaml",
                                 removePrefix: "",
                                 remoteDirectory: "${appname}/workspace",
                                 execCommand: "cd /home/ansadmin/jenkins/${appname}/workspace;ansible-playbook -i /home/ansadmin/jenkins/${appname}/hostconfig/hosts -u ansadmin  /home/ansadmin/jenkins/${appname}/workspace/${appname}-deployment-playbook.yaml;"
                              )
                           ]  
                        )
                     ]
                  )
               }
            }
         }
      }
   }   
}
