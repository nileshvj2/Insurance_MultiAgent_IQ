# Foundry Agents - AI Agent Implementations

This folder contains the Microsoft Foundry agent definitions that power the Insurance Multi-Agent IQ workflow.

## Overview

The Foundry-Agents folder contains YAML configuration files for specialized AI agents that work together to automate the insurance underwriting pre-screening process.

## Agents

### InsuranceE2EWorkflow.yaml
**End-to-End Workflow Orchestration**

The main workflow orchestrator that coordinates all individual agents in the insurance underwriting process. It manages:
- Agent execution sequence
- Data flow between agents
- Error handling and retries
- Workflow state management

### 1_IntakeAgent.yaml
**Intake Agent - Data Extraction**

Receives new insurance submissions and extracts structured information:
- Parses submission emails
- Extracts key data fields (applicant info, coverage details, etc.)
- Validates data completeness
- Structures data for downstream processing

**Inputs**: 
- Raw submission emails
- Attached documents

**Outputs**:
- Structured submission data (JSON)
- Extracted metadata

### 2_InsightAgent.yaml
**Insight Agent - Data Enrichment**

Enriches submission data with analytics and historical information:
- Queries FabricIQ for historical data
- Retrieves relevant analytics from OneLake
- Adds context from insurance databases
- Calculates risk indicators

**Inputs**:
- Structured submission data from Intake Agent
- Historical data queries

**Outputs**:
- Enriched submission with analytics
- Risk indicators and trends

### 3_TriageAgent.yaml
**Triage Agent - Validation and Assessment**

Validates submissions and creates comprehensive triage reports:
- Validates against underwriting policies (from FoundryIQ)
- Checks SLA compliance
- Identifies red flags and risk factors
- Generates recommended actions
- Creates triage summary

**Inputs**:
- Enriched submission data from Insight Agent
- Policy rules from FoundryIQ

**Outputs**:
- Triage summary report
- Risk assessment
- Recommended actions
- Red flags and alerts

### 4_SendTriage.yaml
**Triage Communication Agent - Report Distribution**

Sends the final triage report to underwriters:
- Formats triage report for readability
- Sends email notifications
- Updates workflow status
- Archives processed submissions

**Inputs**:
- Triage summary from Triage Agent
- Underwriter contact information

**Outputs**:
- Email notification sent
- Status updates
- Archived records

## Prerequisites

- Microsoft Foundry subscription
- Azure AI services
- Appropriate API keys and endpoints
- Access to FabricIQ and FoundryIQ resources

## Deployment

### Deploy Individual Agents

To deploy an agent to Microsoft Foundry:

```bash
# Using Azure CLI or Foundry CLI
foundry agent deploy --file 1_IntakeAgent.yaml
foundry agent deploy --file 2_InsightAgent.yaml
foundry agent deploy --file 3_TriageAgent.yaml
foundry agent deploy --file 4_SendTriage.yaml
```

### Deploy Complete Workflow

```bash
# Deploy the orchestrated workflow
foundry workflow deploy --file InsuranceE2EWorkflow.yaml
```

## Configuration

Each agent YAML file contains:
- Agent name and description
- Model configuration (Azure OpenAI, etc.)
- System prompts and instructions
- Input/output schemas
- Tool and function definitions
- Error handling rules

### Environment Variables

Set these variables before deploying:

```env
FOUNDRY_ENDPOINT=your_foundry_endpoint
FOUNDRY_API_KEY=your_api_key
AZURE_OPENAI_ENDPOINT=your_openai_endpoint
AZURE_OPENAI_API_KEY=your_openai_key
AZURE_OPENAI_DEPLOYMENT=your_deployment_name
```

## Testing

Test individual agents:

```bash
# Test Intake Agent
foundry agent test --file 1_IntakeAgent.yaml --input sample-submission.json

# Test full workflow
foundry workflow test --file InsuranceE2EWorkflow.yaml --input test-data/
```

## Monitoring

Monitor agent performance through:
- Microsoft Foundry dashboard
- Agent execution logs
- Performance metrics
- Error rates and latency

## Customization

### Modifying Agent Behavior

1. Edit the YAML file for the target agent
2. Update system prompts, tools, or parameters
3. Test changes locally
4. Redeploy to Foundry

### Adding New Agents

1. Create new agent YAML file
2. Define agent capabilities and tools
3. Update InsuranceE2EWorkflow.yaml to include new agent
4. Test integration
5. Deploy

## Agent Communication

Agents communicate through structured data:
- JSON schemas define input/output formats
- Workflow orchestrator manages data flow
- State is maintained across agent handoffs

## Best Practices

- **Prompts**: Keep system prompts clear and specific
- **Error Handling**: Define fallback behaviors
- **Testing**: Test each agent independently before workflow integration
- **Versioning**: Version control all YAML configurations
- **Monitoring**: Set up alerts for agent failures

## Troubleshooting

Common issues:
- **Agent deployment failures**: Check YAML syntax and required fields
- **Runtime errors**: Review agent logs in Foundry dashboard
- **Data flow issues**: Validate input/output schemas match
- **Performance issues**: Optimize prompts and model selection

## Support

For support with Foundry agents:
- Microsoft Foundry documentation
- Azure AI documentation
- Check agent execution logs
- Review YAML configuration syntax

## Version History

- **v1.0**: Initial agent implementations
  - Basic intake, insight, triage, and communication agents
  - E2E workflow orchestration
