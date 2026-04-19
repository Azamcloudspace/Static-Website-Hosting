#  AWS Static Website Hosting with Multi-Environment CI/CD

##  Project Summary

Designed and implemented a fully automated, production-ready static website hosting solution on AWS using serverless services. The system leverages a multi-environment CI/CD architecture to separate infrastructure provisioning from application content deployment, ensuring scalable, repeatable, and controlled releases across development, staging, and production environments.

---

##  Key Outcomes

- Built a serverless static website hosting architecture using S3 and CloudFront  
- Implemented global content delivery with low latency via CDN  
- Designed multi-environment deployment strategy (dev, staging, prod)  
- Automated infrastructure provisioning using CloudFormation  
- Developed CI/CD pipelines for both infrastructure and application deployment  
- Implemented cache invalidation strategy for real-time content updates  

---

##  Architecture Overview

The system follows a serverless web hosting and deployment model:


User → Route 53 → CloudFront → S3 (Static Website)

![Architecture](/screenshots/Architecture.jpeg)

---

##  System Design

### Content Hosting
- **Amazon S3**
  - Stores static website files (`index.html`, assets)  
  - Acts as origin for CloudFront  

---

### Content Delivery
- **Amazon CloudFront**
  - Distributes content globally  
  - Provides HTTPS via ACM integration  
  - Caches content for performance  

---

### DNS Management
- **Amazon Route 53**
  - Routes domain traffic to CloudFront distribution  

---

### Security
- **AWS Certificate Manager (ACM)**
  - Provides SSL/TLS certificates for HTTPS  

---

##  CI/CD Architecture 

This project uses a **two-tier pipeline architecture**:

---

### 1. Infrastructure Pipeline

Responsible for provisioning environments and resources.

**Workflow:**

CodePipeline → GitHub(source) → CodeBuild(Build) → CloudFormation → Environment Deployment

**Capabilities:**

**Codepipeline**

- Integrates with services such as GitHub and CodeBuild
- Automatically triggers and executes workflows
- Uses manual approval gates to control promotion between stages
- Implements properly structured IAM permissions across services

**GitHub**

- Repository (Source)

**CodeBuild**

Mutiple Codebuild services are used for the different environments with their individual builspec.yml i.e `dev-buidspec.yml`, `staging-buildspec.yml`, and
`prod-buildspec.yml` respectively , their capabilities are :

- Uploads all nested CloudFormation templates to an S3 bucket for stack referencing
- Deploys the `cloudformation/master/master-stack.yaml` template
- Implements properly structured IAM permissions across services

**Cloudformation**

- Provisions `cloudformation/master/master-stack.yaml` template child stacks resources 

![Pipeline](/screenshots/Screenshot1.png)
Infrastructure Pipeline

---

### 2. Application Pipelines

Dedicated pipeline per environment for content deployment.

**Workflow:**

GitHub(Source) → CodeBuild(Build) → S3 Sync → CloudFront Invalidation

**Responsibilities:**

**GitHub**

- Repository (Source)

**CodeBuild(Build)**

Mutiple Codebuild services are used for the different environments with their individual builspec.yml i.e `dev-update-buidspec.yml`, `staging-update-buildspec.yml`, and
`prod-update-buildspec.yml` respectively , their capabilities are :

- Retrieves the CloudFront distribution ID from CloudFormation stack output
- Uploads/Sync static files from `app/` to S3  
- Invalidate CloudFront cache using the retrieved distribution ID
- Deploy content updates without infrastructure changes  

![Pipeline](/screenshots/Screenshot2.png)
Application Pipeline (prod environment)

---

##  Deployment Lifecycle

### Initial Provisioning

Infrastructure Pipeline  
→ CloudFormation  
→ Creates:
- S3 buckets  
- CloudFront distributions  
- Route 53 records  
- Application pipelines  

![Deployment](/screenshots/Screenshot17.png)
Cloudformation Deployment (prod environment)

---

### Continuous Delivery

Code Push  
→ Application Pipeline Trigger  
→ Build (S3 Sync)  
→ CloudFront Cache Invalidation  
→ Updated Content Delivered Globally  

![Deployment](/screenshots/Screenshot10.png)
Application pipeline trigger (prod environment)

![Deployment](/screenshots/Screenshot16.png)
Codebuild Logs showing s3 sync and cloudfront invalidation update (prod environment)

![Deployment](/screenshots/Screenshot3.png)
S3 bucket before pipeline trigger (prod environment)

![Deployment](/screenshots/Screenshot11.png)
S3 bucket after pipeline trigger (prod environment)

---

##  Deployment Strategy

- **Type:** Rolling Content Update (CDN-based)  
- **Mechanism:** S3 sync + CloudFront invalidation  
- **Downtime:** Zero downtime  

---

##  Repository Structure


.
├── app/
│ └── index.html
├── ci/
├── cloudformation/
│ ├── child-templates/
│ ├── infrastructure-pipeline/
│ ├── master/
│ └── params/


---

##  Automation

- Infrastructure defined using CloudFormation (nested stacks)  
- CI/CD pipelines defined and deployed via IaC  
- Build and deployment logic managed via CodeBuild 
- Environment-specific parameterization  

---

##  Monitoring & Observability

- **CloudFront Metrics**
  - Request count, latency  

- **S3 Metrics**
  - Object access tracking  

- **Pipeline Monitoring**
  - CodePipeline execution status  



---

##  Security Implementation

- HTTPS enforced via ACM  
- Access controlled through CloudFront  
- IAM roles scoped with least privilege  
- No direct public exposure of S3 (recommended via OAI/OAC)  

---

##  DevOps Capabilities Demonstrated

- CI/CD pipeline design and automation  
- Infrastructure as Code (CloudFormation)  
- Multi-environment deployment strategy  
- Serverless architecture implementation  
- Content delivery optimization (CDN)  
- Zero-downtime deployment strategy  

---

##  Challenges & Resolutions

- **Cache inconsistency after deployment**  
  Resolved using CloudFront invalidation  

- **Environment isolation**  
  Implemented separate pipelines and resources per environment  

- **Infrastructure and application coupling**  
  Resolved using two-tier pipeline architecture  

---

##  Future Improvements

- Implement S3 Origin Access Control (OAC) for stricter security  
- Add WAF for application protection  
- Introduce automated testing before deployment    

---

##  Validation

- Verify website is accessible via domain
- Confirm CloudFront distribution is serving content
- Validate content updates reflect after deployment

![Validation](/screenshots/Screenshot4.png)
Distributions showing domain name, alternate domain, and origin

![Validation](/screenshots/Screenshot14.png)
Cloudfront Distribution Serving Content (prod environment)

![Validation](/screenshots/Screenshot13.png)
Domain name Serving content (prod environment)

---

##  Conclusion

This project demonstrates the design and implementation of a scalable, serverless static website hosting solution with automated CI/CD pipelines. It reflects strong DevOps engineering practices focused on automation, environment management, and efficient content delivery.
