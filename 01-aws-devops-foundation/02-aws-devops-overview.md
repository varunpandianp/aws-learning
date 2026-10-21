# AWS Cloud & DevOps Overview

## 1. Introduction to DevOps

DevOps is a combination of practices, processes, culture, and tools that helps development and operations teams collaborate to deliver software faster, more reliably, and securely.

### Objectives of DevOps

- Automate software build, testing, and deployment.
- Reduce manual work and human errors.
- Improve collaboration between development and operations teams.
- Deliver application updates frequently and safely.
- Monitor application health and resolve failures quickly.
- Maintain reliable, scalable, and secure infrastructure.

### What Is AWS DevOps?

AWS DevOps combines DevOps practices with AWS cloud services to automate infrastructure provisioning, application delivery, configuration management, monitoring, and operations.

Examples of AWS services:
- Amazon EC2 — virtual servers.
- Amazon VPC — network isolation.
- Amazon S3 — object storage.
- AWS IAM — identity and access management.
- Amazon ECR — Docker image registry.
- Amazon EKS — managed Kubernetes.
- Amazon CloudWatch — monitoring and logging.
- AWS CloudFormation — infrastructure as code.

---

## 2. DevOps Lifecycle

The DevOps lifecycle is a continuous process.

**Plan → Code → Build → Test → Release → Deploy → Operate → Monitor → Feedback**

Infrastructure provisioning, configuration management, security, and automation support multiple stages throughout this lifecycle.

---

## 3. DevOps Tools: Stage-by-Stage Overview

### Stage 1: Plan

**Purpose:** Plan features, tasks, requirements, and development work.

Tools:
- Jira
- Trello
- Azure Boards
- GitHub Issues

Example:
A team plans a new login feature and creates development tasks.

### Stage 2: Code

**Purpose:** Write and maintain application source code.

Tools:
- VS Code
- IntelliJ IDEA
- PyCharm
- Vim

Languages and frameworks:
- Java
- Python
- JavaScript
- Spring Boot
- Flask
- Node.js

Example:
A developer writes application code for a web application.

### Stage 3: Version Control

**Purpose:** Track source code changes and collaborate with other developers.

Tools:
- Git
- GitHub
- GitLab
- Bitbucket

Important Git concepts:
- Repository
- Commit
- Branch
- Merge
- Pull request
- Rebase
- Revert
- Tags

Workflow:

1. Developer modifies source code.
2. Developer commits changes.
3. Developer pushes code to GitHub.
4. A pull request is reviewed and merged.
5. The CI/CD pipeline is triggered.

### Stage 4: Build

**Purpose:** Compile source code, resolve dependencies, and generate a deployable application artifact.

Tools:
- Maven — commonly used for Java projects.
- Gradle — build automation, including Java projects.
- npm — JavaScript dependency and script management.
- Make — build automation for various projects.

Example Maven commands:

```bash
mvn clean
mvn compile
mvn test
mvn package
```

A successful Java build may generate:

```text
target/application.jar
```

**Output:** A packaged application artifact ready for testing or deployment.

### Stage 5: Code Quality and Security

**Purpose:** Identify bugs, maintainability problems, vulnerable dependencies, and security issues.

Tools:
- SonarQube — code quality and static analysis.
- Checkstyle — Java code style.
- SpotBugs — potential Java defects.
- OWASP Dependency-Check — dependency vulnerability analysis.
- Trivy — container image and configuration scanning.

These tools can run as automated pipeline checks. The exact tools depend on project requirements.

### Stage 6: Testing

**Purpose:** Verify that the application works correctly before release.

Types of testing:

- Unit testing — tests individual functions or components.
- Integration testing — tests interactions between components.
- API testing — tests application APIs.
- End-to-end testing — tests complete user workflows.
- Smoke testing — verifies critical functionality after deployment.
- Performance testing — evaluates speed and capacity.

Tools:
- JUnit
- Mockito
- pytest
- Postman
- Newman
- Selenium
- JMeter

Example:

```bash
mvn test
```

If a required test fails, the pipeline should stop before promoting the release.

### Stage 7: Artifact Storage

**Purpose:** Store versioned build artifacts so they can be retrieved and deployed consistently.

Tools:
- Sonatype Nexus Repository
- JFrog Artifactory
- Amazon S3
- Amazon ECR — container image registry.

Examples:

```text
application-1.0.0.jar
application-1.0.1.jar
application-1.0.2.jar
```

Typical flow:

1. Jenkins builds the application.
2. Automated tests pass.
3. Jenkins publishes the artifact to Nexus or Artifactory.
4. A deployment stage retrieves the approved artifact.

For Docker applications, Jenkins can build an image and push it to Amazon ECR.

**Important:** Nexus/Artifactory and ECR serve related but different purposes. Nexus and Artifactory support multiple artifact formats; ECR is a managed container image registry.

### Stage 8: CI/CD Automation

**Purpose:** Automate integration, validation, release, and deployment.

Tools:
- Jenkins
- GitHub Actions
- GitLab CI/CD
- AWS CodePipeline
- AWS CodeBuild
- AWS CodeDeploy

CI means Continuous Integration:
- Detect source changes.
- Build the application.
- Run tests and quality checks.
- Publish build artifacts.

CD can mean Continuous Delivery or Continuous Deployment:
- Continuous Delivery keeps releases ready for controlled deployment, often with manual production approval.
- Continuous Deployment automatically deploys changes that pass the required checks.

Example Jenkins pipeline:

```text
GitHub Push
    |
    v
Jenkins Pipeline
    |
    v
Checkout Source Code
    |
    v
Build with Maven
    |
    v
Run Tests and Quality Checks
    |
    v
Publish Artifact
    |
    v
Deploy to Test Environment
    |
    v
Smoke Tests and Validation
    |
    v
Production Approval
    |
    v
Deploy to Production
```

### Stage 9: Infrastructure as Code (IaC)

**Purpose:** Create and manage infrastructure using version-controlled configuration files instead of relying on manual console operations.

Tools:
- Terraform
- AWS CloudFormation
- AWS CDK
- Pulumi

Terraform is commonly used to provision AWS resources.

Examples:
- VPC
- Public and private subnets
- Route tables
- Internet Gateway
- NAT Gateway
- Security groups
- EC2 instances
- Application Load Balancer
- Auto Scaling groups
- EKS clusters
- IAM resources

Example Terraform commands:

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

Typical Terraform workflow:

1. Write infrastructure configuration.
2. Initialize providers and modules.
3. Validate the configuration.
4. Review the proposed changes.
5. Apply the approved changes.
6. Track infrastructure changes through version control and state management.

**Where does Terraform fit?**

Terraform is not limited to one CI/CD stage. It is used to provision the environment that applications will run on and can also be integrated into CI/CD workflows.

For example:

```text
Terraform
    |
    v
Create AWS VPC and EC2 Instances
    |
    v
Prepare Test and Production Infrastructure
    |
    v
Deploy Applications Through CI/CD
```

Infrastructure changes should be reviewed and controlled separately from routine application releases when appropriate.

### Stage 10: Configuration Management

**Purpose:** Configure servers and maintain consistent software and operating-system settings.

Tools:
- Ansible
- Chef
- Puppet
- Salt

Ansible is commonly used for:
- Installing Java, Docker, and Nginx.
- Creating users and groups.
- Configuring Linux servers.
- Managing systemd services.
- Deploying application files.
- Updating configuration files.
- Applying consistent settings across multiple servers.

Example Ansible commands:

```bash
ansible --version
ansible-inventory --list
ansible-playbook -i inventory.ini site.yml
```

**Where does Ansible fit?**

Ansible generally operates after infrastructure exists, although it can also orchestrate broader workflows. It configures the resources that Terraform provisions.

Example:

```text
Terraform
    |
    v
Provision EC2 Instances
    |
    v
Ansible
    |
    v
Install Java and Nginx
Configure Services and Directories
    |
    v
Jenkins
    |
    v
Deploy Application Artifact
```

Jenkins can invoke Ansible playbooks as part of the deployment pipeline.

### Stage 11: Containerization

**Purpose:** Package an application with its runtime dependencies into a portable container image.

Tools:
- Docker
- Podman
- Amazon ECR
- Docker Hub

Important Docker concepts:
- Dockerfile
- Image
- Container
- Registry
- Volume
- Network
- Docker Compose

Example:

```bash
docker build -t myapp:1.0 .
docker run -d -p 8080:8080 myapp:1.0
```

Typical flow:

1. Build the application artifact.
2. Create a Docker image.
3. Test the image.
4. Push the image to ECR.
5. Deploy the image to an EC2 server or Kubernetes cluster.

Docker packages and runs applications. It does not replace Terraform or Ansible.

### Stage 12: Container Orchestration

**Purpose:** Manage containers across a cluster of machines.

Tools:
- Kubernetes
- Amazon EKS
- Amazon ECS
- Docker Swarm
- Helm

Kubernetes provides:
- Deployment management.
- Service discovery and load balancing.
- Horizontal scaling.
- Rolling updates.
- Self-healing mechanisms.
- Secret and configuration integration.

Helm packages and manages Kubernetes applications using charts.

Example deployment flow:

```text
Jenkins
   |
   v
Build and Test Application
   |
   v
Build Docker Image
   |
   v
Push Image to Amazon ECR
   |
   v
Deploy Image to Amazon EKS
   |
   v
Kubernetes Runs Application Pods
```

Kubernetes is useful when the application needs container orchestration. It is not mandatory for every DevOps project; a single EC2 deployment may be sufficient for simpler applications.

### Stage 13: Deployment and Release Management

**Purpose:** Deliver approved application versions to target environments.

Deployment targets:
- AWS EC2 instances
- Amazon ECS
- Amazon EKS
- AWS Elastic Beanstalk
- Other cloud or on-premises servers

Deployment strategies:
- Rolling deployment
- Blue-green deployment
- Canary deployment
- Recreate deployment

Supporting tools:
- Jenkins
- Ansible
- AWS CodeDeploy
- Kubernetes
- Argo CD
- Helm

Example environments:

```text
Development
    |
    v
Testing / QA
    |
    v
Staging
    |
    v
Production
```

Not every organization uses all four environments. The exact setup depends on the team's release process.

### Stage 14: Monitoring, Logging, and Alerting

**Purpose:** Understand application health, infrastructure performance, failures, and operational issues.

Metrics and dashboards:
- Prometheus
- Grafana
- Amazon CloudWatch

Logs:
- Amazon CloudWatch Logs
- Elastic Stack (ELK)
- Loki

Tracing:
- OpenTelemetry
- Jaeger
- AWS X-Ray

Alerting:
- Grafana Alerting
- Alertmanager
- Amazon CloudWatch Alarms
- Amazon SNS for notifications

Common monitoring metrics:
- CPU utilization
- Memory utilization
- Disk space
- Network traffic
- HTTP response codes
- Request latency
- Application error rates
- Container restarts
- Kubernetes pod health

Example:

```text
AWS EC2 / Application / Kubernetes
              |
              v
       Metrics and Logs
              |
              v
 Prometheus / CloudWatch / Loki
              |
              v
          Grafana
              |
              v
     Dashboards and Alerts
```

Monitoring feeds back into development and operations, helping teams identify incidents and improve future releases.

### Stage 15: Operations and Continuous Improvement

**Purpose:** Maintain reliability and improve the delivery process.

Activities:
- Incident response.
- Root-cause analysis.
- Rollback and recovery.
- Capacity planning.
- Cost optimization.
- Security patching.
- Disaster recovery.
- Post-incident reviews.

Tools may include:
- Jira or other incident-management systems.
- AWS CloudWatch.
- Grafana and Alertmanager.
- AWS Systems Manager.
- AWS Backup.
- Runbooks and automation scripts.

Operational feedback helps teams improve the next planning and development cycle.

---

## 4. Terraform vs Ansible

Terraform and Ansible solve different but complementary problems.

| Area | Terraform | Ansible |
|---|---|---|
| Primary purpose | Provision infrastructure | Configure systems |
| Main focus | Infrastructure resources | Operating systems and software |
| Typical input | HCL configuration | YAML playbooks |
| Example | Create EC2 instances | Install Java on EC2 |
| Example | Create a VPC and subnets | Configure Nginx |
| Example | Create an EKS cluster | Install software or configure supporting hosts |
| State | Maintains Terraform state | Typically uses inventory and execution results rather than Terraform-style infrastructure state |
| Common execution | Before or during environment provisioning | After resources are reachable, or as part of orchestration |

### Practical Example

Suppose a company needs two EC2 instances:

- Test server
- Production server

**Step 1: Terraform**

Creates the VPC, networking, security groups, and EC2 instances.

**Step 2: Ansible**

Connects to the instances and installs the required runtime, configures services, and prepares deployment directories.

**Step 3: Jenkins**

Builds and tests the application, then deploys the approved artifact.

**Step 4: Monitoring**

CloudWatch or Prometheus collects operational metrics, and alerts notify the team of problems.

This separation improves repeatability, maintainability, and troubleshooting.

---

## 5. End-to-End AWS DevOps Architecture

```text
Developer
   |
   v
Git / GitHub
   |
   v
Jenkins CI/CD
   |
   +--> Build: Maven / Gradle
   |
   +--> Test: JUnit / pytest
   |
   +--> Quality: SonarQube
   |
   +--> Security: Trivy / dependency scanning
   |
   v
Artifact Repository
Nexus / Artifactory / Amazon ECR
   |
   v
Deployment Pipeline
   |
   +--> Test Environment
   |       |
   |       v
   |    Validation
   |
   +--> Approval
           |
           v
      Production
           |
           v
    EC2 / ECS / EKS
           |
           v
  Monitoring and Logging
           |
           v
 CloudWatch / Prometheus /
 Grafana / Loki
           |
           v
   Alerts and Feedback
```

Infrastructure and server preparation support the architecture:

```text
Terraform
   |
   v
AWS Infrastructure
VPC / Subnets / EC2 / ALB / EKS
   |
   v
Ansible (where required)
   |
   v
Server Configuration
```

Security, IAM, networking, secrets management, and access controls apply throughout the entire architecture.

---

## 6. Complete DevOps Tool Mapping

| Lifecycle stage | Common tools | Main purpose |
|---|---|---|
| Plan | Jira, GitHub Issues | Work planning |
| Code | VS Code, IntelliJ | Write code |
| Version control | Git, GitHub | Track and share code |
| Build | Maven, Gradle, npm | Build application |
| Code quality | SonarQube | Analyze source code |
| Testing | JUnit, pytest, Postman | Verify functionality |
| Security | Trivy, OWASP Dependency-Check | Find vulnerabilities |
| Artifact storage | Nexus, Artifactory, S3 | Store build artifacts |
| Container registry | Amazon ECR | Store container images |
| CI/CD | Jenkins, GitHub Actions | Automate delivery |
| Infrastructure as Code | Terraform, CloudFormation | Provision resources |
| Configuration management | Ansible | Configure servers |
| Containerization | Docker | Package and run applications |
| Orchestration | Kubernetes, EKS, ECS | Run and manage workloads |
| Packaging | Helm | Manage Kubernetes releases |
| GitOps | Argo CD, Flux | Reconcile deployments from Git |
| Cloud infrastructure | AWS EC2, VPC, ALB | Host and network applications |
| Secrets and identity | AWS IAM, Secrets Manager | Control access and secrets |
| Monitoring | Prometheus, CloudWatch | Collect metrics |
| Visualization | Grafana | Dashboards and alerts |
| Logging | CloudWatch Logs, ELK, Loki | Search and analyze logs |
| Tracing | OpenTelemetry, Jaeger | Trace distributed requests |
| Operations | AWS Systems Manager, runbooks | Maintain and troubleshoot systems |

These tools are examples, not a requirement to install every tool in one project.

---

## 7. Recommended Learning Order for an AWS DevOps Engineer

### Phase 1: Prerequisites
1. Linux fundamentals.
2. Networking: IP addressing, DNS, routing, ports, and firewalls.
3. Git and GitHub.
4. Basic Bash scripting.
5. Python fundamentals.

### Phase 2: Build and CI/CD
6. Maven or another build tool.
7. Unit testing fundamentals.
8. Jenkins installation and pipelines.
9. GitHub webhooks and automated builds.
10. Artifact repositories.

### Phase 3: AWS Cloud
11. IAM.
12. VPC, subnets, route tables, and security groups.
13. EC2 and SSH.
14. Load balancers and Auto Scaling.
15. S3, CloudWatch, and ECR.

### Phase 4: Infrastructure Automation
16. Terraform fundamentals and state.
17. Terraform modules and remote state.
18. Ansible inventory and playbooks.
19. Ansible roles and variables.
20. Terraform plus Ansible automation.

### Phase 5: Containers and Orchestration
21. Dockerfiles, images, containers, volumes, and networks.
22. Docker Compose.
23. Kubernetes fundamentals.
24. Amazon EKS or ECS.
25. Helm and deployment strategies.

### Phase 6: Production Readiness
26. Monitoring with CloudWatch or Prometheus and Grafana.
27. Centralized logging.
28. Security scanning and secrets management.
29. Health checks and rollback strategies.
30. End-to-end portfolio projects.

---

## 8. Recommended Hands-On Portfolio Project

**Project: Automated AWS CI/CD Pipeline with Jenkins, Terraform, Ansible, and Docker**

### Architecture

- GitHub stores application source code and the Jenkinsfile.
- Jenkins builds the application and runs automated tests.
- Terraform provisions AWS infrastructure.
- Ansible configures EC2 servers where needed.
- Docker packages the application.
- Amazon ECR stores the Docker image.
- Jenkins deploys the image to the test environment.
- Automated health checks validate the release.
- Production deployment requires approval.
- CloudWatch collects logs and metrics.

### Project workflow

1. Create a GitHub repository.
2. Develop a small Flask or Spring Boot application.
3. Write a Dockerfile.
4. Build and test the application through Jenkins.
5. Write Terraform code to provision AWS resources.
6. Use Ansible to configure EC2 servers if using an EC2-based deployment.
7. Push the Docker image to ECR.
8. Deploy the image to the test environment.
9. Run a health check.
10. Promote the same validated image to production after approval.
11. Monitor the application using CloudWatch.
12. Document rollback and troubleshooting steps.

### Expected learning outcomes

- Understand the full DevOps lifecycle.
- Build an automated Jenkins pipeline.
- Provision AWS infrastructure using Terraform.
- Configure Linux servers using Ansible.
- Containerize applications with Docker.
- Deploy to AWS environments.
- Apply testing, security, monitoring, and release controls.

---

## 9. Key Takeaways

- Git manages source code history and collaboration.
- Maven or Gradle builds the application.
- Testing and security checks validate the build.
- Nexus or Artifactory stores application artifacts; ECR stores container images.
- Jenkins automates and orchestrates CI/CD.
- Terraform provisions infrastructure.
- Ansible configures servers and software.
- Docker packages applications into containers.
- Kubernetes or ECS manages container workloads.
- CloudWatch, Prometheus, Grafana, and logging tools help monitor production.
- IAM, secrets management, and secure networking protect the entire system.

**Remember:** DevOps is not about using the maximum number of tools. It is about choosing the right tools, automating repeatable work, delivering reliable releases, and continuously improving the system.
