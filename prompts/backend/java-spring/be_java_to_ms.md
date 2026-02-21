---
name: Java to Microservice
dcc_uri: dev/prompts/be_java_to_ms
version: '1.2'
schema: v1
description: >-
  Convert a Java class into a production-ready Spring Boot microservice with
  Docker and Kubernetes Helm charts
dcc_definition_type: prompt
dcc_tags:
  - dev
  - spring
  - backend
  - microservices
invokable: true
prompt: >-
  You are a senior Java architect with deep expertise in Spring Boot 3.x,
  Docker, and Kubernetes. You will receive one or more plain Java classes and
  convert them into a complete, production-ready Spring Boot microservice
  packaged as a Docker container and deployable to Kubernetes via Helm charts.

  {{{ input }}}

  ---

  ## Conversion Rules

  ### Spring Boot Service

  - Use Spring Boot 3.x with Java 21 - Use constructor injection exclusively —
  no `@Autowired` on fields - Expose the class logic as a REST API using
  `@RestController` - Keep the controller thin: validate input, delegate to a
  `@Service`, return a response - Use `@ConfigurationProperties` for all
  configuration — no scattered `@Value` annotations - Validate all request
  bodies with Bean Validation (`@NotNull`, `@NotBlank`, etc.) - Return
  consistent error responses using `ProblemDetail` (RFC 7807) from a
  `@RestControllerAdvice` - Use `async/await` patterns via `CompletableFuture`
  only if the original class has async intent; otherwise keep it synchronous -
  Expose `/actuator/health`, `/actuator/info`, and `/actuator/prometheus`
  endpoints - All configuration values (ports, timeouts, external URLs) must
  come from `application.yml` with safe defaults - Structure packages by
  feature, not by layer

  ### Project Structure

  Generate the following files:

  ``` {service-name}/ ├── src/main/java/com/{org}/{service}/ │   ├──
  {ServiceName}Application.java │   ├── controller/ │   │   └──
  {ServiceName}Controller.java │   ├── service/ │   │   └──
  {ServiceName}Service.java │   ├── model/ │   │   ├──
  {ServiceName}Request.java   (record) │   │   └── {ServiceName}Response.java 
  (record) │   ├── config/ │   │   └── {ServiceName}Properties.java │   └──
  exception/ │       └── GlobalExceptionHandler.java ├── src/main/resources/ │  
  └── application.yml ├── src/test/java/com/{org}/{service}/ │   └──
  {ServiceName}ControllerTest.java  (@WebMvcTest) ├── Dockerfile ├──
  build.gradle (or pom.xml — match the original project if detectable) └── helm/
      └── {service-name}/
          ├── Chart.yaml
          ├── values.yaml
          └── templates/
              ├── deployment.yaml
              ├── service.yaml
              ├── configmap.yaml
              ├── hpa.yaml
              └── ingress.yaml
  ```

  ### Dockerfile

  - Use a multi-stage build - Stage 1: build with
  `eclipse-temurin:21-jdk-alpine` - Stage 2: run with
  `eclipse-temurin:21-jre-alpine` - Run as a non-root user - Copy only the built
  jar — no source code in the final image - Expose the correct port - Set
  `JAVA_OPTS` as an environment variable for JVM tuning

  ```dockerfile # Example structure — adapt to the actual service FROM
  eclipse-temurin:21-jdk-alpine AS builder WORKDIR /app COPY . . RUN ./gradlew
  bootJar --no-daemon FROM eclipse-temurin:21-jre-alpine RUN addgroup -S app &&
  adduser -S app -G app WORKDIR /app COPY --from=builder /app/build/libs/*.jar
  app.jar USER app ENV JAVA_OPTS="" EXPOSE 8080 ENTRYPOINT ["sh", "-c", "java
  $JAVA_OPTS -jar app.jar"] ```

  ### Helm Charts

  Generate a complete Helm chart with the following:

  **Chart.yaml** - `apiVersion: v2` - `type: application` - Correct `name`,
  `description`, and `version: 0.1.0`

  **values.yaml** — must include: - `replicaCount: 1` - `image.repository`,
  `image.tag: latest`, `image.pullPolicy: IfNotPresent` - `service.type:
  ClusterIP`, `service.port: 80`, `service.targetPort: 8080` -
  `resources.requests` and `resources.limits` for CPU and memory -
  `autoscaling.enabled: false`, `autoscaling.minReplicas`,
  `autoscaling.maxReplicas`, `autoscaling.targetCPUUtilizationPercentage` -
  `ingress.enabled: false`, `ingress.className`, `ingress.host` -
  `livenessProbe` and `readinessProbe` pointing to `/actuator/health/liveness`
  and `/actuator/health/readiness` - `configmap` section for all externalised
  application properties

  **deployment.yaml** — must include: - `RollingUpdate` strategy - Environment
  variables sourced from the ConfigMap - Liveness and readiness probes -
  Resource requests and limits - `terminationGracePeriodSeconds: 60` - Security
  context: `runAsNonRoot: true`, `readOnlyRootFilesystem: true`,
  `allowPrivilegeEscalation: false`

  **hpa.yaml** — HorizontalPodAutoscaler gated by `autoscaling.enabled`

  **service.yaml** — ClusterIP service wiring container port to service port

  **ingress.yaml** — gated by `ingress.enabled`, with TLS block ready to be
  filled in

  ### Output Order

  Produce files in this order, each clearly labelled with its path as a comment
  or heading:

  1. `Application.java` 2. `Controller.java` 3. `Service.java` 4. Request and
  Response records 5. `ConfigurationProperties` class 6.
  `GlobalExceptionHandler.java` 7. `application.yml` 8. `build.gradle` or
  `pom.xml` 9. `ControllerTest.java` 10. `Dockerfile` 11. `helm/Chart.yaml` 12.
  `helm/values.yaml` 13. `helm/templates/deployment.yaml` 14.
  `helm/templates/service.yaml` 15. `helm/templates/configmap.yaml` 16.
  `helm/templates/hpa.yaml` 17. `helm/templates/ingress.yaml`

  At the end include a **Deployment Notes** section covering: - How to build the
  Docker image - How to install/upgrade the Helm chart - Any assumptions made
  about the original class - Any TODO items where the original class logic was
  ambiguous
---

