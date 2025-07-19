# PACT Consumer-Driven Contract Testing

## Introduction

PACT is a consumer-driven contract testing tool that enables testing interactions between service consumers and providers without requiring them to be deployed at the same time. This guide provides comprehensive guidance on implementing PACT testing in your projects.

## What is Consumer-Driven Contract Testing?

Consumer-driven contract testing is an approach where:

1. The consumer of an API defines the expectations (the contract) for interactions with the provider
2. These expectations are codified as tests on both the consumer and provider sides
3. The provider verifies that it can meet the expectations set by its consumers

This approach ensures that API changes don't break existing consumers and provides clear documentation of service interactions.

## Key Concepts

### Pacts

A "pact" is a contract between a consumer and provider. It consists of:

- **Consumer**: The application that initiates a request to another application
- **Provider**: The application that responds to the request
- **Interactions**: The expected request/response pairs that define the contract

### Pact Broker

The Pact Broker is a repository for pacts that:

- Enables sharing of pacts between consumers and providers
- Provides webhooks to trigger provider verification when contracts change
- Visualizes the relationships between services
- Implements version control for contracts

## Implementation Guidelines

### Consumer-Side Testing

#### Maven Dependencies

```xml
<dependency>
    <groupId>au.com.dius.pact.consumer</groupId>
    <artifactId>junit5</artifactId>
    <version>4.6.1</version>
    <scope>test</scope>
</dependency>
```

#### Writing Consumer Tests

1. **Define the expected interaction**:

```java
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "user-service")
public class UserServiceConsumerTest {

    @Pact(consumer = "order-service")
    public RequestResponsePact createPact(PactDslWithProvider builder) {
        return builder
            .given("a user with ID 1 exists")
            .uponReceiving("a request for user 1")
                .path("/users/1")
                .method("GET")
            .willRespondWith()
                .status(200)
                .body(new PactDslJsonBody()
                    .integerType("id", 1)
                    .stringType("name", "Test User")
                    .stringType("email", "test@example.com"))
            .toPact();
    }

    @Test
    @PactTestFor(pactMethod = "createPact")
    void testUserRetrieval(MockServer mockServer) {
        // Configure your client to use the mock server URL
        UserClient client = new UserClient(mockServer.getUrl());
        
        // Execute the client method that should make the expected request
        User user = client.getUserById(1);
        
        // Verify the response was processed correctly
        assertThat(user.getId()).isEqualTo(1);
        assertThat(user.getName()).isEqualTo("Test User");
        assertThat(user.getEmail()).isEqualTo("test@example.com");
    }
}
```

2. **Generate the pact file**: The test will generate a pact file in the target/pacts directory

3. **Publish the pact**: Configure the pact-jvm-provider-maven plugin to publish the pact to a Pact Broker

```xml
<plugin>
    <groupId>au.com.dius.pact.provider</groupId>
    <artifactId>maven</artifactId>
    <version>4.6.1</version>
    <configuration>
        <pactBrokerUrl>https://your-pact-broker-url</pactBrokerUrl>
        <pactBrokerUsername>username</pactBrokerUsername>
        <pactBrokerPassword>password</pactBrokerPassword>
        <projectVersion>${project.version}</projectVersion>
        <tags>
            <tag>prod</tag>
            <tag>${project.version}</tag>
        </tags>
    </configuration>
</plugin>
```

### Provider-Side Testing

#### Maven Dependencies

```xml
<dependency>
    <groupId>au.com.dius.pact.provider</groupId>
    <artifactId>junit5</artifactId>
    <version>4.6.1</version>
    <scope>test</scope>
</dependency>
```

#### Writing Provider Tests

```java
@Provider("user-service")
@PactBroker(
    host = "your-pact-broker-host",
    port = "80",
    authentication = @PactBrokerAuth(username = "username", password = "password")
)
public class UserServiceProviderTest {

    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void pactVerificationTestTemplate(PactVerificationContext context) {
        context.verifyInteraction();
    }

    @BeforeEach
    void setUp(PactVerificationContext context) {
        // Start your Spring Boot application or set up the test target
        context.setTarget(new HttpTestTarget("localhost", 8080));
    }

    @State("a user with ID 1 exists")
    void userWithId1Exists() {
        // Set up the state in your provider
        // For example, insert test data into the database
    }
}
```

## Best Practices

### General Guidelines

1. **Focus on the contract, not implementation**: Test only what matters for the interaction
2. **Keep pacts simple**: Include only the fields that the consumer actually uses
3. **Use provider states**: Clearly define the preconditions for each interaction
4. **Version your pacts**: Tag pacts with versions to manage compatibility
5. **Automate verification**: Run provider verification in CI/CD pipelines

### Consumer-Side Best Practices

1. **Test the client code**: Ensure your tests verify that your client correctly processes the response
2. **Use type matchers**: Prefer type matchers over exact values to create more flexible contracts
3. **Minimize expectations**: Only include fields and headers that your consumer actually uses
4. **Handle error cases**: Test how your consumer handles provider errors

### Provider-Side Best Practices

1. **Verify against all consumer pacts**: Ensure your provider works with all its consumers
2. **Implement provider states**: Set up the correct test data for each interaction
3. **Run verification on every change**: Verify pacts before deploying changes
4. **Use pending pacts**: Mark new pacts as pending until they're ready to be enforced

## Common Pitfalls and Solutions

### Overspecified Pacts

**Problem**: Including too many details in pacts makes them brittle.

**Solution**: Only include fields and matching rules that the consumer actually cares about.

### Missing Provider States

**Problem**: Provider verification fails because the expected state isn't set up.

**Solution**: Implement all required provider states and ensure they set up the correct test data.

### Version Conflicts

**Problem**: Breaking changes affect existing consumers.

**Solution**: Use semantic versioning and can-i-deploy checks in your deployment pipeline.

## Advanced Topics

### Pact Broker Webhooks

Configure webhooks in the Pact Broker to trigger provider verification when a consumer publishes a new pact:

```json
{
  "events": [{
    "name": "contract_content_changed"
  }],
  "request": {
    "method": "POST",
    "url": "https://your-ci-server/trigger-build",
    "headers": {
      "Content-Type": "application/json"
    },
    "body": {
      "pactUrl": "${pactbroker.pactUrl}"
    }
  }
}
```

### Contract Testing Microservices

For microservice architectures:

1. Identify all consumer-provider relationships
2. Implement consumer tests for each relationship
3. Publish all pacts to a central Pact Broker
4. Verify provider compatibility before deployment
5. Use the Pact Broker's can-i-deploy feature to ensure safe deployments

## Conclusion

PACT consumer-driven contract testing provides a powerful way to ensure that services can communicate effectively without requiring simultaneous deployment or end-to-end testing environments. By following the guidelines in this document, you can implement robust contract testing that improves the reliability of your distributed systems.

## References

- [Official Pact Documentation](https://docs.pact.io/)
- [Pact JVM Documentation](https://github.com/pact-foundation/pact-jvm)
- [Pact Broker Documentation](https://docs.pact.io/pact_broker)