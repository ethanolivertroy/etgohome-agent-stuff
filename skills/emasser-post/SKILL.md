---
name: emasser-post
description: Execute eMASSer CLI POST commands to add data to the eMASS API. Use when registering certificates, uploading test results, creating POA&Ms, adding milestones, uploading artifacts, submitting CAC/PAC workflows, adding hardware/software assets, or uploading scan results (device scans, cloud resources, container scans, static code scans).
user-invocable: true
argument-hint: "[test_results|poams|milestones|artifacts|cac|pac|hardware|software|device_scans|cloud_resource|container|scan_findings]"
---

# eMASSer POST Commands

You are an expert in using the eMASSer CLI POST endpoints to add data to the eMASS API.

## Prerequisites
- eMASSer CLI installed and `.env` file configured (see emasser-setup skill)
- Valid eMASS API credentials and access

## Common Flags
- Boolean TRUE: `--flagName` (e.g., `--isTemplate`)
- Boolean FALSE: `--no-flagName` (e.g., `--no-isTemplate`)
- Dates: Unix time format (e.g., `1499990400`)

---

## Register Certificate
```bash
emasser post register cert
```

---

## Test Results
Add a test result for a security control assessment procedure.

```bash
emasser post test_results add \
  -s <systemId> \
  --assessmentProcedure <value> \
  --testedBy "LastName, FirstName" \
  --testDate <unix-timestamp> \
  --description "<test result description>" \
  --complianceStatus <Compliant|Non-Compliant|Not Applicable>
```

**Business Rules:**
- `testDate` cannot be in the future
- `description` cannot exceed 4,000 characters
- Security Control cannot be "Inherited"

```bash
# Example
emasser post test_results add -s 100 \
  --assessmentProcedure "AC-1.1" \
  --testedBy "Smith, John" \
  --testDate 1499990400 \
  --description "Verified access control policy is documented and reviewed." \
  --complianceStatus Compliant
```

---

## POA&Ms (Plan of Action & Milestones)
Create a new POA&M item.

```bash
emasser post poams add \
  -s <systemId> \
  --status <Ongoing|Risk Accepted|Completed|Not Applicable> \
  --vulnerabilityDescription "<description>" \
  --sourceIdentifyingVulnerability "<source>" \
  --pocOrganization "<org>" \
  --resources "<resources>"
```

**Status-Dependent Required Fields:**
| Status | Additional Required Fields |
|--------|---------------------------|
| Risk Accepted | `--comments` |
| Ongoing | `--scheduledCompletionDate`, `--milestones` (at least 1) |
| Completed | `--scheduledCompletionDate`, `--comments`, `--completionDate`, `--milestones` (at least 1) |

**Milestone Format:**
```bash
--milestone description:[value] scheduledCompletionDate:[value]
```

```bash
# Example: Ongoing POA&M
emasser post poams add -s 100 \
  --status Ongoing \
  --vulnerabilityDescription "Missing patch for CVE-2024-1234" \
  --sourceIdentifyingVulnerability "ACAS scan" \
  --pocOrganization "IT Security" \
  --resources "Security team" \
  --scheduledCompletionDate 1735689600 \
  --milestone description:"Apply patch" scheduledCompletionDate:1735689600
```

---

## Milestones
Add a milestone to an existing POA&M.

```bash
emasser post milestones add \
  -s <systemId> \
  -p <poamId> \
  -d "<milestone description>" \
  -c <scheduledCompletionDate-unix-timestamp>
```

```bash
# Example
emasser post milestones add -s 100 -p 45 \
  -d "Complete patch deployment" \
  -c 1735689600
```

---

## Artifacts
Upload artifact files to a system.

```bash
emasser post artifacts upload \
  -s <systemId> \
  --isTemplate or --no-isTemplate \
  -t <type> \
  -c <category> \
  -f <file1> [file2 ...]
```

**Type values:** `Procedure`, `Diagram`, `Policy`, `Labor`, `Document`, `Image`, `Other`, `Scan Result`, `Auditor Report`
**Category values:** `Implementation Guidance`, `Evidence`

```bash
# Example: Upload a policy document
emasser post artifacts upload -s 100 \
  --no-isTemplate \
  -t Policy \
  -c "Implementation Guidance" \
  -f /path/to/access-control-policy.pdf

# Example: Bulk upload (expects .zip)
emasser post artifacts upload -s 100 \
  --no-isTemplate \
  -t Document \
  -c Evidence \
  --isBulk \
  -f /path/to/artifacts.zip
```

**Constraints:** File size ≤ 30MB, filename ≤ 1,000 chars, last review date cannot be in the future.

**Required agent workflow before upload:**
1. Read/inspect the local file path: `test -f`, `file`, and `stat`. Do not upload `.env`, private keys, client certificates, API keys, or other secrets.
2. Use an absolute path when possible.
3. Choose the right type/category: use `Evidence` for assessment proof, `Implementation Guidance` for policy/procedure guidance, and `Scan Result` for scan artifacts.
4. Run `emasser post artifacts help upload` if any flag is uncertain.
5. Verify after upload: `emasser get artifacts forSystem -s <systemId> -f "$(basename /path/to/file)"`.

```bash
# Safe inspect → upload → verify flow
test -f /path/to/evidence.pdf && file /path/to/evidence.pdf && stat /path/to/evidence.pdf
emasser post artifacts upload -s 100 \
  --no-isTemplate \
  -t Document \
  -c Evidence \
  -f /path/to/evidence.pdf
emasser get artifacts forSystem -s 100 -f "evidence.pdf"
```

---

## CAC (Control Approval Chain)
Submit a control to the Control Approval Chain.

```bash
emasser post cac add \
  -s <systemId> \
  -a <controlAcronym> \
  [-c "<comments>"]
```

**Note:** Comments are required at the second role of the CAC (max 10,000 chars).

```bash
# Example
emasser post cac add -s 100 -a "AC-1" -c "Control verified and approved."
```

---

## PAC (Package Approval Chain)
Initiate a Package Approval Chain workflow.

```bash
emasser post pac add \
  -s <systemId> \
  -f <workflow> \
  -n "<package name>" \
  -c "<comments>"
```

**Workflow values:** `"Assess and Authorize"`, `"Assess Only"`, `"Security Plan Approval"`

```bash
# Example
emasser post pac add -s 100 \
  -f "Assess and Authorize" \
  -n "FY2024 ATO Package" \
  -c "Initiating A&A workflow for annual review."
```

---

## Hardware Assets
Add hardware assets to a system.

```bash
emasser post hardware add -s <systemId> -a "<assetName>"
# Example
emasser post hardware add -s 100 -a "Dell PowerEdge R740 - Web Server 01"
```

---

## Software Assets
Add software assets to a system.

```bash
emasser post software add \
  -s <systemId> \
  -V "<softwareVendor>" \
  -N "<softwareName>" \
  -v "<version>"
```

```bash
# Example
emasser post software add -s 100 \
  -V "Apache Software Foundation" \
  -N "Apache HTTP Server" \
  -v "2.4.58"
```

---

## Device Scan Results
Upload device scan results (ACAS, STIG Viewer, etc.).

```bash
emasser post device_scans add \
  -s <systemId> \
  -f <file1> [file2 ...] \
  -t <scanType>
```

**Scan types:** `acasAsrArf`, `acasNessus`, `disaStigViewerCklCklb`, `disaStigViewerCmrs`, `policyAuditor`, `scapComplianceChecker`

```bash
# Example: Upload STIG Viewer CKL file
emasser post device_scans add -s 100 \
  -f /path/to/system-stig-results.ckl \
  -t disaStigViewerCklCklb

# Example: Upload ACAS Nessus scan
emasser post device_scans add -s 100 \
  -f /path/to/scan.nessus \
  -t acasNessus
```

---

## Cloud Resource Results
Add cloud resource scan results.

```bash
emasser post cloud_resource add \
  -s <systemId> \
  --provider "<provider>" \
  --resourceId "<resourceId>" \
  --resourceName "<name>" \
  --resourceType "<type>" \
  --cspPolicyDefinitionId "<policyId>" \
  --isCompliant or --no-isCompliant \
  --policyDefinitionTitle "<title>"
```

```bash
# Example
emasser post cloud_resource add -s 100 \
  --provider "AWS" \
  --resourceId "arn:aws:ec2:us-east-1:123456789:instance/i-0abc123" \
  --resourceName "prod-web-server-01" \
  --resourceType "EC2 Instance" \
  --cspPolicyDefinitionId "aws-cis-benchmark-v1.4" \
  --isCompliant \
  --policyDefinitionTitle "CIS AWS Benchmark v1.4"
```

---

## Container Scan Results
Add container scan results.

```bash
emasser post container add \
  -s <systemId> \
  --containerId "<id>" \
  --containerName "<name>" \
  --time <unix-timestamp> \
  --benchmark "<benchmark>" \
  --lastSeen <unix-timestamp> \
  --ruleId "<ruleId>" \
  --status "<status>"
```

```bash
# Example
emasser post container add -s 100 \
  --containerId "sha256:abc123" \
  --containerName "nginx-frontend" \
  --time 1499990400 \
  --benchmark "DISA STIG for NGINX" \
  --lastSeen 1499990400 \
  --ruleId "V-55969" \
  --status "pass"
```

---

## Static Code Scan Findings
Add static code scan findings.

```bash
# Add findings
emasser post scan_findings add \
  -s <systemId> \
  --applicationName "<name>" \
  --version "<version>" \
  --codeCheckName "<checkName>" \
  --scanDate <unix-timestamp> \
  --cweId "<cweId>" \
  --count <count>
```

```bash
# Example
emasser post scan_findings add -s 100 \
  --applicationName "MyWebApp" \
  --version "2.1.0" \
  --codeCheckName "SQL Injection" \
  --scanDate 1499990400 \
  --cweId "CWE-89" \
  --rawSeverity High \
  --count 3

# Clear all findings for an application version
emasser post scan_findings clear \
  -s 100 \
  --applicationName "MyWebApp" \
  --version "2.1.0" \
  --clearFindings
```
