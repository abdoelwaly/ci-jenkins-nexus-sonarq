# jpro App Jenkins Pipeline 🚀

This project defines a Jenkins pipeline for building, testing, analyzing, and deploying the jpro application. It integrates with Maven, SonarQube, and Nexus, running within an AWS environment. ☁️

## Architecture

![Architecture Diagram](https://github.com/abdoelwaly/ci-jenkins-nexus-sonarq/blob/0573498c003e60985d94c182afa9b3dccffeb4b2/architectures/WhatsApp%20Image%202025-02-07%20at%2018.16.18_56b175b4.jpg) 

This diagram illustrates the flow of the pipeline and the interaction between the different components. ⚙️ It visually represents the AWS infrastructure hosting the Jenkins, Nexus, and SonarQube servers. 🖥️

## Pipeline Stages

The pipeline consists of the following stages:

1.  **Build:** Compiles the application using Maven, skipping tests. 🛠️ Archives the generated WAR file. 📦
2.  **Test:** Executes unit tests using Maven. ✅
3.  **Checkstyle Analysis:** Performs code style analysis using Checkstyle. 🧐
4.  **Sonar Analysis:** Analyzes the code for bugs, vulnerabilities, and code smells using SonarQube. 🐞
5.  **Quality Gate:** Waits for the SonarQube quality gate to pass. 🚦 Fails the pipeline if the gate fails. 🛑
6.  **UploadArtifact:** Uploads the built WAR file to the Nexus repository. ⬆️
7.  **Slack Integration:** Slack notification to slack workspace

## Jenkins Configuration

Before running this pipeline, you need to configure the following in your Jenkins server:

1.  **Install Plugins:** Ensure you have the following Jenkins plugins installed:
    *   Maven Integration plugin
    *   SonarQube Scanner plugin
    *   Nexus Artifact Uploader plugin
    *   Slack Notification plugin (optional, for notifications) 🔔
    *   Credentials Binding Plugin

2.  **Configure Tools:**
    *   **Maven:** Define a Maven tool installation named "MAVEN3.9.9" pointing to your Maven installation directory.
    *   **JDK:** Define a JDK tool installation named "JDK17" pointing to your JDK 17 installation directory.
    *   **SonarQube Scanner:** Define a SonarQube Scanner tool installation named "sonarscanner" pointing to your SonarQube scanner installation directory.

3.  **Manage Credentials:**
    *   **Nexus Credentials:** Create a new "Username with password" credential named "nexuslogin" with the username "admin" and the password you set for your Nexus server (e.g., "Nexuspassword"). *Important:  Do NOT hardcode passwords in your Jenkinsfile.* ⚠️ Use Jenkins credentials.
    *   **SonarQube Token:** Create a new "Secret text" credential containing your SonarQube token. 🔑 You'll reference this in the `withSonarQubeEnv` step.

4.  **Configure SonarQube Server:**
    *   In Jenkins global tool configuration, add a SonarQube Server configuration and name it "sonarserver". Configure the URL to your SonarQube server and select the credential you created for the SonarQube token.

5.  **Configure Nexus Server:**
    *   No specific configuration is needed in Jenkins for the Nexus server other than the credentials. The connection parameters are passed via environment variables in the pipeline.

6.  **Slack Integration:**
    *   Configure the Slack notification plugin with your Slack webhook URL. 💬

## AWS Environment Setup

1.  **EC2 Instances:** Launch EC2 instances for Jenkins, Nexus, and SonarQube. ☁️ Ensure that these instances can communicate with each other (e.g., within the same security group). 🌐 The Nexus server's private IP address (e.g., `172.31.43.144`) should be accessible from the Jenkins server.
2.  **Install Software:** Install the necessary software (Java, Maven, SonarQube, Nexus, Jenkins) on their respective instances. 📦
3.  **Network Configuration:** Ensure that the necessary ports (e.g., 8080 for Jenkins, 8081 for Nexus, 9000 for SonarQube) are open in the security groups. 🔒
4.  **Nexus Configuration:** Configure Nexus with the repositories defined in the pipeline (`jpro-snapshot`, `jpro-release`, `jpro-maven-central`, `jpro-maven-group`).
5.  **SonarQube Configuration:** Configure SonarQube and create the project "jprofile".
6.  **Jenkins Server Setup:**  Install Jenkins and configure it as described in the "Jenkins Configuration" section above.

## GitHub Webhook Integration for Automated Triggers

This project leverages GitHub webhooks to automate the CI/CD pipeline.  By configuring a webhook in your GitHub repository, Jenkins is automatically notified of any pushes (or other specified events) to the repository, triggering a new build. This eliminates the need for manual build triggering or polling, ensuring continuous integration and faster feedback.

**How it Works:**

1.  **GitHub Configuration:** A webhook is set up in the GitHub repository's settings, pointing to the Jenkins server's webhook endpoint, A secret is also configured for enhanced security.

2.  **Jenkins Configuration:** The Jenkins job is configured to listen for GitHub webhook events.  The same secret configured in GitHub is also configured in the Jenkins job.

3.  **Code Push:** When a developer pushes code to the GitHub repository, GitHub sends a webhook event (a POST request with information about the push) to the Jenkins server.

4.  **Jenkins Trigger:** Jenkins receives the webhook event, verifies the secret (if configured), and if valid, triggers the pipeline build.

5.  **CI/CD Process:** The pipeline then proceeds with the defined stages (build, test, analyze, deploy, etc.).

## Running the Pipeline

1.  Create a new Pipeline job in Jenkins.
2.  Point the pipeline to this Jenkinsfile (either from SCM or by copying the script).
3.  Trigger the build. ▶️


## Artifacts

The pipeline generates a WAR file named `jprofile-v2.war` (adjust the name if needed) which is then uploaded to the Nexus repository. ⬆️

This README provides a comprehensive guide for setting up and running the jpro app Jenkins pipeline. 👍 Remember to replace placeholder values with your actual configuration details. 📌