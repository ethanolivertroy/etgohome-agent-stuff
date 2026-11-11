---
name: emasser-get
description: Execute eMASSer CLI GET commands to retrieve data from the eMASS API. Use when querying systems, controls, POA&Ms, milestones, artifacts, test results, CAC/PAC status, hardware/software baseline, CMMC assessments, workflow definitions, or dashboards.
user-invocable: true
argument-hint: "[systems|controls|poams|milestones|artifacts|test_results|cac|pac|hardware|software|cmmc|workflow|dashboards]"
---

# eMASSer GET Commands

You are an expert in using the eMASSer CLI GET endpoints to retrieve data from the eMASS API.

## Prerequisites
- eMASSer CLI installed and `.env` file configured (see emasser-setup skill)
- Valid eMASS API credentials and access

## Common Flags
- Boolean TRUE: `--flagName` (e.g., `--includePackage`)
- Boolean FALSE: `--no-flagName` (e.g., `--no-includePackage`)
- Dates: Unix time format (e.g., `1499644800`)

---

## Test Connection
```bash
emasser get test connection
```

---

## Systems

### Get system ID by name or owner
```bash
emasser get system id --system_name "My System Name" --system_owner "owner@example.com"
```

### Get system details by ID
```bash
emasser get system byId -s <systemId>
# Optional: --includePackage, --policy [diacap|rmf|reporting]
emasser get system byId -s 100 --includePackage --policy rmf
```

### Get all systems
```bash
emasser get systems all
# Optional parameters:
#   -c, --coamsId <id>
#   -t, --ditprId <id>
#   -r, --registrationType [assessAndAuthorize|assessOnly|guest|regular|functional|cloudServiceProvider|commonControlProvider]
#   -I, --includeDecommissioned
#   -M, --includeDitprMetrics
#   -P, --includePackage
#   -p, --policy [diacap|rmf|reporting]
#   -S, --reportsForScorecard
emasser get systems all --includeDecommissioned --policy rmf
```

---

## System Roles
```bash
# Get all roles
emasser get roles all

# Get roles by category
emasser get roles byCategory -c <roleCategory> -r <role>
# roleCategory: PAC, CAC, Other
# role: AO, Auditor, "Artifact Manager", "C&A Team", IAO, ISSO, "PM/IAM", SCA, "User Rep (View Only)", "Validator (IV&V)"
# Optional: -p, --policy [diacap|rmf|reporting]
emasser get roles byCategory -c PAC -r ISSO --policy rmf
```

---

## Controls
```bash
emasser get controls forSystem -s <systemId>
# Optional: -a, --acronyms "AC-1, AC-2"
emasser get controls forSystem -s 100 -a "AC-1, AC-2"
```

---

## Test Results
```bash
emasser get test_results forSystem -s <systemId>
# Optional:
#   -a, --controlAcronyms "AC-1, AC-2"
#   -p, --assessmentProcedures "AC-1.1,AC-1.2"
#   -c, --ccis <ccis-string>
#   -L, --latestOnly
emasser get test_results forSystem -s 100 -a "AC-1" --latestOnly
```

---

## POA&Ms (Plan of Action & Milestones)
```bash
# Get all POA&Ms for a system
emasser get poams forSystem -s <systemId>
# Optional:
#   -d, --scheduledCompletionDateStart <unix-timestamp>
#   -e, --scheduledCompletionDateEnd <unix-timestamp>
#   -a, --controlAcronyms "AC-1, AC-2"
#   -p, --assessmentProcedures "AC-1.1,AC-1.2"
#   -c, --ccis <ccis-string>
#   -Y, --systemOnly
emasser get poams forSystem -s 100 -d 1499644800 -e 1499990400

# Get a specific POA&M
emasser get poams byPoamId -s <systemId> -p <poamId>
emasser get poams byPoamId -s 100 -p 45
```

---

## Milestones
```bash
# Get milestones for a POA&M
emasser get milestones byPoamId -s <systemId> -p <poamId>
# Optional: -d --scheduledCompletionDateStart, -e --scheduledCompletionDateEnd
emasser get milestones byPoamId -s 100 -p 45

# Get a specific milestone
emasser get milestones byMilestoneId -s <systemId> -p <poamId> -m <milestoneId>
emasser get milestones byMilestoneId -s 100 -p 45 -m 10
```

---

## Artifacts
```bash
# Get all artifacts for a system
emasser get artifacts forSystem -s <systemId>
# Optional:
#   -f, --filename <name>
#   -a, --controlAcronyms "AC-1, AC-2"
#   -p, --assessmentProcedures "AC-1.1,AC-1.2"
#   -c, --ccis <ccis-string>
#   -Y, --systemOnly
emasser get artifacts forSystem -s 100 -a "AC-1"

# Export/read artifact file content
emasser get artifacts export -s <systemId> -f <filename>
# Optional: -C, --compress  |  -o, --printToStdout
# Prefer downloading binary files; use --printToStdout only for safe, small text artifacts.
export EMASSER_DOWNLOAD_DIR="$PWD/eMASSerDownloads"
emasser get artifacts export -s 100 -f "security-plan.pdf"
emasser get artifacts export -s 100 -f "evidence.txt" --printToStdout
```

---

## CAC (Control Approval Chain)
```bash
emasser get cac controls -s <systemId>
# Optional: -a, --controlAcronyms "AC-1, AC-2"
emasser get cac controls -s 100 -a "AC-1, AC-2"
```

---

## PAC (Package Approval Chain)
```bash
emasser get pac package -s <systemId>
emasser get pac package -s 100
```

---

## Hardware Baseline
```bash
emasser get hardware assets -s <systemId>
# Optional: -i, --pageIndex <index>  |  --pageSize <size>
# Prefer long --pageSize because -s is also used for --systemId in help output.
emasser get hardware assets -s 100 --pageSize 100
```

---

## Software Baseline
```bash
emasser get software assets -s <systemId>
# Optional: -i, --pageIndex <index>  |  --pageSize <size>
# Prefer long --pageSize because -s is also used for --systemId in help output.
emasser get software assets -s 100 --pageSize 100
```

---

## CMMC Assessment
```bash
emasser get cmmc assessments -d <sinceDate>
# -d: Unix timestamp (e.g., 1499644800)
emasser get cmmc assessments -d 1499644800
```

---

## Workflow Definitions
```bash
emasser get workflow_definitions forSite
# Optional:
#   -I, --includeInactive
#   -r, --registrationType [assessAndAuthorize|assessOnly|guest|regular|functional|cloudServiceProvider|commonControlProvider]
emasser get workflow_definitions forSite --includeInactive
```

---

## Workflow Instances
```bash
# Get all workflow instances
emasser get workflow_instances all
# Optional:
#   -C, --includeComments
#   -D, --includeDecommissionSystems
#   -p, --pageIndex <index>
#   -d, --sinceDate <unix-timestamp>
#   -s, --status [active|inactive|all]
emasser get workflow_instances all -s active

# Get by workflow instance ID
emasser get workflow_instances byInstanceId --workflowInstanceId <id>
emasser get workflow_instances byInstanceId --workflowInstanceId 99
```

---

## Dashboards
```bash
emasser get dashboards help
# Dashboard commands require -o, --orgId <orgId>.
emasser get dashboards control_compliance_summary -o <orgId>
emasser get dashboards poam_summary -o <orgId>
emasser get dashboards hardware_summary -o <orgId>
emasser get dashboards software_summary -o <orgId>
emasser get dashboards workflows_history_summary -o <orgId>

# Optional dashboard flags:
#   -I, --excludeinherited
#   -i, --pageIndex <index>
#   -s, --pageSize <size>
```
