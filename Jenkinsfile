pipeline {
  agent any

tools { 
        maven 'M2_HOME'
        jdk 'jdk' 
  }
  
  
  stages {
  stage ('Versioning') {
        steps {
             script {
    // read info from pom (see: http://maven.apache.org/components/ref/3.3.9/maven-model/apidocs/org/apache/maven/model/Model.html)
    pom = readMavenPom file: 'pom.xml'
    printf("Version: %s", pom.version)
        
    // list modules
    printf ("Modules: %s", pom.getModules().join(","))
        
    // set version explicit in maven pom, always add build number
    // format of version should be: <name>-x.y.x-<branch?>-<build>
    //version = getVersion(pom)
        versions = pom.version
    withMaven(jdk: 'jdk', maven: 'M2_HOME') {
        sh "mvn versions:set -DnewVersion=${versions}"
    }               

    // either release, develop or feature(default)
    printf("Version set to: %s", versions)
               
    IMAGE = readMavenPom().getArtifactId()
    VERSION = readMavenPom().getVersion()
    artifactPath= "target/"  + IMAGE + "-" + VERSION + "-mule-application.jar"

    echo "IMAGE: ${IMAGE}"
    echo "VERSION: ${VERSION}"
       echo "artifactPath: ${artifactPath}"
              
    }
       //     sh "mvn build-helper:parse-version versions:set -DnewVersion='2.3.0' versions:commit"
      }
    }
    stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
       }
    }
    stage('Unit Test') {
      steps {

        sh 'mvn clean test'
      }
    }
    stage('Deploy ARM') { 
      environment {
        ANYPOINT_CREDENTIALS = credentials('anypoint.credentials') 
      }
      steps {
       
        sh 'mvn deploy -P arm -Darm.target.name=mule-default4.3-dev3 -Dmule.artifact=${artifactPath} -Danypoint.username=${ANYPOINT_CREDENTIALS_USR}  -Danypoint.password=${ANYPOINT_CREDENTIALS_PSW} -e' 
      }
    }
    
  }
}
