# Spring Boot Hello World

This project demonstrates a simple Spring Boot application that prints "Hello from CI/CD Pipeline!".

## Build and Run Locally
\`\`\`bash
mvn clean package
java -jar target/springboot-hello-world-0.0.1-SNAPSHOT.jar
\`\`\`

## Docker
\`\`\`bash
docker build -t springboot-hello-world .
docker run -p 8080:8080 springboot-hello-world
\`\`\`

## CI/CD Tech Stack
- Jenkins
- Maven
- SonarQube
- Docker
- Trivy
- JFrog
- ArgoCD
- Kubernetes
