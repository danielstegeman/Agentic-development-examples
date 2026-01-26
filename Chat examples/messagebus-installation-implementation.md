## About this example

This example demonstrates implementing a Message Broker installation pipeline by creating a comprehensive plan from a backlog item, researching existing implementation patterns, refining the plan based on discovered specifics, and executing the full implementation following established architectural patterns. It is based on the refinement from the [Refinement of Contoso Message Broker Installation PBI](messagebus-refinement.md) example.

The goal was to achieve an implementation that is about 90% correct after the initial implementation, so the last 10% of changes can be made based on testing and feedback. This was highly successful, with only minor adjustments in the installer parameters needed after initial testing.

This item had an estimated effort of about 5-7 work days based on story points, but the implementation was completed in between meetings in a few hours.

## Why did this work so well?
* There was a clearly refined backlog item with acceptance criteria to guide the plan creation. 
* This is the 3rd component installation pipeline implemented using this pattern (after Data Service and API Service), so there was strong existing context to build from.
* There were legacy scripts to research for specifics on the Message Broker installation process.
* A high level design document exists for the pipeline architecture, providing additional context on how the component fits into the overall system.
* The use of plan mode before implementation allowed for gathering all necessary context and refining the approach before writing any code.
---

### User

[Link to work item #12345]

Create a plan to implement this backlog item. Use the devops mcp to retrieve it. Make sure to look at the implementation of the Data Service install and the high level design.

---

### GitHub Copilot

I'll gather context about the backlog item and related implementation details to create a comprehensive plan. 

Gathered backlog item details and searched codebase for Data Service implementation patterns

Ran `wit_get_work_item`

Searched codebase for "Contoso Message Broker installation setup component install", 1 result

Perfect! I have enough context. Let me create the plan for implementing Message Broker installation.

## Plan: Automate Message Broker Installation Pipeline

Implement the Message Broker component installation following the established Data Service/API Service pattern. The Message Broker is a critical messaging infrastructure that enables asynchronous communication between ApplicationX services via a dedicated SQL Server database. This implementation will support fresh installation, upgrade, reconfigure, and skip modes across all OTAP environments with multi-instance isolation on shared servers.

### Steps

1. **Create Message Broker PowerShell Installation Script** - Develop ExecuteInstallation.ps1 following Data Service pattern with parameters for `Instance`, `Environment`, `Reconfigure`, `TargetVersion`, `ModulePath`, `ServerInstance`, `Database`, `[Redacted]`, `Port`, `MonitorPort`, and `WorkingDirectory`. Implement mode detection using `Get-InstallationMode`, registry file generation, service stop/start, and MessageBrokerSetup.exe execution with instance-specific service naming (e.g., `INST1_MessageBroker`).

2. **Implement Dependency Validation Script** - Create PreSetupChecks.ps1 to validate SQL Server connectivity, Message Broker database existence (`MSGBROKER_INST1`, `MSGBROKER_PROD1`), port availability (connection and monitor ports), Data Service running status, and service account credentials from Key Vault.

3. **Configure Message Broker Registry Settings** - Add `MessageBroker` section to AppRegistry.json with hierarchical configuration (global → environment → instance overrides) for cache settings, database connection parameters, message retention policies, and service startup behavior. Verify Message Broker-specific registry requirements via VendorX documentation.

4. **Update Pipeline Template** - Replace placeholder install.yml with functional template that downloads Message Broker setup artifact from blob storage using shared/download-component-artifact.yml, invokes PreSetupChecks.ps1, and executes ExecuteInstallation.ps1 with error handling.

5. **Integrate Message Broker into Server Deployment Workflows** - Modify server/install-application-server.yml to invoke Message Broker installation after Deployment Service but before Integration Service, ensuring dependency order (Data Service → Deployment Service → Message Broker → Integration Service). Update AppConfig.json with Message Broker-specific configuration if needed (database naming, port allocation scheme).

### Further Considerations

1. **Message Broker Database Initialization** - How are Message Broker message/system tables created? Does MessageBrokerSetup.exe handle this automatically, or is SQL script execution required? Check VendorX installation guide for database bootstrap requirements.

2. **Port Allocation Scheme** - Should Message Broker use a different port pattern than Data Service (3XXX/4XXX)? Document the port allocation strategy to prevent conflicts with other components (e.g., 5XXX for Message Broker connection, 6XXX for monitor).

3. **Service Account Strategy** - Does Message Broker require a dedicated service account (`svc-msgbroker-{env}`) or share with Data Service? Verify CompanyX TAB requirements and update Key Vault secrets accordingly.

4. **MessageBrokerSetup.exe Parameter Discovery** - Extract full installer parameter list via `MessageBrokerSetup.exe /?` or VendorX documentation. Critical parameters likely include `/COMPONENT`, `/DBSQLSERVER`, `/DBNAME`, `/PORT`, `/MONITORPORT`, `/REGISTRYINPUTFILE`, `/INSTALL`, `/UPGRADE`, `/RECONFIGURE`, `/USEDEFAULT`.

5. **Database vs Data Service Dependency** - Does Message Broker installation require an existing Data Service database, or is it independent? Clarify whether Message Broker database references Data Service schema or is fully isolated.

---

### User

research the existing message broker install script for parameters and patterns and incorporate it for the specifics on how to install message broker

---

### GitHub Copilot

I'll analyze the existing Message Broker batch installation script to extract critical parameters and patterns, then update the plan with specific implementation details. 

Searched codebase for existing batch scripts and registry configuration files

Perfect! Now I have all the information needed. Let me create the updated plan incorporating the specific Message Broker installation details:

## Plan: Automate Message Broker Installation Pipeline

Implement the Message Broker component installation following the established Data Service/API Service pattern. Message Broker is a critical messaging infrastructure that stores and routes messages between ApplicationX services via a dedicated SQL Server database (`MSGBROKER_PROD1`, `MSGBROKER_INST1`, etc.). Based on existing batch scripts, MessageBrokerSetup.exe requires specific database sizing parameters, encrypted credentials, and manual SQL login creation. This implementation will support fresh installation, upgrade, reconfigure, and skip modes across all OTAP environments with multi-instance isolation on shared servers.

### Steps

1. **Create Message Broker PowerShell Installation Script** - Develop ExecuteInstallation.ps1 with parameters matching existing batch script: `Instance`, `Environment`, `Reconfigure`, `TargetVersion`, `ModulePath`, `ServerInstance`, `Database`, `[Redacted]`, `[Redacted]`, `Port`, `MonitorPort`, `WorkingDirectory`, `RegistrySettingsConfigPath`. [REDACTED] , mode detection via `Get-InstallationMode`, registry file generation from MessageBroker_main.txt and MessageBroker.txt, service stop/start, and MessageBrokerSetup.exe execution with `/COMPONENT="MessageBroker-SQL Server (x64)"`, `/DATADEVICESIZE=1000`, `/LOGDEVICESIZE=500`, `/DBUNICODE=1`, `/DOWNGRADEFILES`, `/TASK!="Instance scheduling"`, and instance-specific service naming (e.g., `INST1_MessageBroker`). Port pattern: `5XXX` for connection (e.g., 5211), `6XXX` for monitor (e.g., 6211).

2. **Implement Dependency Validation Script** - Create PreSetupChecks.ps1 to validate SQL Server connectivity, verify Data Service running status (Message Broker depends on Data Service), check Message Broker database existence (`MSGBROKER_INST1`, `MSGBROKER_PROD1`), validate port availability (5XXX and 6XXX), verify service account credentials from Key Vault (`[Redacted]`, `[Redacted]`), and confirm SQL login "mb" exists with correct credentials (critical manual prerequisite from existing process). Add validation warning if "mb" login missing.

3. **Configure Message Broker Registry Settings** - Add `MessageBroker` section to AppRegistry.json using existing registry template structure (global → environment → instance overrides). Base settings from MessageBroker_main.txt: `Unicode=1` (required), and MessageBroker.txt: `db_options="sql_login_timeout=30"`, `RestartRetries="1"`. Verify against VendorX MessageBrokerSetup documentation for additional cache/connection settings. Registry path: `HKEY_LOCAL_MACHINE\SOFTWARE\ApplicationX\MessageBroker\MessageBroker{Instance}`.

4. **Update Pipeline Template and Configuration** - Replace placeholder install.yml with functional template downloading Message Broker artifact via shared/download-component-artifact.yml, invoking PreSetupChecks.ps1 and ExecuteInstallation.ps1 with error handling. Add Message Broker port allocation to AppConfig.json following Data Service pattern: `msgbrokerPort: 5XXX`, `msgbrokerMonitorPort: 6XXX` where X = instance number. Update shared/download-component-artifact.yml if Message Broker not already in supported components list.

5. **Integrate Message Broker into Server Deployment Workflows** - Modify server/install-application-server.yml to invoke Message Broker installation after Deployment Service but before Integration Service, respecting dependency order (Data Service → Deployment Service → **Message Broker** → Integration Service). Ensure Message Broker deployment targets application server agent (not batch/FTP), matching Data Service server targeting pattern. Pass `[Redacted]` and `[Redacted]` from environment secrets loaded via Key Vault.

### Further Considerations

1. **Manual "mb" SQL Login Automation** - Existing batch script requires manual creation of SQL login "mb" before installation (pauses with warning message). Should we automate this via PowerShell (`Invoke-Sqlcmd`) in PreSetupChecks.ps1, or preserve manual step for security approval? If automated, verify login credentials match `[Redacted]` from Key Vault.

2. **Database Uninstall Strategy** - Existing batch script comments suggest manual DB deletion after uninstall (`sqlcmd -E -S %DBSQLSERVER% -Q "drop database MSGBROKER_PROD1"`). Should automated uninstall include database cleanup, or leave orphaned databases for manual review? Consider safety implications for upgrade scenarios.

3. **DATADEVICESIZE/LOGDEVICESIZE Tunability** - Existing script hardcodes 1000MB data / 500MB log sizes. Should these be configurable per environment in AppConfig.json, or remain static? Production environments may need larger allocations based on message volume.

4. **TASK Parameter Exclusion** - Existing script uses `/TASK!="Instance scheduling"` to exclude a specific task. Verify this is still required in current versions or if MessageBrokerSetup behavior changed. Check VendorX documentation for task component list.

5. **SILENT Mode .pyc File Workaround** - Existing batch script conditionally disables `/SILENT` mode if `silent.txt` file absent, with comment about .pyc file replacement failures. Should automated pipeline always use `/SILENT`, or implement similar conditional logic based on version/environment? Test `.pyc` replacement behavior in current versions.

---

### User

Remove step 6. not relevant for now. Ports should be explicitely configured, like API Service. The registry file should be passed to the installer like in Data Service. DSEncrypt does not need to be used anymore, it is handled by an encrypt param.

---

### GitHub Copilot

## Plan: Automate Message Broker Installation Pipeline

Implement the Message Broker component installation following the established Data Service/API Service pattern. Message Broker is a critical messaging infrastructure that stores and routes messages between ApplicationX services via a dedicated SQL Server database (`MSGBROKER_PROD1`, `MSGBROKER_INST1`, etc.). Based on existing batch scripts, MessageBrokerSetup.exe requires specific database sizing parameters, encrypted credentials, and registry configuration. This implementation will support fresh installation, upgrade, reconfigure, and skip modes across all OTAP environments with multi-instance isolation on shared servers.

### Steps

1. **Create Message Broker PowerShell Installation Script** - Develop ExecuteInstallation.ps1 with parameters matching Data Service pattern: `Instance`, `Environment`, `Reconfigure`, `TargetVersion`, `ModulePath`, `ServerInstance`, `Database`,  `Port`, `MonitorPort`, `WorkingDirectory`, `RegistrySettingsConfigPath`. Implement mode detection via `Get-InstallationMode`, registry file generation using `Create-MessageBrokerRegistryFile` function (loading from AppRegistry.json), service stop/start using `Stop-ComponentService`/`Start-ComponentService`, and MessageBrokerSetup.exe execution with `/COMPONENT="MessageBroker-SQL Server (x64)"`, `/DBSQLSERVER={server}\{instance}`, `/DATADEVICESIZE=1000`, `/LOGDEVICESIZE=500`, `/DBUNICODE=1`, `/PORT={port}`, `/MONITORPORT={monitorPort}`, `/REGISTRYINPUTFILE={regfile}`, `/DOWNGRADEFILES`, `/TASK!="Instance scheduling"`, `/ENCRYPT` parameter for credential handling, and instance-specific service naming (e.g., `MSGBROKER_INST1`, `MSGBROKER_PROD1`).

2. **Implement Dependency Validation Script** - Create PreSetupChecks.ps1 following Data Service validation pattern to verify SQL Server connectivity, confirm Data Service running status (Message Broker depends on Data Service), check Message Broker database existence or creation capability, validate explicit port availability from pipeline parameters, verify service account credentials from Key Vault (`[Redacted]`, `[Redacted]`), and confirm SQL login "mb" exists with correct credentials (critical manual prerequisite from existing batch process). Add validation warning if "mb" login missing with instructions for manual creation.

3. **Configure Message Broker Registry Settings** - Add `MessageBroker` section to AppRegistry.json using hierarchical configuration (global → environment → instance overrides) matching existing registry file structure. Base settings from MessageBroker_main.txt: `Unicode=1` (required), and MessageBroker.txt: `db_options="sql_login_timeout=30"`, `RestartRetries="1"`. Registry path: `HKEY_LOCAL_MACHINE\SOFTWARE\ApplicationX\MessageBroker\MessageBroker{Instance}`. Implement `Create-MessageBrokerRegistryFile` function in ExecuteInstallation.ps1 to load settings via `Get-AppRegistryConfig.ps1` and generate `.reg` file using `New-RegistryFileFromHashtable`, matching Data Service registry file handling pattern.

4. **Update Pipeline Template and Configuration** - Replace placeholder install.yml with functional template downloading Message Broker artifact via shared/download-component-artifact.yml, invoking PreSetupChecks.ps1 and ExecuteInstallation.ps1 with error handling. Add explicit Message Broker port configuration to AppConfig.json following API Service pattern with instance-level port definitions (e.g., INST1: `msgbrokerPort: 5211`, `msgbrokerMonitorPort: 6211`, INST2: `msgbrokerPort: 5212`, `msgbrokerMonitorPort: 6212`). Update shared/download-component-artifact.yml if Message Broker not already in supported components list.

5. **Integrate Message Broker into Server Deployment Workflows** - Modify server/install-application-server.yml to invoke Message Broker installation after Deployment Service but before Integration Service, respecting dependency order (Data Service → Deployment Service → **Message Broker** → Integration Service). Ensure Message Broker deployment targets application server agent (not batch/FTP), matching Data Service server targeting pattern. Pass explicit `Port` and `MonitorPort` values from instance configuration, plus `[Redacted]` and `[Redacted]` from environment secrets loaded via Key Vault. Map pipeline parameters to ExecuteInstallation.ps1 parameters following Data Service template invocation pattern.

### Further Considerations

1. **Manual "mb" SQL Login Automation** - Existing batch script requires manual creation of SQL login "mb" before installation with warning prompt. Should we automate this via PowerShell (`Invoke-Sqlcmd`) in PreSetupChecks.ps1 or preserve manual step requiring DBA approval? If automated, verify login password matches [Redacted] from Key Vault and handle idempotent re-creation.

2. **DATADEVICESIZE/LOGDEVICESIZE Tunability** - Existing script hardcodes 1000MB data / 500MB log sizes. Should these be configurable per environment/instance in AppConfig.json, or remain static? Production environments may need larger allocations based on message volume history.

3. **TASK Parameter Exclusion** - Existing script uses `/TASK!="Instance scheduling"` to exclude a specific task component. Verify this exclusion is still required in current MessageBrokerSetup versions or if installer behavior changed. Check VendorX documentation for complete task component list and confirm exclusion syntax.

---

### User

Start implementation

---

### GitHub Copilot

I'll start implementing the Message Broker installation automation following the established Data Service/API Service patterns. 

Created 6 implementation tasks

Gathered context from Data Service implementation

Now let me create the Message Broker installation script: 

Created [ExecuteInstallation.ps1](deployment/components/MessageBroker/scripts/ExecuteInstallation.ps1)

Starting: *Create Message Broker PreSetupChecks.ps1 validation script* (2/6)

Now let me create the PreSetupChecks.ps1 script: 

Gathered validation patterns from API Service

Created [PreSetupChecks.ps1](deployment/components/MessageBroker/scripts/PreSetupChecks.ps1)

Starting: *Add Message Broker registry configuration to AppRegistry.json* (3/6)

Now let me add Message Broker configuration to AppRegistry.json: 

Made changes to [AppRegistry.json](config/AppRegistry.json)

Starting: *Update Message Broker pipeline template install.yml* (4/6)

Now let me update the Message Broker install.yml template: 

Made changes to [install.yml](deployment/components/MessageBroker/install.yml)

Starting: *Add Message Broker port configuration to AppConfig.json* (5/6)

Now let me add Message Broker port configuration to AppConfig.json: 

Made changes to add port configuration for all instances:
- Dev instances (INST1-INST5): ports 5211-5215 / 6211-6215
- Staging instances (STAG1-STAG8, STAG42): ports based on instance number
- Prod instances (PROD1): ports 5201 / 6201

Made changes to [AppConfig.json](config/AppConfig.json)

Starting: *Integrate Message Broker into install-application-server.yml* (6/6)

Now let me integrate Message Broker into the application server installation workflow: 

Made changes to [install-application-server.yml](deployment/server/install-application-server.yml)

Completed: *Integrate Message Broker into install-application-server.yml* (6/6)

Perfect! All implementation tasks are complete. Let me provide a summary of what was implemented:

## Implementation Complete ✅

The Message Broker installation automation has been successfully implemented following the established Data Service/API Service patterns. Here's what was created:

### Files Created/Modified:

1. **ExecuteInstallation.ps1** (New)
   - Main installation script with mode detection (install/upgrade/reconfigure/skip)
   - Registry file generation from AppRegistry.json
   - Service management (stop/start)
   - MessageBrokerSetup.exe execution with all required parameters including `/ENCRYPT`, `/DATADEVICESIZE=1000`, `/LOGDEVICESIZE=500`, `/DBUNICODE=1`
   - Instance-specific service naming (e.g., MSGBROKER_INST1, MSGBROKER_PROD1)

2. **PreSetupChecks.ps1** (New)
   - Validates Data Service is running (critical dependency)
   - Checks SQL Server connectivity
   - Verifies SQL login 'mb' exists (with warning if missing)
   - Tests Message Broker database existence
   - Validates port availability (connection and monitor ports)

3. **AppRegistry.json** (Modified)
   - Added global Message Broker section with `Unicode=1`, `db_options="sql_login_timeout=30"`, `RestartRetries="1"`
   - Follows hierarchical configuration pattern (global → environment → instance)

4. **install.yml** (Modified)
   - Replaced placeholder with functional pipeline template
   - Downloads Message Broker setup artifact
   - Invokes PreSetupChecks and ExecuteInstallation scripts
   - Passes all required parameters including ports, passwords, and database names

5. **AppConfig.json** (Modified)
   - Added explicit `msgbrokerPort`, `msgbrokerMonitorPort`, and `msgbrokerDatabase` configuration for all instances
   - Port pattern: 5XXX for connection (e.g., 5211), 6XXX for monitor (e.g., 6211) where X = instance last digit
   - Configured for: INST1-INST5, STAG1-STAG8, STAG42, PROD1

6. **install-application-server.yml** (Modified)
   - Added Message Broker installation template invocation with parameters
   - Positioned between Deployment Service and Integration Service (respecting dependency order: Data Service → Deployment Service → **Message Broker** → Integration Service)

### Key Implementation Details:

- **Port Allocation**: Uses 5XXX for Message Broker connection and 6XXX for monitor port (matching last digit of instance)
- **Database Naming**: `MSGBROKER_{Instance}` (e.g., MSGBROKER_INST1, MSGBROKER_PROD1, MSGBROKER_STAG1)
- **Service Naming**: Same as database (e.g., MSGBROKER_INST1 service for INST1 instance)
- **Credential Handling**: [REDACTED]
- **Manual Prerequisite**: SQL login must be created manually before installation (script warns if missing)
- **Registry Configuration**: Loaded from AppRegistry.json and passed via `/REGISTRYINPUTFILE`

The implementation is ready for testing on a development environment.
