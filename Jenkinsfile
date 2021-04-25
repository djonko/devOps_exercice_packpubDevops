node {
   def mvnHome
   stage('Prepare') {
      git url: 'git@github.com:djonko/devOps_exercice_packpubDevops.git', branch: 'develop'
      mvnHome = tool 'MAVEN_HOME'
   }
   stage('Build') {
      if (isUnix()) {
         sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
      } else {
         bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
      }
   }
   stage('Unit Test') {
      junit '**/target/surefire-reports/TEST-*.xml'
      archive 'target/*.jar'
   }
   stage('Integration Test') {
     if (isUnix()) {
        sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean verify"
     } else {
        bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean verify/)
     }
   }
   stage('Sonar') {
      if (isUnix()) {
         sh "'${mvnHome}/bin/mvn' sonar:sonar -Partifactory -Psonar"
      } else {
         bat(/"${mvnHome}\bin\mvn" sonar:sonar -Partifactory -Psonar/)
      }
   }
   stage('Deploy') {
        withCredentials([usernamePassword(credentialsId: 'tomcat_manager', usernameVariable: 'TM_USER', passwordVariable: 'TM_PWD')]) {
            sh '''
                response=$(curl -s -o /dev/null -w "%{http_code}\n" -u ${TM_USER}:${TM_PWD} -T target/**.war "http://tomcat9:8080/manager/text/deploy?path=/devops&update=true")
                if [ "$response" != "200" ]
                then
                 exit 1
                fi
            '''
        }
   }
   stage("Smoke Test"){
       sh "curl --retry-delay 10 --retry 5 http://tomcat9:8080/devops"
   }
}