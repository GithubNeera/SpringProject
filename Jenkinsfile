env.mvnHome = '/usr/share/maven'
node('mavenlabel') {
   
   stage('Preparation') { // for display purpos
     
      git 'https://github.com/jglick/simple-maven-project-with-tests.git'
        
      
   }
   stage('Build') {
      
      if (isUnix()) {
         sh "'${mvnHome}/bin/mvn' clean install"
      } else {
         bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
      }
   }
   stage('Results') {
      junit '**/target/surefire-reports/TEST-*.xml'
      archive 'target/*.jar'
   }
   

}
