User: [Link to work item #12345]
I want to refine the PBI for the installation of the Contoso Message Broker service in the deployment pipeline. 

GitHub Copilot: I'll help you refine the PBI for Contoso Message Broker installation. Let me first retrieve the current PBI content and gather relevant context.

[Retrieved work item from Azure DevOps and searched codebase for relevant documentation and installation patterns]

Perfect! I now have enough context. Let me present the refinement plan.

## Analysis Complete

**Current PBI State**: The PBI is essentially a blank template with placeholder text. It needs complete refinement based on the Contoso Message Broker component installation requirements within the deployment pipeline.

## Refinement Plan

### Template Compliance Issues
- Missing all mandatory sections (Description, Background Information, Solution realisation, Testplan, Acceptance Criteria)
- Placeholder content only

### TL;DR
Transform the blank Contoso Message Broker installation PBI into a complete work item defining the business outcome of automated message broker installation within the deployment pipeline, following the established component installation pattern. Focus on the message broker's ability to install without manual intervention across all environments with instance-specific configurations.

### Steps

1. **Define user story and business context** from HLD Design focusing on Contoso Message Broker's role as the message bus interface between application components

2. **Extract solution approach** from existing component pattern showing the standardized installation process (mode detection → configuration → installer execution → validation)

3. **Document message broker-specific requirements** including database configuration, instance-prefixed service names, port configuration (connection port, monitor port), and dependency on related services being installed

4. **Create Given-When-Then test scenarios** covering fresh install, upgrade, instance isolation validation, and dependency verification

5. **Define business-verifiable acceptance criteria** focused on service availability, multi-instance support, and environment consistency (avoiding technical implementation details like file paths or script names)

6. **Link to message broker documentation** in Documentation section for operational context

### Further Considerations

1. **Database Strategy**: The message broker uses separate databases for different purposes. Should the PBI address both installations or focus on the primary database only, with others as separate PBIs?

2. **Validation Complexity**: Message broker validation involves checking the service status AND verifying the message/system tables are properly initialized. Should acceptance criteria include functional message passing validation, or is service running sufficient?

3. **Dependencies on Related PBIs**: The HLD shows the message broker is installed after other services in the deployment sequence. Should this PBI explicitly reference/link to the dependency installation PBIs to maintain traceability in the backlog?

---

Would you like me to proceed with creating the complete refined PBI content, or would you like to provide guidance on any of the considerations above first?

User: refine the PBI

GitHub Copilot: I'll now present the complete refined PBI for your review.

---

## Complete Updated PBI: Contoso Message Broker Installation

**Description**

As a Platform Engineer
I want the deployment pipeline to automatically install and configure the Contoso Message Broker component
So that application instances can reliably exchange messages between services without manual installation effort or configuration errors.

[Background Information]:

The Contoso Message Broker is a critical messaging infrastructure component that enables asynchronous communication between application services (data services, API services, integration services, etc.). Messages are stored in a dedicated SQL Server database (with instance-specific naming like `MSGBROKER_INST1`, `MSGBROKER_INST2`), allowing services to send and receive messages even when temporarily unavailable.

Currently, Contoso Message Broker installation is performed manually, which creates risk for:
- Configuration inconsistencies across environments
- Port conflicts when multiple instances share physical servers
- Delayed environment provisioning
- Human error in database setup and service configuration

This PBI is part of the automated deployment pipeline project and follows the established component installation pattern used by other components.

[Stakeholders]:
- Platform Team (implementation)
- Operations team (deployment and monitoring)
- Infrastructure team (infrastructure dependencies)

[Dependencies]:
- Data Service installation PBI must be completed first (message broker requires data service to be running)
- Database schema installation PBI (database schema prerequisite)
- Infrastructure configuration (SQL Server, network, service accounts)

[Documentation link]:
- Message Broker Application Documentation
- Message Broker Monitor Guide
- High Level Design - Deployment Pipeline

[Solution realisation]:

Follow the established component installation pattern:
1. **Mode Detection**: Automatically determine whether to perform fresh install, upgrade, reconfigure, or skip based on Windows registry inspection
2. **Configuration Preparation**: Generate instance-specific registry settings from hierarchical configuration (global → environment → instance overrides)
3. **Installation Execution**: Run message broker installer with appropriate parameters for detected mode
4. **Post-Installation**: Initialize database tables (message, system), configure service startup, validate connectivity

The implementation will use PowerShell scripts orchestrated by CI/CD pipeline templates, consistent with other component approaches. Each instance receives unique database names, service names (e.g., `INST1_MSGBROKER`, `INST2_MSGBROKER`), connection ports, and monitor ports to prevent conflicts on shared servers.

[Testplan]:

**Scenario 1: Fresh Installation**
- **Given** a server with no existing message broker installation and data service running
- **When** the pipeline executes message broker installation for instance INST1
- **Then** the message broker service starts successfully with instance-specific naming (`INST1_MSGBROKER`)
- **And** the database (`MSGBROKER_INST1`) contains initialized message and system tables
- **And** monitoring confirms service is accepting connections on configured ports

**Scenario 2: Upgrade**
- **Given** an existing message broker installation at version 2.1
- **When** the pipeline executes installation targeting version 2.2
- **Then** the service upgrades to version 2.2 preserving existing configuration
- **And** existing message history remains accessible in the database
- **And** all previously configured sender/receiver entries persist in the system table

**Scenario 3: Multi-Instance Isolation**
- **Given** two instances (INST1 and INST2) deployed on the same physical server
- **When** the pipeline installs message broker for both instances
- **Then** each instance has a separate service with unique service names
- **And** each instance uses separate database names and port allocations
- **And** one instance's service can restart without affecting the other instance

**Scenario 4: Dependency Validation**
- **Given** message broker installation is triggered but data service is not running
- **When** the pipeline validates dependencies
- **Then** the installation fails with a clear error message indicating data service prerequisite
- **And** no partial installation or configuration changes are applied

**Scenario 5: Idempotent Re-execution**
- **Given** message broker is already installed at the target version
- **When** the pipeline re-executes without the reconfigure flag
- **Then** the installation skips without errors
- **And** no service disruption occurs

**Acceptance Criteria**

* Message broker installs successfully on servers without manual intervention for all environments (DEV, TEST, STAGING, PROD)
* Each application instance has an isolated message broker installation with unique service names, database names, and port allocations
* The installation pipeline automatically detects and performs the appropriate action: fresh install, upgrade, reconfigure, or skip
* Message broker service starts after installation and remains operational with database accessible
* Installation supports multi-instance architecture where multiple instances coexist on shared physical servers without conflicts
* Message broker installation validates that data service prerequisite is met before proceeding
* The installation is idempotent, allowing safe re-execution without causing errors or service disruption
* Operations team can validate successful installation using monitoring commands (connection test, status check)

[Release Notes]:
N/A

---

## Validation Checklist

✅ All mandatory sections present  
✅ Description follows "As.../I want.../So that..." format  
✅ Solution realisation is high-level (component pattern, no script names/paths)  
✅ Testplan uses Given-When-Then format  
✅ Acceptance Criteria focused on business outcomes (service availability, multi-instance support, automation)  
✅ No implementation details in AC (no file paths, no PowerShell script names)  
✅ Documentation links provided for context  
✅ Dependencies and stakeholders identified  

---

**Please review this refined PBI.** Would you like me to:
1. Make any adjustments to the content?
2. Update Azure DevOps with this refined version (requires your explicit approval)?
3. Adjust the scope or level of detail in any section?

User: looks good update the PBI

GitHub Copilot: I'll update the PBI in Azure DevOps now.

Ran `wit_update_work_item`

✅ **PBI Updated Successfully!**

The PBI [#12345 - Automate Contoso Message Broker Installation] has been updated with:

- **Complete user story** defining the business value of automated message broker installation
- **Background information** explaining the message broker's role and current manual installation risks
- **High-level solution approach** following the established component installation pattern
- **5 Given-When-Then test scenarios** covering fresh install, upgrade, multi-instance isolation, dependency validation, and idempotency
- **8 business-verifiable acceptance criteria** focused on outcomes, not implementation
- **Links to documentation** for message broker application details and monitoring guides
- **Stakeholders and dependencies** clearly identified

The PBI now follows the standard template and is ready for backlog refinement discussions with the team. The acceptance criteria focus on business outcomes that can be verified by operations staff without requiring technical knowledge of PowerShell scripts or file structures.