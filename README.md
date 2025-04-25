# CircuitBreaker-Pattern

![circuit](https://github.com/user-attachments/assets/71a664c7-d4ae-412e-b704-bf6208450429)


Steps to involve for implementing Circuit Breaker Pattern

# 1. pom.xml add the dependencies
              <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-aop</artifactId>
		</dependency>

	        <dependency>
			<groupId>io.github.resilience4j</groupId>
			<artifactId>resilience4j-spring-boot3</artifactId>
			<version>2.0.2</version>
		</dependency>

# 2. Add annotations in activity controller method along with fallBack method.

    @GetMapping
    @CircuitBreaker(name = "randomActivity", fallbackMethod = "fallbackRandomActivity")
    public String getRandomActivity() {
        ResponseEntity<Activity> responseEntity = restTemplate.getForEntity(BORED_API, Activity.class);
        Activity activity = responseEntity.getBody();
        log.info("Activity received: " + activity.getActivity());
        return activity.getActivity();
    }

# 3. Update application.yml for configurations.

    spring:
      application.name: resilience4j-demo
      jackson.serialization.indent_output: true
    
    management:
      endpoints.web.exposure.include:
        - '*'
      endpoint.health.show-details: always
      health.circuitbreakers.enabled: true
    
    resilience4j.circuitbreaker:
      configs:
        default:
          registerHealthIndicator: true
          slidingWindowSize: 10
          minimumNumberOfCalls: 5
          permittedNumberOfCallsInHalfOpenState: 3
          automaticTransitionFromOpenToHalfOpenEnabled: true
          waitDurationInOpenState: 5s
          failureRateThreshold: 50
          eventConsumerBufferSize: 10
