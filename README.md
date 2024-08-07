## DEVSECOPS

In the software deployment process, security plays a crucial role in helping the system avoid common security vulnerabilities that traditional systems often encounter. These security vulnerabilities affect the quality of software products, create bad user experiences, and increase the time to market. In this repository, I will introduce and guide the use of several tools to detect security vulnerabilities from both the development and operation perspectives. I will use Docker to build these security scanning tools, aiming for easy usage without directly installing them on the server. This makes it easy to integrate these tools into the pipeline for quick usage.

### Snyk

Snyk is a developer security platform that helps detect and fix security vulnerabilities in applications. Here are some highlights about Snyk:

#### Vulnerability Detection:
- Snyk deeply scans the source code, identifies vulnerabilities, and provides results.
- It offers detailed information about vulnerabilities, including severity levels and remediation steps.

#### CI/CD Integration:
- Snyk can easily integrate with CI/CD tools like Jenkins, GitLab CI, CircleCI, and other services to automatically scan code during build and deploy.
- This helps detect and fix vulnerabilities early in the development process.

#### Dependency Management:
- Snyk monitors open-source libraries and dependencies used in your project.
- It alerts when there are vulnerabilities in these libraries and suggests safer versions.

#### Security Policies:
- Snyk allows setting custom security policies to control how vulnerabilities are handled.
- Development teams can define priorities and remediation rules that align with their organization.

#### Community and Vulnerability Database:
- Snyk maintains a rich and up-to-date security vulnerability database.
- The platform also provides extensive documentation and guides to help developers improve their security knowledge.

Snyk is used by many companies and organizations to ensure their applications are safe from potential security threats. It helps integrate security into the software development process easily and effectively.

To integrate this tool into the DevSecOps pipeline, you need to define the values of the variables `YOUR_AUTH_TOKEN`, `SNYK_OUTPUTFILE`, `DOCKER_TAGS` and run the following commands in your pipeline:
```sh
docker build --rm --network host --build-arg SNYK_AUTH_TOKEN=$YOUR_AUTH_TOKEN --build-arg OUTPUT_FILENAME=$SNYK_OUTPUTFILE -t snyk_images_${DOCKER_TAGS} -f Dockerfile_snyk .
docker create --name snyk_container snyk_images_${DOCKER_TAGS}
docker cp snyk_container:/app/$SNYK_FILE.html .
```

After the command is completed, you will receive an HTML formatted result file. You can save this result for fixing and remediating security vulnerabilities in your source code.

### Trivy

Trivy is a popular open-source tool for scanning security vulnerabilities in applications, containers, and infrastructure. Developed by Aqua Security, Trivy helps developers and security administrators ensure the safety of their application environment. Here are some highlights about Trivy:

#### Vulnerability Detection:
- Trivy scans security vulnerabilities in container images, source code repositories, and Infrastructure as Code (IaC) documents like Terraform and Kubernetes.
- It provides detailed information about vulnerabilities, including severity levels and remediation steps.

#### Support for Multiple Targets:
- Trivy can scan multiple targets, including container images, source code repositories, and file systems.
- It supports popular operating systems like Alpine, RHEL, CentOS, Ubuntu, Debian, and open-source libraries like Node.js, Ruby, Python, Java, and more.

#### Ease of Use:
- Trivy has a simple and easy-to-use command-line interface (CLI). Users can easily install and start scanning vulnerabilities with a few basic commands.
- It can also integrate with CI/CD tools like Jenkins, GitLab CI, CircleCI, and other services to automatically scan code during build and deploy.

#### Vulnerability Database:
- Trivy uses a vulnerability database from reliable sources like NVD (National Vulnerability Database), Red Hat Security Data, and many others.
- Trivy's vulnerability database is regularly updated to ensure users always have the latest information about security vulnerabilities.

#### Dependency Management:
- Trivy monitors and scans project dependencies to detect vulnerabilities in open-source libraries and packages.
- It provides detailed reports on vulnerabilities in these dependencies and suggests patches or safer versions.

#### Container and IaC Security:
- Trivy scans Docker images and IaC documents like Kubernetes YAML files and Terraform to detect and fix security issues.
- It ensures that your infrastructure configuration and deployment comply with security standards.

Trivy is a powerful and flexible tool that helps developers and security administrators ensure the safety of their application environment. With its ease of use and broad support, Trivy has become a popular choice in the open-source community.

In this guide, I focus on Trivy's Docker image scanning feature. After scanning the source code and packaging it in a Docker image, to ensure the Docker image is safe, you should use Trivy to scan and early detect potential threats to your infrastructure. The Trivy provider has created a Docker image that is publicly available on Dockerhub. You can pull it and integrate it into your pipeline easily with the command below. You need to define the values `CI_PROJECT_NAME`, `YOUR_DOCKER_IMAGE`, `TAG`. Trivy will return the scan result of the container `${YOUR_DOCKER_IMAGE}:${TAG}`.

```sh
docker run --rm -v $(pwd):/${CI_PROJECT_NAME} -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --format template --template "@contrib/html.tpl" --output /${CI_PROJECT_NAME}/report_${DOCKER_TAGS}.html ${YOUR_DOCKER_IMAGE}:${TAG}
```
### Arachni

Arachni is a powerful and comprehensive open-source web application security scanner designed to help detect and remediate security vulnerabilities. Here are some highlights about Arachni:

#### Vulnerability Detection:
- Arachni can detect a wide range of common web security vulnerabilities such as SQL Injection, XSS (Cross-Site Scripting), Local File Inclusion, Remote File Inclusion, and many other vulnerabilities.
- It provides detailed information about vulnerabilities including descriptions, severity levels, and remediation steps.

#### Automation and High Performance:
- Arachni is designed to run automatically and handle multiple requests concurrently, speeding up the scanning process.
- It uses a flexible plugin system to extend functionality and customize scanning processes.

#### Extensibility:
- Arachni supports extensibility through modules and plugins, allowing users to add new features or improve existing ones.
- The Arachni open-source community regularly updates and develops new modules to address new security vulnerabilities.

#### User Interface and API:
- Arachni provides a powerful command-line interface (CLI) that is easy to use for security professionals and developers.
- Additionally, Arachni has a web interface and a RESTful API, making it easy to integrate into security processes and CI/CD.

#### Dynamic and Static Analysis:
- Arachni performs both dynamic (runtime) and static (code) analysis to detect security vulnerabilities.
- It can scan and test web applications in real-time and analyze source code to find potential security vulnerabilities.

#### Detailed Reporting:
- Arachni generates detailed reports with information about vulnerabilities, including descriptions, evidence, severity levels, and remediation guidance.
- Reports can be exported in various formats such as HTML, JSON, XML, and more.

#### Support for Multiple Languages and Frameworks:
- Arachni can scan web applications developed in various programming languages and frameworks such as Ruby, Python, PHP, Java, and many others.

Arachni is a comprehensive and flexible web application security scanner suitable for both security professionals and developers. With its powerful vulnerability detection capabilities and extensible features, Arachni ensures that web applications are always safe from security threats.

To use Arachni, you need to build `Dockerfile-arachni`:

```sh
docker build -t arachni:v1.4-0.5.10 -f Dockerfile-arachni .
```

After successfully building on the server, you can push it to a Docker registry for reuse when you want to use Arachni on other servers. To integrate Arachni into the pipeline, you need to set the following commands in your pipeline:

```sh
docker run --rm -v /tmp/:/tmp/ arachni:v1.4-0.5.10 bin/arachni --output-verbose --scope-include-subdomains $YOUR_URL --report-save-path=/tmp/scan.afr
docker run --rm -v /tmp/:/tmp arachni:v1.4-0.5.10 bin/arachni_reporter /tmp/scan.afr --reporter=html:outfile=/tmp/scan.html.zip
```
