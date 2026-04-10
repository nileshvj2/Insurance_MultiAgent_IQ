# Insurance Multi-Agent IQ

An intelligent multi-agent system designed to streamline insurance processes through AI-powered automation and intelligent decision-making.

<img width="1288" height="855" alt="image" src="Insurance-Underwriting MultiAgent Workflow.jpg"/>

## Overview

Insurance Multi-Agent IQ is a agentic workflow solution that leverages multi agent architecture and Microsoft's IQ platform to handle various aspects of insurance operations to assist underwriters process submissions/claims. It helps with the triage summary report, recommended next steps and red flags based on underwriting rules, SLAs, policies and enterprise analytics data stored in Lakehouse and semantic models in onelake.

## Features

- **Multi-Agent Architecture**: Specialized AI agents for different insurance tasks
- **Underwriting workflow automation**: Using Foundry IQ, Fabric IQ and WorkIQ end to end automation of underwriting pre-screening process.
- **Automated Claims Processing**: Streamlined claims handling and assessment
- **Policy Analysis**: Automated policy comparison and recommendation
- **Customer Service**: AI-powered customer support and assistance
- **Risk Assessment**: Intelligent risk evaluation and underwriting support

## Getting Started


### Configuration


- Will be added here soon


## Usage

- Example usage will be added here


## Architecture

The system consists of multiple specialized agents:

- **Intake Agent**: Receives new submissions/claims and extracts information from submission emails to structured format.
- **Insight Agent**: This agent enriches information received through intake agent by using analytics data and insurance database records.
- **Triage Agent**: Validates submissions against policies, underwriting guidelines, SLAs and create final triage summary with recommended action items, red flags, and risk indicators for underwriter review process.
- **Triage Communication Agent**: This agent sends final triage summary report to underwriter. 

## Project Structure

```
Insurance_MultiAgent_IQ/
├── FabricIQ/                                     # Fabric Lakehouse, Semantic model, Ontology and Data Agent components
├── Foundry-Agents/                               # Foundry agent implementations
│   ├── InsuranceE2EWorkflow.yaml                 # End-to-End Insurance Workflow orchestration
│   ├── 1_IntakeAgent.yaml                        # Intake Agent - Receives and extracts submission data
│   ├── 2_InsightAgent.yaml                       # Insight Agent - Enriches with analytics data
│   ├── 3_TriageAgent.yaml                        # Triage Agent - Validates and creates summary
│   └── 4_SendTriage.yaml                         # Triage Communication Agent - Sends reports
├── FoundryIQ/                                    # Foundry IQ and policy documents

```

## Deployment

This solution consists of multiple components that need to be deployed and configured. Refer to the README files in each component folder for detailed deployment instructions:

### Component Deployment Guides

- **[FabricIQ](FabricIQ/README.md)** - Deploy Fabric IQ components -Fabric Lakehouse, Semantic models, Ontology, and Data Agent.
- **[FoundryIQ](FoundryIQ/README.md)** - Set up storage account, policy documents, AI search index and knowledgebase for Foundry IQ.
- **[WorkIQ](WorkIQ/README.md)**: No additional set up is required (Pre-req - M365 Copilot license to use WorkIQ features)
- **[Foundry-Agents](Foundry-Agents/README.md)** - Deploy AI agents to Microsoft Foundry Agent service and  set up end to end workflow orchestration.

For detailed step-by-step instructions, please refer to the individual README files in each folder. 

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## Acknowledgments

- Built with Microsoft Azure AI services
- Powered by Azure OpenAI
- Developed using Microsoft Foundry Agent Service

## Disclaimer

This solution is created for demo purpose only.

---

**Status**: 🚧 Under Development

