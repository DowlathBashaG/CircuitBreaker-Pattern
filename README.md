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



Five Rate Limiting Algorithms.

	1. Leaky Bucket
	
	2. Token Bucket
	
	3. Fixed Window Counter
	
	4. Sliding Window Log
	
	5. Sliding Window Counter


API Rate Limiting :  < Limit the calls per API >

	1. Rate limiting is a strategy to limit access to APIs.
	
	2. It restricts the numbr of API calls that a client make within a certain time frame.
	
	3. This helps defend the API against overuse,both unintentional and malicious.
	
	4. HTTP 429 too many requests.


Circuit Breaker :
---------------

Step 0:

     Add two dependencies in POM.xml
	 
	 1. starter-AOP
	 2. starter-actuator

Step 1:

	Project : serviceA 

	web
	Resilience4j


	project : serviceB

	web
	
Step 2:

	serviceA
	
		controller
		     
			 ServiceAController

	application.yml :
	
		server:
		  port: 8080

        management:
		    health:
			  circuitbreakers:
			      enabled: true
		    endpoints:
			   web:
			     exposure:
				    include: health
		    endpoint:
			  health:
			    show-details:always
		
		
		resilience4j:
		    circuitbreaker
			  instances:
			    serviceA:
				  registerHealthIndicator: true
				  eventConsumerBufferSize: 10
				  failureRateThreshold: 50
				  minimumNumberOfCalls: 5
				  automaticTransitionFromOpenToHalfOpenEnabled: tue
				  waitDurationInOpenState: 5s
				  permittedNumberOfCallsInHalfOpenState: 3
				  slidingWindowSize: 10
				  slidingWindowType: COUNT_BASED
				  
		

Step 3: 

    @RestController
	@RequestMapping("/a")
	public class ServiceAController{
	
	   @Autowired
	   private RestTemplate restTemplate;
	   
	   private statinc final String BASE_URL = "http://localhost:8081/";
	   
	   private static final String SERVICE_A = "serviceA";
	
	   @GetMapping
	   @CircuitBreaker(name = SERVICE_A, fallbackMethod = "serviceAfallback")
	   public String serviceA(){
	       String url = BASE_URL +"b";
	       return restTemplate.getForObject(url,String.class);
	   }
	   
	   public String serviceAfallback(Exception e){{
	     return "This is a fallback method for Service A";
		}
	   
	
    }
	
	
Step 4:

    create the bean in the main class.
	
	@SpringBootApplication
	public class ServiceAApplication{
	   public static void main(String args[]){
	       SpringApplication.run(.....
		   }
		   
		   @Bean         <--- this is very very important.
		   public RestTemplate restTemplate(){
		     return new RestTemplate();
		}
	}
	
Step 5:

	serviceB
	
		controller
		     
			 ServiceBController

	application.yml :
	
	server:
	  port: 8081


Step 6: 

    @RestController
	@RequestMapping("/b")
	public class ServiceBController{
	
	
	public String serviceB(){
	    return "Service B is called from Service A";
	  }
	
	}
	

Browser :

	localhost:8080/actuator/health

    localhost:8080/a
	
		This is a fallback method for servieA
	
	Now ServiceB is down, 
	



Retry Mechanisim :
----------------

Step 0:

     Add two dependencies in POM.xml
	 
	 1. starter-AOP
	 2. starter-actuator

Step 1:

	Project : serviceA 

	web
	Resilience4j


	project : serviceB

	web
	
Step 2:

	serviceA
	
		controller
		     
			 ServiceAController

	application.yml :
	
		server:
		  port: 8080

        management:
		    health:
			  circuitbreakers:
			      enabled: true
		    endpoints:
			   web:
			     exposure:
				    include: health
		    endpoint:
			  health:
			    show-details:always
		
		
		resilience4j:
		   retry:
		     instances:
			   serviceA:
			     registerHealthIndicator: true
				 maxRetryAttempts: 5
				 waitDuration: 10s
				  
		

Step 3: 

    @RestController
	@RequestMapping("/a")
	public class ServiceAController{
	
	   @Autowired
	   private RestTemplate restTemplate;
	   
	   private statinc final String BASE_URL = "http://localhost:8081/";
	   
	   private static final String SERVICE_A = "serviceA";
	    
	   int count = 1;
	   
	   @GetMapping
	   @Retry(name = SERVICE_A, fallbackMethod = "serviceAfallback")
	   public String serviceA(){	   
	       String url = BASE_URL +"b";
		   System.out.println("Retry method called " + count++ + "times at "+ new Date());
	       return restTemplate.getForObject(url,String.class);
	   }
	   
	   public String serviceAfallback(Exception e){{
	     return "This is a fallback method for Service A";
		}
	   
	
    }
	
	
Step 4:

    create the bean in the main class.
	
	@SpringBootApplication
	public class ServiceAApplication{
	   public static void main(String args[]){
	       SpringApplication.run(.....
		   }
		   
		   @Bean         <--- this is very very important.
		   public RestTemplate restTemplate(){
		     return new RestTemplate();
		}
	}
	
Step 5:

	serviceB
	
		controller
		     
			 ServiceBController

	application.yml :
	
	server:
	  port: 8081


Step 6: 

    @RestController
	@RequestMapping("/b")
	public class ServiceBController{
	
	
	public String serviceB(){
	    return "Service B is called from Service A";
	  }
	
	}
	
Note :

ServiceA & B.

Now stop Service B, start service A only,Now check logs count....then now start Service B.

Browser :

	localhost:8080/actuator/health

    localhost:8080/a
	
		This is a fallback method for servieA
	
	Now ServiceB is down, 
	


Rate Limiter :
-------------

Step 0:

     Add two dependencies in POM.xml
	 
	 1. starter-AOP
	 2. starter-actuator

Step 1:

	Project : serviceA 

	web
	Resilience4j


	project : serviceB

	web
	
Step 2:

	serviceA
	
		controller
		     
			 ServiceAController

	application.yml :
	
		server:
		  port: 8080

        management:
		    health:
			  circuitbreakers:
			      enabled: true
		    endpoints:
			   web:
			     exposure:
				    include: health
		    endpoint:
			  health:
			    show-details:always
		
		ratelimiter:
		    instances: 
			   serviceA:
			     registerHealthIndicator: false
				 limitForPeriod: 10
				 limitRefreshPeriod: 10s
				 timeoutDuration: 3s
		
				  
		

Step 3: 

    @RestController
	@RequestMapping("/a")
	public class ServiceAController{
	
	   @Autowired
	   private RestTemplate restTemplate;
	   
	   private statinc final String BASE_URL = "http://localhost:8081/";
	   
	   private static final String SERVICE_A = "serviceA";	    
	   
	   
	   @GetMapping
	   @RateLimiter(name = SERVICE_A, fallbackMethod = "serviceAfallback")
	   public String serviceA(){	   
	       String url = BASE_URL +"b";
			       return restTemplate.getForObject(url,String.class);
	   }
	   
	   public String serviceAfallback(Exception e){{
	     return "This is a fallback method for Service A";
		}
	   
	
    }
	
	
Step 4:

    create the bean in the main class.
	
	@SpringBootApplication
	public class ServiceAApplication{
	   public static void main(String args[]){
	       SpringApplication.run(.....
		   }
		   
		   @Bean         <--- this is very very important.
		   public RestTemplate restTemplate(){
		     return new RestTemplate();
		}
	}
	
Step 5:

	serviceB
	
		controller
		     
			 ServiceBController

	application.yml :
	
	server:
	  port: 8081


Step 6: 

    @RestController
	@RequestMapping("/b")
	public class ServiceBController{
	
	
	public String serviceB(){
	    return "Service B is called from Service A";
	  }
	
	}
	
Note :

	Open network developer tool. check time nano seconds for taking request.

ServiceA & B.

	Now stop Service B, start service A only,Now check logs count....then now start Service B.

Browser :

	localhost:8080/actuator/health

    localhost:8080/a
	
		This is a fallback method for servieA
	
	Now ServiceB is down, 
	
