// Jenkinsfile for Liberty App - CI/CD
def templateName = 'gse-liberty'

openshift.withCluster() {
  env.NAMESPACE = openshift.project()
  env.APP_NAME = "${JOB_NAME}".replaceAll(/-build.*/, '')
  echo "Starting Pipeline for ${APP_NAME}..."
  env.BUILD = "${env.NAMESPACE}"
  env.DEV = "${APP_NAME}-dev"
  env.STAGE = "${APP_NAME}-stage"
  env.PROD = "${APP_NAME}-prod"
  
  
  env.EXTERNAL_IMAGE_REPO_URL = "harbor.jkwong.cloudns.cx"
  env.EXTERNAL_IMAGE_REPO_NAMESPACE = "roland-demo"
  env.EXTERNAL_IMAGE_REPO_CREDENTIALS = "harbor"
  env.DST_IMAGE = "${env.EXTERNAL_IMAGE_REPO_URL}/${env.EXTERNAL_IMAGE_REPO_NAMESPACE}/${env.APP_NAME}:${env.BUILD_NUMBER}"
  
}

pipeline {
  agent {
    label "maven"
  }
  stages {
    stage('preamble') {
        steps {
            script {
                openshift.withCluster() {
                    openshift.withProject() {
                        echo "Using project: ${openshift.project()}"
                        echo "APPLICATION_NAME: ${params.APPLICATION_NAME}"
                    }
                }
            }
        }
    }
    // Build Application using Maven
//    stage('Maven build') {
//      steps {
//        sh """
//        env
//        mvn -v 
//        cd CustomerOrderServicesProject
//        mvn clean package
//        """
//      }
//    }
//      
//    // Run Maven unit tests
//    stage('Unit Test'){
//     steps {
//        sh """
//        mvn -v 
//        cd CustomerOrderServicesProject
//        mvn test
//        """
//      }
//    }
      
    // Build Container Image using the artifacts produced in previous stages
//    stage('Build Liberty App Image'){
//     steps {
//        script {
//          // Build container image using local Openshift cluster
//          openshift.withCluster() {
//            openshift.withProject() {
//              timeout (time: 10, unit: 'MINUTES') {
//                // run the build and wait for completion
//                def build = openshift.selector("bc", "${params.APPLICATION_NAME}").startBuild("--from-dir=.")
//                                    
//                // print the build logs
//                build.logs('-f')
//              }
//            }        
//          }
//        }
//      }
//    } 
    
    
    stage ('Push Container Image') {
          agent {
            kubernetes {
              cloud 'openshift'
              label 'skopeo-jenkins'
              yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: jkwong/skopeo-jenkins
    tty: true
  serviceAccountName: jenkins
"""
            }
          }

          steps {
            script {
                openshift.withCluster() {
                    openshift.withProject() {
                        echo "Using project: ${openshift.project()}"
                        echo "APPLICATION_NAME: ${params.APPLICATION_NAME}"
                      
                      def openshift_token = readFile "/var/run/secrets/kubernetes.io/serviceaccount/token"
                      echo "Username: AFuser: ${env.AFuser}"
                      echo "Username: AFpasswd: ${env.AFpasswd}"
                      echo "Username: AFuser: ${params.AFuser}"
                      echo "Username: AFpasswd: ${params.AFpasswd}"
                //      echo "Username: AFuser: ${AFuser}"
                //      echo "Username: AFpasswd: ${AFpasswd}"
                      def openshift_token = readFile "/var/run/secrets/kubernetes.io/serviceaccount/token"
                      
                      withCredentials([usernamePassword(credentialsId: "${env.EXTERNAL_IMAGE_REPO_CREDENTIALS}", passwordVariable: 'username', usernameVariable: 'password')]) {
                              sh """
                              /usr/bin/skopeo copy \
                              --src-creds openshift:${openshift_token} \
                              --src-tls-verify=false \
                              --dest-creds ${AFuser}:${AFpassword} \
                              --dest-tls-verify=false \
                              docker://${env.BUILD}/${env.APP_NAME}:latest \
                              docker://vandepol/jenkins-test
                              """
                              println("Image is successfully pushed to https://${env.DST_IMAGE}")
                          }
                    }
                }
            }
        }
//                  def srcImage = ${env.BUILD}/${env.APP_NAME}
//
//                  openshift.withCluster() {
//                      openshift.withProject() {
//                         openshift.tag("${env.BUILD}/${env.APP_NAME}:latest", "${env.DEV}/${env.APP_NAME}:latest")
//                        def openshift_token = readFile "/var/run/secrets/kubernetes.io/serviceaccount/token"
//
 //                       println "source image: ${srcImage}, dest image: ${env.PROD}/${env.APP_NAME}:latest"
//
//                        withCredentials([usernamePassword(credentialsId: "${env.EXTERNAL_IMAGE_REPO_CREDENTIALS}", passwordVariable: 'AFpasswd', usernameVariable: 'AFuser')]) {
//                              sh """
//                              /usr/bin/skopeo copy \
//                              --src-creds openshift:${openshift_token} \
//                              --src-tls-verify=false \
//                              --dest-creds ${AFuser}:${AFpasswd} \
//                              --dest-tls-verify=false \
//                              docker://${srcImage} \
//                              docker://${env.DST_IMAGE}
//                              """
//                              println("Image is successfully pushed to https://${env.DST_IMAGE}")
//                          }
//                      }
//                  }
//              }
//          }
        }

  }
}
