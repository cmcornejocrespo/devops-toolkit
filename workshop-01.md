# HP DevOps Toolkit

## Workshop 1 - Jenkins

### Setup

- Download [Jenkins](http://mirrors.jenkins.io/war-stable/latest/jenkins.war)
- Copy the password from the console ouput or from:

```sh
#check java version 8
java -version

#run jenkins
java -jar jenkins.war

#copy password
$USER/.jenkins/secrets/initialAdminPassword
```

- Select plugins to install, None and then select:
  - Pipeline
  - Pipeline: GitHub Groovy Libraries
  - Pipeline: Stage View
  - Cobertura
  - JUnit
  - Git
- Continue as admin
- Save and Finish
- Start using Jenkins

### Scenario 1: Run your first pipeline job

- New Item >> Pipeline >> My first Pipeline >> OK
- Scroll down to Pipeline section and select from try sample pipeline dropdown .. Github + Maven
- Save
- Build now
- Check error by clicking on the red ball
- Check ERROR: No tool named M3 found
- Click on Jenkins banner
- Manage Jenkins >> Global Tool Configuration
- Scroll down to Maven >> Add Maven >> Name: M3 >> Install automatically is checked >> Save
- Click on Jenkins banner
- Select My first Pipeline Job >> Build Now

### Scenario 2: Create a CI pipeline

- New Item >> Pipeline >> My first CI Pipeline >> OK
- Scroll down to Pipeline section and paste [this](https://raw.githubusercontent.com/cmcornejocrespo/devops-training-material/develop/jenkins/Jenkinsfile) or this:

```yml

node {
    def mvnHome = tool 'M3'

    stage('Preparation') { // for display purposes
        // Get some code from a GitHub repository
        git url: 'https://github.com/cmcornejocrespo/webinar-bat-desk.git', branch: 'feature/jbcnconf-2017'
        // Get the Maven tool.
        // ** NOTE: This 'M3' Maven tool must be configured
        // **       in the global configuration.

    }
    stage('Build') {
        // Run the maven build
        sh "'${mvnHome}/bin/mvn' clean package"
    }

    stage('Run unit tests') {
        // Run the maven test
        sh "'${mvnHome}/bin/mvn' test"
    }

    stage('Run Sonar reports') {
        // Run the maven test
        echo("Sonar reports successfully run");
    }

    stage('Run Integration Tests') {
        // Run integration tests
        sh "'${mvnHome}/bin/mvn' clean verify -Pintegration-tests"
    }

    stage('Results') {
        junit '**/target/surefire-reports/TEST-*.xml'
        archiveArtifacts 'bat-desk-common/target/*.jar'
    }
}
```

- Save
- Build Now
- Click on Latest Test Results

### Scenario 3: Create a dummy CI/CD pipeline

- New Item >> Pipeline >> My first CICD Pipeline >> OK
- Scroll down to Pipeline section and paste [this](https://raw.githubusercontent.com/cmcornejocrespo/devops-training-material/develop/jenkins/Jenkinsfile.complete.pipeline) ot this:

```yml

node {

    stage('Preparation') {
        // for display purposes
        echo("Preparation successfully run");
    }
    stage('Build') {
        // Run the maven build
        echo("Build successfully run");
    }
    stage('Run unit tests') {
        // Run the maven test
        echo("unit tests successfully run");
    }
    stage('Run Sonar reports') {
        // Run the maven test
        echo("Sonar reports successfully run");
    }
   stage('Run Integration Tests') {
      // Run integration tests
      echo("Integration Tests successfully run");
   }
    stage('Create Docker Image and Push to Registry') {
        // Run the maven test
        echo("Create Docker Image and Push to Registry successfully run");
    }
    stage('Deploy to Integration Environment') {
        // Run the maven test
        echo("Deploy to Integration Environment successfully run");
    }
    stage('Run e2e Tests') {
        // Run the maven test
        echo("e2e Tests successfully run");
    }
    stage('Deploy to Nexus') {
        // Run the maven test
        echo("Deploy to Nexus successfully run");
    }
}
```

- Save
- Build Now