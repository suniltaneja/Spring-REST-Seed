node {

   // Mark the code checkout 'stage'....
   stage 'Checkout'

   // Get some code from a GitHub repository
   git credentialsId: '3642aacf-de9b-4112-8865-d196ef212782', url: 'gazelle@mdlgd1bl2.africanbank.net:africanbank/AfricanBank-RulesEngine.git'


   // Get the maven tool.
   // ** NOTE: This 'M3' maven tool must be configured
   // **       in the global configuration.           
   def mvnHome = tool 'Maven'
   def javaHome = tool 'Java'

   // Mark the code build 'stage'....
   stage 'Build'
   // Run the maven build
   withEnv(["JAVA_HOME=${javaHome}", 'PATH=$PATH:$JAVA_HOME/bin']) {
     sh "${mvnHome}/bin/mvn -f SpringRESTRule/pom.xml clean package -Dmaven.test.skip=true"
   }
   
   stage 'Test'
   // Run the maven build
   withEnv(["JAVA_HOME=${javaHome}", 'PATH=$PATH:$JAVA_HOME/bin']) {
      parallel xunit: {
        sh "${mvnHome}/bin/mvn -f SpringRESTRule/pom.xml  surefire-report:report"
      }, coverage: {
        sh "${mvnHome}/bin/mvn -f SpringRESTRule/pom.xml  cobertura:cobertura"
      },
      failFast: false

      step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar,**/target/*.war,**/target/site/**,**/target/surefire-reports/*', fingerprint: true])
      step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
   }
   
   stage 'Deploy to Dev'
   // todo: replace this with ansible once we work that out.
   sh "cp SpringRESTRule/target/*.jar /home/gazelle/deploy/RulesEngine/latest.jar"
   
   stage 'Integration Test'
   echo 'IT happens here'

   stage 'Load Test'
     withEnv(["JAVA_HOME=${javaHome}", 'PATH=$PATH:$JAVA_HOME/bin']) {
       sh "${mvnHome}/bin/mvn -f SpringRESTRule/pom.xml gatling:execute -Dgatling.runMultipleSimulations=true"
     } 
     step([$class: 'ArtifactArchiver', artifacts: '**/target/gatling/**', fingerprint: true])
     step([$class: 'io.gatling.jenkins.GatlingPublisher', enabled: true])

   stage 'Official Signoff'
   echo 'The idea is that relative stakeholders can tick a box to say they agree that this can be deployed'

   stage 'Deploy to production'
   echo 'Deploy to prod'

   
}
