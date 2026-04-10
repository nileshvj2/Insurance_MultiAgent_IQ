# FoundryIQ - Insurance Policy and Knowledge Base

This folder contains the Foundry IQ components that provide insurance domain knowledge, policy documents, and underwriting guidelines to the multi-agent system.

## Overview

FoundryIQ serves as the knowledge repository for the Insurance Multi-Agent IQ solution, containing:
- Insurance policy documents
- Underwriting guidelines and rules
- SLA (Service Level Agreement) definitions
- Compliance and regulatory requirements
- Domain-specific knowledge for insurance operations

## Components

### Policy Documents
- Standard insurance policy templates
- Coverage definitions
- Exclusions and limitations
- Terms and conditions

### Underwriting Guidelines
- Risk assessment criteria
- Approval workflows
- Underwriting rules and thresholds
- Industry best practices

### SLA Definitions
- Processing time requirements
- Response deadlines
- Escalation procedures
- Quality benchmarks

### Knowledge Base
- Insurance domain ontology
- Industry terminology
- Regulatory compliance rules
- Historical precedents

## Prerequisites

- Microsoft Foundry access
- Azure AI services
- Document indexing capabilities
- Vector search configuration (if applicable)

## Setup

1. **Organize Policy Documents**
   ```
   FoundryIQ/
   ├── policies/
   ├── guidelines/
   ├── sla/
   └── knowledge-base/
   ```

2. **Configure Document Indexing**
   - Set up document search
   - Configure vector embeddings (if using semantic search)
   - Index policy documents

3. **Link to Agents**
   - Connect to Triage Agent for policy validation
   - Enable access from Insight Agent for enrichment
   - Configure retrieval mechanisms

## Configuration

Update your environment variables:

```env
FOUNDRY_IQ_ENDPOINT=your_foundry_endpoint
FOUNDRY_IQ_API_KEY=your_api_key
DOCUMENT_INDEX_NAME=insurance_policies
```

## Usage

The FoundryIQ knowledge base is queried by:
- **Triage Agent**: To validate submissions against policies and guidelines
- **Insight Agent**: To retrieve relevant policy information
- **Intake Agent**: To understand submission requirements

## Document Management

### Adding New Policies
1. Add document to appropriate folder
2. Update indexing
3. Verify agent access

### Updating Guidelines
1. Version control for guideline changes
2. Re-index updated documents
3. Test agent responses

## Integration

FoundryIQ integrates with:
- Microsoft Foundry Agent Service
- Azure AI Search (for document retrieval)
- Agent workflows defined in Foundry-Agents

## Best Practices

- Keep policy documents up to date
- Version control all guidelines
- Regular audit of knowledge base accuracy
- Test agent retrieval for key policies

## Support

For issues with Foundry IQ:
- Microsoft Foundry documentation
- Azure AI Search documentation
- Contact support for indexing issues
