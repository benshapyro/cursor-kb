# Production Best Practices

Transition AI projects to production with best practices.

This guide provides a comprehensive set of best practices to help you transition from prototype to production. Whether you are a seasoned machine learning engineer or a recent enthusiast, this guide should provide you with the tools you need to successfully put the platform to work in a production setting: from securing access to our API to designing a robust architecture that can handle high traffic volumes. Use this guide to help develop a plan for deploying your application as smoothly and effectively as possible.

## Setting up your Organization

Once you log in to your OpenAI account, you can find your organization name and ID in your organization settings. The organization name is the label for your organization, shown in user interfaces. The organization ID is the unique identifier for your organization which can be used in API requests.

Users who belong to multiple organizations can pass a header to specify which organization is used for an API request. Usage from these API requests will count against the specified organization's quota. If no header is provided, the default organization will be billed. You can change your default organization in your user settings.

You can invite new members to your organization from the Team page. Members can be readers or owners.

### Readers:
- Can make API requests
- Can view basic organization information
- Can create, update, and delete resources (like Assistants) in the organization, unless otherwise noted

### Owners:
- Have all the permissions of readers
- Can modify billing information
- Can manage members within the organization

## Managing Billing Limits

To begin using the OpenAI API, enter your billing information. If no billing information is entered, you will still have login access but will be unable to make API requests.

Once you've entered your billing information, you will have an approved usage limit of $100 per month, which is set by OpenAI. Your quota limit will automatically increase as your usage on your platform increases and you move from one usage tier to another. You can review your current usage limit in the limits page in your account settings.

You can set up:
- Notification thresholds to receive email alerts when usage exceeds a certain amount
- Monthly budgets to reject API requests once the budget is reached (Note: these limits are best effort with 5-10 minute delays)

## API Keys

The OpenAI API uses API keys for authentication. Visit your API keys page to retrieve the API key you'll use in your requests.

### Best Practices for API Keys:
- Avoid exposing keys in code or public repositories
- Store keys in a secure location
- Use environment variables or secret management services
- Enable usage tracking for monitoring (available on the API key management dashboard)
  - Note: API keys generated before Dec 20, 2023 need manual tracking enablement
  - All keys generated after Dec 20, 2023 have tracking enabled by default

## Staging Projects

As you scale, consider creating separate projects for staging and production environments. Benefits include:
- Isolation of development and testing work
- Prevention of accidental disruption to live applications
- Ability to limit user access to production
- Custom rate and spend limits per project

## Scaling Your Solution Architecture

When designing your production application, consider these key scaling areas:

### Horizontal Scaling
- Deploy additional servers/containers to distribute load
- Ensure architecture handles multiple nodes
- Implement load balancing mechanisms

### Vertical Scaling
- Upgrade server capabilities for additional load
- Design application to utilize additional resources effectively

### Caching
- Store frequently accessed data
- Improve response times
- Implement cache invalidation
- Options include:
  - Database storage
  - Filesystem storage
  - In-memory cache

### Load Balancing
- Distribute requests evenly across servers
- Options include:
  - Front-end load balancer
  - DNS round-robin
- Helps improve performance and reduce bottlenecks

## Managing Latency

### Request Lifecycle
1. Network: End user to API latency
2. Server: Time to process prompt tokens
3. Server: Time to sample/generate tokens
4. Network: API to end user latency

### Key Factors Affecting Latency

#### 1. Model Selection
- More capable models (e.g., gpt-4) offer better results but higher latency
- Smaller models provide faster, cheaper completions with potential trade-offs in accuracy

#### 2. Number of Completion Tokens
Optimize by:
- Setting lower max_tokens
- Including stop sequences
- Generating fewer completions
- Adjusting n and best_of parameters

#### 3. Streaming
- Set stream: true for faster first-token response
- Improves user experience for partial progress display

#### 4. Infrastructure
- Consider US-based infrastructure to minimize latency
- Future global redundancy planned

#### 5. Batching
- Batch multiple prompts in single requests (up to 20)
- Test effectiveness for your specific use case

## Managing Costs

### Cost Monitoring
- Set notification thresholds for email alerts
- Establish monthly budgets
- Use usage tracking dashboard
- Monitor token usage during billing cycles

### Cost Optimization Strategies
1. Reduce cost per token:
   - Use smaller models where appropriate
2. Reduce token count:
   - Use shorter prompts
   - Implement fine-tuning
   - Cache common queries

Use the interactive tokenizer tool to estimate costs and monitor token counts in API responses.

## MLOps Strategy

Consider these key areas when designing your MLOps strategy:

### Data and Model Management
- Manage training/fine-tuning data
- Track versions and changes

### Model Monitoring
- Track performance over time
- Detect issues or degradation

### Model Retraining
- Keep models updated
- Adapt to changing requirements

### Model Deployment
- Automate deployment processes
- Manage model artifacts

## Security and Compliance

### Key Considerations
- Assess data handling requirements
- Understand API data processing
- Identify applicable regulations
- Review security practices and compliance documentation

### Implementation Areas
- Data storage security
- Secure data transmission
- Data retention policies
- Privacy protections (encryption, anonymization)
- Secure coding practices
- Input sanitization
- Error handling

Refer to our Privacy Policy and Terms of Use for detailed compliance information.

## Safety Best Practices

Ensure application safety by:
- Conducting extensive testing
- Proactively addressing potential issues
- Limiting opportunities for misuse
- Following recommended safety guidelines

For the most current information, consult our security practices and trust and compliance portal.
