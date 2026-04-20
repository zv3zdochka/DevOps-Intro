## Task 1 — Artifact Registries Research

### 1.1 Overview

Artifact registries are used to store, manage, secure, and distribute software artifacts such as container images, OCI artifacts, and language-specific packages. A key difference between cloud providers is that Google Cloud offers a more unified registry model, while AWS and Azure split this functionality across separate services depending on artifact type. ([Google Cloud Documentation][2])

### 1.2 Main services by cloud provider

#### AWS

For AWS, there is no single universal artifact registry for all formats. In practice, AWS uses two main services:

* **Amazon Elastic Container Registry (Amazon ECR)** for container images, OCI images, and OCI-compatible artifacts. It supports private repositories, IAM-based access control, vulnerability scanning, pull-through cache, and cross-region / cross-account replication. It is tightly integrated with ECS, EKS, EC2, IAM, Amazon Inspector, and CI/CD workflows in AWS. Pricing is based mainly on stored data and data transfer. Common use cases include storing container images for Kubernetes or ECS deployments and distributing OCI artifacts in production pipelines. ([AWS Documentation][1])

* **AWS CodeArtifact** for software packages. It supports npm, PyPI, Maven, NuGet, Swift, Ruby, Cargo, and generic package formats. Its main strengths are centralized package management, upstream access to public repositories, access control, and integration with AWS developer tooling such as CodeBuild, EventBridge, and CloudTrail. Pricing is pay-as-you-go based on storage, requests, and outbound transfer. Typical use cases are private package hosting, dependency control, and internal package sharing across teams. ([AWS Documentation][3])

#### GCP

* **Google Artifact Registry** is Google Cloud’s main and most unified artifact registry service. It supports Docker images, Helm charts in OCI format, Maven, npm, Python, Go, and also remote/virtual repository patterns; some OS package formats such as APT and YUM are available in preview. Key features include IAM-based access control, vulnerability scanning through Artifact Analysis, remote repositories for caching upstream sources, and virtual repositories for unified access. It integrates well with Cloud Build, GKE, Cloud Run, IAM, and other Google Cloud services. Pricing is mainly based on storage and data transfer, while automatic vulnerability scanning is billed separately. Common use cases include container delivery, private package management, and acting as a centralized artifact hub inside a cloud-native CI/CD pipeline. ([Google Cloud Documentation][2])

#### Azure

Azure also effectively uses two services depending on artifact type:

* **Azure Container Registry (ACR)** for Docker images, OCI images, Helm charts stored as OCI artifacts, and related OCI content. Key features include integrated authentication, geo-replication, Private Link / network isolation options, and broad Azure integration with AKS, App Service, Azure Machine Learning, Azure Batch, and Azure Red Hat OpenShift. Pricing is tier-based (Basic, Standard, Premium), with Premium enabling advanced features such as geo-replication. Common use cases include storing production container images and distributing OCI artifacts close to deployment regions. ([Microsoft Learn][4])

* **Azure Artifacts** for software packages. It supports feeds that can host multiple package types, including npm, NuGet, Maven, Python, Cargo, and Universal Packages, and it can cache public packages through upstream sources. It is strongly integrated with Azure DevOps and Azure Pipelines. Pricing includes a free storage allowance and then charges per GiB. Common use cases are private dependency management, secure package sharing, and reducing exposure to public registry outages. ([Microsoft Azure][5])

### 1.3 Comparison table

| Cloud | Main service(s)                            | Supported artifacts                                                                                                       | Key features                                                                                              | Integrations                                                           | Pricing basics                                                           | Best fit                                                                                  |                                   |
| ----- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- | --------------------------------- |
| AWS   | Amazon ECR + AWS CodeArtifact              | ECR: Docker/OCI images and OCI artifacts; CodeArtifact: npm, PyPI, Maven, NuGet, Swift, Ruby, Cargo, generic packages     | IAM access control, vulnerability scanning, pull-through cache, replication, upstream package consumption | ECS, EKS, EC2, IAM, Inspector, CodeBuild, EventBridge, CloudTrail      | ECR: storage + transfer; CodeArtifact: storage + requests + transfer out | Teams already deep in AWS, especially when containers and packages are managed separately | ([AWS Documentation][1])          |
| GCP   | Artifact Registry                          | Docker, OCI-based Helm, Maven, npm, Python, Go; remote and virtual repos; some OS package support in preview              | Unified registry model, IAM, vulnerability scanning, remote repos, virtual repos                          | Cloud Build, GKE, Cloud Run, IAM                                       | Storage + transfer; scanning billed separately                           | Organizations that want one registry layer for both containers and packages               | ([Google Cloud Documentation][2]) |
| Azure | Azure Container Registry + Azure Artifacts | ACR: Docker/OCI images, Helm as OCI, OCI artifacts; Azure Artifacts: npm, NuGet, Maven, Python, Cargo, Universal Packages | Geo-replication, network controls, OCI support, upstream sources, Azure DevOps feeds                      | AKS, App Service, Azure ML, Azure Batch, Azure DevOps, Azure Pipelines | ACR: tiered SKU pricing; Azure Artifacts: 2 GiB free then per GiB        | Azure-native environments, especially those using AKS or Azure DevOps                     | ([Microsoft Azure][6])            |

### 1.4 Analysis: Which registry would I choose for a multi-cloud strategy?

For a **multi-cloud strategy**, I would choose **Google Artifact Registry** as the strongest overall option.

My reason is that it is the most **unified** service among the three providers. With GCP, one managed registry can cover both container artifacts and several common package ecosystems, while also supporting remote and virtual repository patterns. In AWS and Azure, the artifact story is more fragmented because container registries and package registries are split into different services. That does not make AWS or Azure bad, but it makes the architecture less clean when you want one central artifact management layer across multiple clouds. ([Google Cloud Documentation][2])

Another reason is operational simplicity. Artifact Registry has strong IAM integration and native support for vulnerability-scanning workflows, and it is designed as a central place for build dependencies and deployable artifacts. For a DevOps team that wants fewer moving parts and more consistent policy management, that is a practical advantage. ([Google Cloud Documentation][7])

At the same time, for a **container-only** multi-cloud strategy, Azure Container Registry and Amazon ECR are also strong choices because they both support OCI artifacts and production-grade security and replication features. But for the broader “artifact registry” category, GCP has the clearest all-in-one model. ([AWS Documentation][1])


[1]: https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html "What is Amazon Elastic Container Registry? - Amazon ECR"
[2]: https://docs.cloud.google.com/artifact-registry/docs/overview "Artifact Registry overview"
[3]: https://docs.aws.amazon.com/codeartifact/ "AWS CodeArtifact Documentation"
[4]: https://learn.microsoft.com/en-us/azure/container-registry/container-registry-concepts "About Registries, Repositories, Images, and Artifacts"
[5]: https://azure.microsoft.com/en-us/products/devops/artifacts "Azure Artifacts"
[6]: https://azure.microsoft.com/en-us/products/container-registry "Azure Container Registry"
[7]: https://docs.cloud.google.com/artifact-registry/docs "Artifact Registry documentation"

## Task 2 — Serverless Computing Platform Research

### 2.1 Main serverless platforms by cloud provider

#### AWS

The primary serverless compute platform in AWS is **AWS Lambda**. Lambda is AWS’s event-driven serverless compute service and is designed to run code without provisioning or managing servers. It supports managed runtimes for major languages such as Node.js, Python, Java, and .NET, and it also supports other languages like Go and Rust through OS-only/custom runtimes. AWS Lambda can be invoked through direct push-style integrations or pull-based event source mappings, including HTTP requests through API Gateway, schedules through EventBridge, and storage or stream events from services such as S3, SQS, and Kinesis. ([Amazon Web Services, Inc.][1])

From a performance perspective, Lambda has cold starts when AWS needs to create a new execution environment. AWS documents that cold starts typically happen in under 1% of invocations, usually ranging from under 100 ms to over 1 second, and it offers mitigation features such as SnapStart and provisioned concurrency. Pricing is based mainly on the number of requests and execution duration, and the standard timeout limit for a function is up to 900 seconds, or 15 minutes. Common use cases include event-driven automation, APIs, file processing, stream processing, and backend microservices. ([AWS Documentation][2])

#### GCP

On Google Cloud, the main serverless platforms are **Cloud Run** and **Cloud Run functions**. Cloud Run is the broader serverless application platform and is especially strong for HTTP services, containerized APIs, batch jobs, and background workers. It can run code written in any language if it can be packaged as a container, and for common stacks such as Go, Node.js, Python, Java, .NET, and Ruby, Google also supports source-based deployments. Cloud Run functions are the function-oriented layer on top of Cloud Run and support runtimes including Node.js, Python, Go, Java, Ruby, PHP, and .NET. ([Google Cloud Documentation][3])

Cloud Run supports several execution models: HTTP services, run-to-completion jobs, worker pools, and event-driven functions. Cloud Run functions can be triggered by HTTP requests or by Eventarc-based event sources. For pricing, Cloud Run charges only for the resources used and rounds usage to the nearest 100 milliseconds; Cloud Run functions inherit the Cloud Run pricing model. In terms of limits, Cloud Run services have a default 5-minute request timeout that can be extended to 60 minutes, while Cloud Run functions support up to 60 minutes for HTTP-triggered functions and up to 9 minutes for event-driven functions. Cold-start latency is mainly associated with scale-from-zero and container startup, and Google recommends using startup CPU boost or minimum instances to reduce startup delays. Common use cases include REST APIs, microservices, web backends, event-driven handlers, and batch processing. ([Google Cloud][4])

#### Azure

In Azure, the main serverless compute platforms are **Azure Functions** and **Azure Container Apps**. Azure Functions is the primary event-driven FaaS platform. It supports C#, Java, JavaScript, PowerShell, and Python, and it also allows custom handlers for languages such as Rust and Go. Azure Functions uses triggers and bindings, which makes it well suited for HTTP APIs, timers, storage queues, Event Grid events, and integrations with other Azure services. Azure Container Apps is Azure’s serverless container platform for APIs, microservices, background jobs, and event-driven processing. ([Microsoft Learn][5])

For pricing, Azure Functions depends on the hosting plan. Flex Consumption and Consumption plans are billed by executions and GB-seconds, while Premium is billed by vCPU-seconds and memory and removes execution charges. Azure Container Apps uses pay-as-you-go pricing billed by the second and is designed to scale dynamically based on HTTP traffic or events. For execution limits, Azure Functions varies by plan: Flex Consumption and Premium are effectively unbounded for function execution, while the classic Consumption plan has a 10-minute maximum; however, HTTP-triggered Azure Functions still have a 230-second response limit due to the Azure Load Balancer timeout. For cold starts, Azure says Flex Consumption improves cold-start behavior and supports always-ready instances, Premium can avoid cold starts, and Container Apps behavior depends on the minimum replica count. Common Azure use cases include event-driven automation, serverless APIs, workflow orchestration, and container-based microservices. ([Microsoft Azure][6])

### 2.2 Comparison table

| Cloud provider | Main serverless platform(s)           | Supported runtimes / languages                                                                                                                                             | Execution model                                                                                                                     | Cold start characteristics                                                                                                                       | Pricing model                                                                                                                      | Maximum execution duration                                                                                                                                                                              | Typical use cases                                                                                                   |
| -------------- | ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| AWS            | AWS Lambda                            | Managed runtimes for Node.js, Python, Java, .NET, and others; Go/Rust via custom runtime ([AWS Documentation][7])                                                          | Event-driven, push and pull integrations, HTTP via API Gateway or function URLs ([AWS Documentation][8])                            | Cold starts typically occur in under 1% of invocations; SnapStart and provisioned concurrency reduce latency ([AWS Documentation][2])            | Requests + duration, with free tier ([Amazon Web Services, Inc.][1])                                                               | Up to 15 minutes ([AWS Documentation][9])                                                                                                                                                               | Event-driven automation, APIs, queues, streams, file processing ([AWS Documentation][8])                            |
| GCP            | Cloud Run, Cloud Run functions        | Cloud Run: any language via containers; Cloud Run functions: Node.js, Python, Go, Java, Ruby, PHP, .NET ([Google Cloud Documentation][3])                                  | HTTP services, jobs, worker pools, HTTP functions, Eventarc-driven functions ([Google Cloud Documentation][3])                      | Startup latency mainly appears when scaling from zero; minimum instances and startup CPU boost help reduce it ([Google Cloud Documentation][10]) | Resource-based pay-per-use rounded to 100 ms; functions use Cloud Run pricing ([Google Cloud][4])                                  | Services: default 5 min, up to 60 min; functions: 60 min HTTP / 9 min event-driven ([Google Cloud Documentation][11])                                                                                   | REST APIs, microservices, web apps, event handlers, batch jobs ([Google Cloud][12])                                 |
| Azure          | Azure Functions, Azure Container Apps | Functions: C#, Java, JavaScript, PowerShell, Python; custom handlers for Go/Rust-like scenarios. Container Apps: container-based, language-agnostic ([Microsoft Learn][5]) | Functions: HTTP, timer, queue, event-driven triggers/bindings. Container Apps: HTTP, TCP, custom/KEDA, jobs ([Microsoft Learn][13]) | Flex Consumption improves cold start; Premium can avoid cold starts; Container Apps depends on minimum replicas ([Microsoft Learn][14])          | Functions: executions + GB-s on Consumption/Flex, vCPU+memory on Premium; Container Apps: pay by the second ([Microsoft Azure][6]) | Functions: unbounded on Flex/Premium, 10 min on classic Consumption; HTTP responses capped at 230 s. Container Apps is container-oriented rather than function-timeout-oriented ([Microsoft Learn][14]) | Azure-native automation, serverless APIs, workflows, microservices, event-driven containers ([Microsoft Learn][15]) |

### Analysis: Which serverless platform would I choose for a REST API backend and why?

For a **REST API backend**, I would choose **Google Cloud Run**.

The main reason is that Cloud Run is the most balanced option for HTTP-first applications. It is explicitly designed for web services and APIs, supports any language through containers, also supports source-based deployments for common languages, scales from zero, and charges based on actual resource usage rather than forcing a strictly function-shaped programming model. It also supports configurable request timeouts up to 60 minutes, which is more flexible for API workloads than the hard 15-minute Lambda limit and more convenient than Azure Functions’ HTTP response ceiling of 230 seconds. ([Google Cloud Documentation][3])

AWS Lambda would be my second choice when the backend is deeply integrated with AWS events and services, because Lambda is excellent for event-driven designs and AWS-native architectures. Azure Functions is also strong, especially in Azure-heavy environments, but for a general REST API backend, Cloud Run feels more natural because it is closer to a normal service/container deployment model while still keeping serverless scaling and pricing. This makes it easier to build larger APIs and microservices without being boxed into a narrow FaaS model. ([AWS Documentation][8])

### Reflection: Main advantages and disadvantages of serverless computing

The main advantages of serverless computing are reduced operational overhead, automatic scaling, and pay-for-use billing. All three providers emphasize that you do not need to provision or manage servers directly, and their platforms can scale dynamically with incoming traffic or events. This is especially useful for bursty workloads, APIs with unpredictable traffic, and event-driven systems. ([Amazon Web Services, Inc.][1])

The main disadvantages are cold starts, execution limits, and platform-specific constraints. AWS Lambda still has cold starts, even if they are relatively rare; Cloud Run can add startup latency when scaling from zero; and Azure Functions behavior depends heavily on the hosting plan. There are also practical limits such as Lambda’s 15-minute cap, Cloud Run function timeouts for event-driven functions, and Azure Functions’ 230-second HTTP response limit. In practice, this means serverless is excellent for many modern workloads, but it is not always the best fit for long-running, latency-sensitive, or heavily stateful applications. ([AWS Documentation][2])


[1]: https://aws.amazon.com/lambda/pricing/ "AWS Lambda Pricing"
[2]: https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html "Understanding the Lambda execution environment lifecycle - AWS Lambda"
[3]: https://docs.cloud.google.com/run/docs/overview/what-is-cloud-run "What is Cloud Run  |  Google Cloud Documentation"
[4]: https://cloud.google.com/run/pricing "Cloud Run pricing | Google Cloud"
[5]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview "Azure Functions overview | Microsoft Learn"
[6]: https://azure.microsoft.com/en-us/pricing/details/functions/ "Pricing - Functions | Microsoft Azure"
[7]: https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html "Lambda runtimes - AWS Lambda"
[8]: https://docs.aws.amazon.com/lambda/latest/dg/concepts-event-driven-architectures.html "Creating event-driven architectures with Lambda - AWS Lambda"
[9]: https://docs.aws.amazon.com/lambda/latest/dg/configuration-timeout.html "Configure Lambda function timeout"
[10]: https://docs.cloud.google.com/run/docs/tips/general "General development tips  |  Cloud Run  |  Google Cloud Documentation"
[11]: https://docs.cloud.google.com/run/docs/configuring/request-timeout "Configure request timeout for services  |  Cloud Run  |  Google Cloud Documentation"
[12]: https://cloud.google.com/run "Cloud Run | Google Cloud"
[13]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings "Triggers and Bindings in Azure Functions | Microsoft Learn"
[14]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale "Azure Functions Scale and Hosting | Microsoft Learn"
[15]: https://learn.microsoft.com/en-us/azure/container-apps/overview "Azure Container Apps overview | Microsoft Learn"
