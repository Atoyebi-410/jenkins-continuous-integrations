# Continuous Integration Pipeline with Jenkins, Nexus, SonarQube, and Slack on AWS

## Project Overview
This project implements a Continuous Integration (CI) pipeline that automates code building, testing, quality analysis, artifact management, and notification processes. The pipeline is designed to execute for every code commit, ensuring faster detection and resolution of issues, and maintaining high code quality standards.

## Key Objectives and Solutions
1. **Automated Build and Test**: Trigger builds for each commit, compile code, run tests, and package artifacts automatically.
2. **Centralized Artifact Repository**: Store generated artifacts in a Nexus repository for easy management and deployment.
3. **Automated Notifications**: Send build status notifications to Slack to keep the team updated in real-time.
4. **Early Issue Detection**: Conduct code quality analysis using SonarQube, allowing developers to fix issues immediately.

## Technologies and Tools Used
- **Jenkins**: Orchestrates the CI pipeline.
- **Nexus**: Stores and manages build artifacts.
- **SonarQube**: Analyzes code quality.
- **Slack**: Sends build status notifications.
- **AWS EC2**: Hosts Jenkins, Nexus, and SonarQube servers.
- **Maven**: Manages dependencies and builds Java applications.
- **GitHub Webhook**: Triggers Jenkins pipeline on code commits.

## Infrastructure Setup
Three AWS EC2 instances were configured, each running a dedicated tool:

1. **Jenkins Server** (Ubuntu, t2.medium)
   - Security Group: Jenkins-SG
   - Inbound Rules: SSH (port 22), HTTP (port 8080)

2. **Nexus Server** (Amazon Linux 2023, t2.medium)
   - Security Group: Nexus-SG
   - Inbound Rules: SSH (port 22), HTTP (port 8081)

3. **SonarQube Server** (Ubuntu, t2.medium)
   - Security Group: Sonar-SG
   - Inbound Rules: SSH (port 22), HTTP (port 80)

![AWS EC2 Instances](path/to/your/aws-ec2-instances-screenshot.png)

## Pipeline Workflow and Explanation

The pipeline follows these main stages:

### Build
- **Description**: Maven builds the Java project, packaging it into a `.war` file.
- **Command**: `mvn -s settings.xml -DskipTests install`
- **Outcome**: If successful, the build artifact (`.war` file) is archived within Jenkins.

![Jenkins Build Job](path/to/your/jenkins-build-job-screenshot.png)
![Pipeline Stages in Jenkins](path/to/your/pipeline-stages-screenshot.png)

### Unit Test
- **Description**: Runs unit tests on the project.
- **Command**: `mvn -s settings.xml test`
- **Outcome**: Confirms the project’s basic functionality and reliability.

### Code Quality Analysis with SonarQube
- **Description**: Executes a static code analysis to detect code smells, bugs, and vulnerabilities.
- **Command**:
  ```sh
  ${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
  -Dsonar.projectName=vprofile \
  -Dsonar.projectVersion=1.0 \
  -Dsonar.sources=src/ \
  -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
  -Dsonar.junit.reportsPath=target/surefire-reports/ \
  -Dsonar.jacoco.reportsPath=target/jacoco.exec \
  -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml


- **Outcome**: Generates a detailed report, accessible in the SonarQube dashboard.

### Quality Gate
- **Description**: Uses SonarQube’s quality gate to determine if the code meets predefined quality standards.
- **Command**: waitForQualityGate abortPipeline: true
- **Outcome**: If the code fails the quality gate, the pipeline halts, notifying the team to address issues.

### Artifact Upload to Nexus
- **Description**: Uploads the built artifact to a Nexus repository for centralized management.
- **Code**:
  ```sh
  nexusArtifactUploader(
    nexusVersion: 'nexus3',
    protocol: 'http',
    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
    groupId: 'QA',
    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
    repository: "${RELEASE_REPO}",
    credentialsId: "${NEXUS_LOGIN}",
    artifacts: [
        [artifactId: 'vproapp',
        classifier: '',
        file: 'target/vprofile-v2.war',
        type: 'war']
    ]
  )

- **Outcome**: Artifacts are stored in Nexus and versioned by build timestamp for easy access and traceability.

### Slack Notification

- **Description**: Sends the build status to a Slack channel, notifying the team.
- **Code**:
  ```sh
  slackSend channel: '#jenkinscicd',
           color: COLOR_MAP[currentBuild.currentResult],
           message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"

- **Outcome**: Real-time notifications about the build status keep the team updated.

## Deployment and Configuration Steps
### Jenkins Setup

- **Installed required plugins**: Maven Integration, GitHub Integration, Nexus Artifact Uploader, SonarQube Scanner, Slack Notification, Build Timestamp.
- Configured Maven and JDK paths for Oracle JDK8 and JDK11.
- Added SonarQube and Slack credentials for integration.

### Nexus Setup

- Created repositories for release, snapshot, and proxy storage:
    - vprofile-release: Maven2 (hosted)
    - vprofile-snapshot: Maven2 (hosted)
    - vpro-maven-central: Maven2 (proxy)
    - vpro-maven-group: Maven2 (group)

### SonarQube Setup

- Generated a SonarQube token and configured it in Jenkins for authentication.

### Slack Setup

- Integrated Jenkins with a dedicated Slack workspace and channel.
    Configured a webhook in GitHub to trigger the pipeline on every commit.