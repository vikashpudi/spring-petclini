 pipeline {
    agent { label 'NODE1' }
    /* environment {
       JAVA_HOME = "/usr/lib/jvm/java-11-openjdk-amd64/"
      M2_HOME = "/usr/share/maven/"
        PATH = "$JAVA_HOME/bin:$PATH:$M2_HOME/bin"
    }*/
    //triggers { pollSCM '* * * * *' }
    parameters {  
                 choice(name: 'maven_goal', choices: ['install','package','clean install'], description: 'build the code')
                 choice(name: 'branch_to_build', choices: ['main', 'dev', 'ppm'], description: 'choose build')
                }
    stages {
        stage ('vcs') {
            steps {
                 git url: 'https://github.com/vikashpudi/spring-petclini.git', 
                 branch: 'main'
                //sh'mvn package'
            }
        }
    /*    stage("build & SonarQube analysis") {
            steps {
              sh' echo ***********SONAR SCANING************************'
              withSonarQubeEnv('sonarqube') {
                sh "mvn package sonar:sonar"
              }

            junit testResults: 'target/surefire-reports/*.xml'
            }
          }
          stage("Quality Gate") {
            steps {
              sh' echo ***********QUALITY GATE************************'
              timeout(time: 30, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
          } */

        stage ('Artifactory configuration') {
            steps {
              rtServer (
                  id: 'Artifactory',
                  url: 'https://beatyourlimits.jfrog.io/artifactory/',
                  credentialsId: 'jfrog',
                   bypassProxy: true,
                   timeout: 300
                       )
                sh' echo ***********JFROG CONGIG************************'
                rtMavenDeployer (
                    id: "spc_DEPLOYER",
                    serverId: "Artifactory",
                    releaseRepo: "demo",
                    snapshotRepo: "snapdemo"
                )
            }
        }
     stage ('Exec Maven') {
            steps {
              sh' echo **********SENDING TO ARTIFACTORY************************'
               rtMavenRun (
                    tool: "maven", // Tool name from Jenkins configuration
                     pom: "pom.xml",
                     goals: "clean install ",
                     deployerId: "spc_DEPLOYER"
                 )
                 
                }
                }
stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "Artifactory"
                )
            }
        }

        stage ('Build docker image') {
            steps {//sh 'curl -u "pudivikash:Devops@123456" -X GET https://beatyourlimits.jfrog.io/artifactory/demo/org/springframework/samples/spring-petclinic/2.7.4/spring-petclinic-2.7.4.jar --output spring-petclinic-2.7.4.jar '
               sh"mv /home/murali/remot/workspace/SPRINGPET/target/spring-petclinic-2.7.4.jar spring-petclinic-2.7.4.jar "
               sh "docker image build -t beatyourlimits/spc:${BUILD_ID} ."
            }
        }
        stage ('docker Artifactory configuration') {
            steps {
                rtServer (
                    id: "ARTIFACTORY_SERVER",
                    url: "https://beatyourlimits.jfrog.io/artifactory/api/docker/" ,
                    credentialsId: "jfrog "
                )
            }
        }

         stage ('Push image to Artifactory') {
            steps {
              
              script{
              //  def image = "spc:${BUILD_ID}"
              def app
                app = docker.build  "mydockerrepo/spc:${BUILD_ID}"
                docker.withRegistry('https://beatyourlimits.jfrog.io/artifactory/mydockerrepo', 'jfrog') {            
				        app.push("${env.BUILD_NUMBER}")
	        }    
              }
                 /* 
              //withCredentials([usernameColonPassword(credentialsId: 'jfrog', variable: 'jfrogcred')]) 
              {
     docker login
                }
                sh"docker push beatyourlimits/spc:${BUILD_ID} "
                 rtDockerPush(
                    serverId: "ARTIFACTORY_SERVER",
                    image: "docker image build -t  beatyourlimits/spc:${BUILD_ID}" ,
                   host: 'tcp://20.163.205.39:5000',
                     targetRepo: 'docker-local'
                )*/
            }
        }

        stage ('depolying to ') {
            steps {
              sh'kubectl apply -f depolyments/spc.yaml'
              sh'kubectl get svc'
            }} 

    }

}