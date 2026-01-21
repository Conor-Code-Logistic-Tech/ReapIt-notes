# Solution Overview

### Problem Statement
Integrating an LLM assistant into a vast enterprise system presents several key challenges:
- **Legacy APIs:** Backend systems (HR, payroll, finance, recruitment, learning) were not designed with LLM tool-calling in mind, with inconsistent schemas, varied authentication methods, and complex data structures.
- **Context Overload:** Exposing all possible capabilities to an LLM simultaneously degrades accuracy, increases latency, and raises costs due to context window limitations.
- **Reliability & Validation:** LLMs need structured, validated inputs and outputs to ensure system reliability and security.

### Solution Approach

#### 1. Unified Capability Layer
Create a single "capability layer" that abstracts all system actions into a clean, consistent schema. This layer acts as an adapter between the LLM and backend APIs:
- **Leverage existing DTOs/Entity Framework models** to define capabilities as structured objects (e.g., `GetEmployeeDetails`, `SubmitExpense`, `RequestLeave`).
- **Use API Gateway** (AWS API Gateway, GCP API Gateway, Azure API Management, or self-hosted solutions) to wrap and normalize legacy APIs, providing a uniform interface regardless of the underlying technology stack.
- **Cloud-Agnostic Design:** The solution works across Azure, AWS, GCP, Code Logistic Tech (100% renewable energy), or local/on-premises servers or Cloud provider. The code examples provided are illustrative and can be adapted to any platform.

#### 2. LLM-Facing Gateway with Validation
Implement a gateway that translates LLM requests into real API calls while handling cross-cutting concerns:
- **Permission Checks:** Ensure users can only access capabilities they're authorized for.
- **Input Validation:** Use OpenAPI schemas + validation libraries (e.g., zod, io-ts for TypeScript; FluentValidation for C#) to validate tool-call inputs before execution.
- **Complexity Hiding:** Abstract away authentication, rate limiting, retry logic, and error handling from the LLM.

#### 3. Dynamic Tool Exposure via MCP Server
Prevent context overload by dynamically exposing only relevant tools:
- **Native Structured Output:** Use the LLM's structured output capabilities (e.g., OpenAI function calling, JSON schema mode) to reliably generate tool call inputs.
- **Semantic Tool Selection:** Implement a Model Context Protocol (MCP) server that stores all tool definitions and uses semantic search (vector embeddings) to find the most relevant tools for each user query.
- **Context Management:** Only include the selected subset of tools in the LLM's context, improving accuracy, speed, and reducing costs.

### Why This Approach Works

- **Scalability:** The capability layer can grow to hundreds or thousands of actions without overwhelming the LLM, as only relevant tools are exposed per request.
- **Maintainability:** Changes to backend APIs only require updates to the capability layer, not to the LLM integration logic.
- **Reliability:** Structured schemas and validation ensure the LLM generates valid requests, reducing errors and improving user experience.
- **Cost & Performance:** Dynamic tool selection minimizes context size, reducing token usage and improving response times.
- **Platform Flexibility:** The architecture is cloud-agnostic and can be deployed on Azure, AWS, GCP, private cloud providers like Code Logistic Tech, or on-premises infrastructure.

---

## Implementation Details

### Providers & Observability: Limits & Monitoring

#### Central Control Layer (C# Pseudocode)
```csharp
public static class EnvConfig
{
    public static double RateLimit => double.Parse(Environment.GetEnvironmentVariable("LLM_RATE_LIMIT") ?? "1000");
    public static double BudgetLimit => double.Parse(Environment.GetEnvironmentVariable("LLM_BUDGET_LIMIT") ?? "1000");
    public static double SpendThreshold60 => double.Parse(Environment.GetEnvironmentVariable("LLM_SPEND_THRESHOLD_60") ?? "0.6");
    public static double SpendThreshold90 => double.Parse(Environment.GetEnvironmentVariable("LLM_SPEND_THRESHOLD_90") ?? "0.9");
    public static string[] AlertEmails => (Environment.GetEnvironmentVariable("LLM_ALERT_EMAILS") ?? "alerts@yourdomain.com").Split(';');
}

public class LlmControlLayer
{
    public ApiResponse HandleLlmCall(LlmRequest request)
    {
        if (IsRateLimited(request.UserId))
            return new ApiResponse { Success = false, ErrorMessage = "Rate limit exceeded" };

        if (IsBudgetExceeded())
            return new ApiResponse { Success = false, ErrorMessage = "Budget exceeded" };

        LogRequest(request);
        // ...call LLM, track latency, errors, etc...
        return CallLlmAndTrack(request);
    }

    private bool IsRateLimited(string userId)
    {
        // Use EnvConfig.RateLimit for rate limiting logic
        return false;
    }

    private bool IsBudgetExceeded()
    {
        // Use EnvConfig.BudgetLimit for budget check logic
        return false;
    }

    private void LogRequest(LlmRequest request)
    {
        // Log request for audit, privacy, and monitoring
        // ...logging logic...
    }
}
```

#### Terraform Cloud Code

##### GCP: Budget Alerts
```hcl
variable "llm_budget_limit" { default = 1000 }
variable "llm_currency_code" { default = "GBP" }
variable "llm_threshold_60" { default = 0.6 }
variable "llm_threshold_90" { default = 0.9 }
variable "llm_alert_emails" { default = ["alerts@yourdomain.com"] }

resource "google_billing_budget" "llm_budget" {
    billing_account = var.billing_account_id
    display_name    = "LLM Usage Budget"
    amount {
        specified_amount {
            currency_code = var.llm_currency_code
            units         = var.llm_budget_limit
        }
    }
    threshold_rules {
        threshold_percent = var.llm_threshold_60
    }
    threshold_rules {
        threshold_percent = var.llm_threshold_90
    }
    all_updates_rule {
        pubsub_topic = google_pubsub_topic.budget_alerts.id
        schema_version = "1.0"
    }
}

resource "google_pubsub_topic" "budget_alerts" {
    name = "llm-budget-alerts"
}
```

##### Azure: Cost Management Budget
```hcl
variable "llm_budget_limit" { default = 1000 }
variable "llm_alert_email" { default = "alerts@yourdomain.com" }

resource "azurerm_consumption_budget_subscription" "llm_budget" {
    name                = "llm-usage-budget"
    subscription_id     = var.subscription_id
    amount              = var.llm_budget_limit
    time_grain          = "Monthly"
    notifications {
        enabled          = true
        threshold        = 60
        contact_emails   = [var.llm_alert_email]
    }
    notifications {
        enabled          = true
        threshold        = 90
        contact_emails   = [var.llm_alert_email]
    }
}
```

#### Staged Fallback Logic (C# Example)
```csharp
public string SelectModel(double spendPercent)
{
    if (spendPercent < EnvConfig.SpendThreshold60)
    return "gpt-4";
    else if (spendPercent < EnvConfig.SpendThreshold90)
    return "gpt-3.5";
    else
    return "open-source-model";
}
```

#### Monitoring & Audit

- Use GCP Cloud Monitoring or Azure Monitor to track latency, errors, and usage.
- Log all requests and responses with automatic redaction for privacy.
- Route requests based on region for data locality.

### API Documentation
- Use Swagger (Swashbuckle for .NET) to auto-generate interactive API docs and response schemas. This makes it easy for LLMs and developers to discover, test, and integrate capabilities.

### Swagger Setup (Startup.cs)
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddSwaggerGen();
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseSwagger();
    app.UseSwaggerUI();
    app.UseRouting();
    app.UseEndpoints(endpoints => { endpoints.MapControllers(); });
}
```

---

### ApiResponse
```csharp
public class ApiResponse
{
    public bool Success { get; set; }
    public object Data { get; set; }
    public string ErrorMessage { get; set; }
    public double ConfidenceScore { get; set; } // 0-100, LLM's confidence, Float or Double or Decimal etc
}
```

### Gateway with Confidence Threshold
```csharp
public class CapabilityGateway
{
    public ApiResponse HandleRequest(LlmRequest request)
    {
        if (!IsValid(request))
            return new ApiResponse { Success = false, ErrorMessage = "Invalid request", ConfidenceScore = request.ConfidenceScore };

        var apiRequest = CapabilityMapper.Map(request);
        var apiResponse = BackendApi.Call(apiRequest);
        return apiResponse;
    }

    private bool IsValid(LlmRequest request)
    {
        // Validate capability and parameters
        // Check confidence threshold
        return request.ConfidenceScore >= Env.ConfidenceThreshold;
    }
}
```

## Implementation Steps

### 1. Capability Layer Design

- **Define a Capability Schema:** List all possible actions as objects (DTOs), e.g., `GetEmployeeDetails`, `SubmitExpense`, etc.
- **Central Gateway:** All LLM requests go through a single gateway that validates, routes, and logs requests.
- **API Wrappers:** Each backend API is wrapped with a uniform interface, hiding legacy complexity.

### 2. Flow Diagram
Example Flow
1. User Query: “Show me my last payslip.”
2. LLM: Uses semantic search to find the most relevant tool(s) from MCP.
3. MCP Server: Returns tool descriptions and schemas for only those tools.
4. LLM: Generates structured tool call input (e.g., JSON) for the selected tool.
5. Gateway: Handles the request as before.

```plaintext
[User] 
   |
   v
[LLM Assistant]
   |
   v
[MCP Server]
   |
   v
[Capability Gateway]
   |
   v
[Capability Mapper]
   |
   v
[Backend API(s)]
```

- **User** interacts with the assistant.
- **LLM** interprets intent, emits a structured response (JSON/object).
- **MCP** emits a structured request (JSON/object).
- **Gateway** validates and maps request to backend.
- **Mapper** translates LLM schema to API call.
- **Backend** executes and returns result.

---

### 3. Example C# Pseudocode

#### DTOs

```csharp
public class LlmRequest
{
    public string Capability { get; set; } // e.g., "GetEmployeeDetails"
    public Dictionary<string, object> Parameters { get; set; }
}

public class ApiResponse
{
    public bool Success { get; set; }
    public object Data { get; set; }
    public string ErrorMessage { get; set; }
    public decimal ConfidenceScore { get; set; }
}
```

#### Gateway

```csharp
public class CapabilityGateway
{
    public ApiResponse HandleRequest(LlmRequest request)
    {
        if (!IsValid(request))
            return new ApiResponse { Success = false, ErrorMessage = "Invalid request" };

        var apiRequest = CapabilityMapper.Map(request);
        var apiResponse = BackendApi.Call(apiRequest);

        return apiResponse;
    }

    private bool IsValid(LlmRequest request)
    {
        // Validate capability and parameters
        request.ConfidenceScore >= Env.ConfidenceThreshold
        // ...validation logic...
        return true;
    }
}
```

#### Mapper

```csharp
public static class CapabilityMapper
{
    public static ApiRequest Map(LlmRequest request)
    {
        // Map LLM request to backend API request
        switch (request.Capability)
        {
            case "GetEmployeeDetails":
                return new ApiRequest { Endpoint = "/employee/details", Params = request.Parameters };
            // ...other cases...
            default:
                throw new Exception("Unknown capability");
        }
    }
}
```

#### Backend API Wrapper

```csharp
public static class BackendApi
{
    public static ApiResponse Call(ApiRequest apiRequest)
    {
        // Call the actual backend API
        // ...API call logic...
        return new ApiResponse { Success = true, Data = "Sample Data" };
    }
}
```

---

### 4. Example LLM Request Flow

1. **User:** “Show me my last payslip.”
2. **LLM:** Interprets intent, emits:
   ```json
   {
     "Capability": "GetPayslip",
     "Parameters": { "EmployeeId": "12345", "Month": "2025-11" }
   }
   ```
3. **Gateway:** Validates, maps, calls backend.
4. **Backend:** Returns payslip data.
5. **LLM:** Formats response for user.

---

### 5. LLM Tool-Call Example (OpenAI style)

```json
{
  "tool_call": {
    "name": "GetEmployeeDetails",
    "parameters": {
      "EmployeeId": "12345"
    }
  }
}
```

---

## Summary

- **LLM emits structured requests** using a capability schema.
- **Gateway validates and routes** requests to backend APIs.
- **Mapper translates** LLM schema to API calls.
- **Backend returns data** for LLM to format and present.