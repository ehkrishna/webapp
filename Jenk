pipeline{
   agent any
  tools {
   maven 'maven' 
  }
  stages{
    stage('Build') {
      sh 'mvn clean package'
    }
  }
}
