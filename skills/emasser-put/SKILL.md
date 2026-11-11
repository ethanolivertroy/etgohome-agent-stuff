---
name: emasser-put
description: Execute eMASSer CLI PUT commands to update existing data in the eMASS API. Use when updating security controls, modifying POA&Ms, updating milestones, updating artifact metadata, or updating hardware/software baseline assets.
user-invocable: true
argument-hint: "[controls|poams|milestones|artifacts|hardware|software]"
---

# eMASSer PUT Commands

You are an expert in using the eMASSer CLI PUT endpoints to update existing data in the eMASS API.

## Prerequisites
- eMASSer CLI installed and `.env` file configured (see emasser-setup skill)
- Valid eMASS API credentials and access
- Existing records to update (system ID, POA&M ID, etc.)

## Common Flags
- Boolean TRUE: `--flagName`
- Boolean FALSE: `--no-flagName`
- Dates: Unix time format (e.g., `1499990400`)

---

## Controls
Update a security control's implementation plan.

```bash
emasser put controls update \
  --systemId <systemId> \
  --acronym "<AC-1>" \
  --responsibleEntities "<description>" \
  --controlDesignation <Common|System-Specific|Hybrid> \
  --estimatedCompletionDate <unix-timestamp> \
  --implementationNarrative "<implementation narrative>"
```

**Implementation Status → Required Fields:**
| implementationStatus | Additional Required Fields |
|----------------------|---------------------------|
| Planned or Implemented | `--controlDesignation`, `--estimatedCompletionDate`, `--responsibleEntities`, `--slcmCriticality`, `--slcmFrequency`, `--slcmMethod`, `--slcmTracking`, `--slcmComments` |
| Not Applicable | `--naJustification`, `--controlDesignation`, `--responsibleEntities` |
| Manually Inherited | `--controlDesignation`, `--estimatedCompletionDate`, `--responsibleEntities`, `--slcmCriticality`, `--slcmFrequency`, `--slcmMethod`, `--slcmTracking`, `--slcmComments` |

```bash
# Example: Mark a control as Implemented
emasser put controls update \
  --systemId 100 \
  --acronym "AC-1" \
  --responsibleEntities "IT Security Team" \
  --controlDesignation System-Specific \
  --estimatedCompletionDate 1735689600 \
  --implementationNarrative "Access control policy reviewed and updated." \
  --implementationStatus Implemented \
  --slcmCriticality "High - critical system" \
  --slcmFrequency Quarterly \
  --slcmMethod Automated \
  --slcmTracking "Tracked via SIEM" \
  --slcmComments "Automated monitoring in place"
```

**Character Limits (2,000):** `naJustification`, `responsibleEntities`, `implementationNarrative`, `slcmCriticality`, `slcmReporting`, `slcmTracking`, `vulnerabilitySummary`, `recommendations`
**Character Limits (4,000):** `slcmComments`

---

## POA&Ms
Update an existing POA&M item.

```bash
emasser put poams update \
  -s <systemId> \
  -p <poamId> \
  --status <Ongoing|Risk Accepted|Completed> \
  --vulnerabilityDescription "<description>" \
  --sourceIdentifyingVulnerability "<source>" \
  --pocOrganization "<org>" \
  --resources "<resources>"
```

**Status-Dependent Required Fields:**
| Status | Additional Required Fields |
|--------|---------------------------|
| Risk Accepted | `--comments`, `--resources` |
| Ongoing | `--scheduledCompletionDate`, `--resources`, `--milestones` (at least 1) |
| Completed | `--scheduledCompletionDate`, `--comments`, `--resources`, `--completionDate`, `--milestones` (at least 1) |

```bash
# Example: Update POA&M to Completed status
emasser put poams update \
  -s 100 \
  -p 45 \
  --status Completed \
  --vulnerabilityDescription "Missing patch for CVE-2024-1234" \
  --sourceIdentifyingVulnerability "ACAS scan" \
  --pocOrganization "IT Security" \
  --resources "Security team" \
  --scheduledCompletionDate 1735689600 \
  --completionDate 1735603200 \
  --comments "Patch successfully applied and verified." \
  --milestone description:"Patch applied" scheduledCompletionDate:1735603200
```

---

## Milestones
Update an existing milestone.

```bash
emasser put milestones update \
  -s <systemId> \
  -p <poamId> \
  -m <milestoneId> \
  -d "<new description>" \
  -c <new-scheduledCompletionDate-unix-timestamp>
```

```bash
# Example
emasser put milestones update \
  -s 100 \
  -p 45 \
  -m 10 \
  -d "Patch deployment completed ahead of schedule" \
  -c 1735603200
```

---

## Artifacts
Update artifact metadata for an existing artifact. In eMASSer PUT, `-f/--filename` is the exact artifact filename already in eMASS, not a local file path. Use `post artifacts upload` to upload a new binary file, then use PUT when you need to adjust metadata such as type/category/name/control mappings.

Important: artifact PUT replaces existing metadata with the fields included in the request. Fields omitted from the PUT can become null, and one-to-many mappings such as controls are replaced rather than appended.

```bash
emasser put artifacts update \
  -s <systemId> \
  -f <filename-in-eMASS> \
  --isTemplate or --no-isTemplate \
  -t <type> \
  -c <category>
```

**Type values:** `Procedure`, `Diagram`, `Policy`, `Labor`, `Document`, `Image`, `Other`, `Scan Result`, `Auditor Report`
**Category values:** `Implementation Guidance`, `Evidence`

```bash
# Example: Update metadata for an existing policy artifact
emasser put artifacts update -s 100 \
  -f "access-control-policy.pdf" \
  --no-isTemplate \
  -t Policy \
  -c "Implementation Guidance"
```

---

## Hardware Assets
Update an existing hardware asset.

```bash
emasser put hardware update \
  -s <systemId> \
  -h <hardwareId> \
  -a "<assetName>"
```

Common optional fields include `--componentType`, `--nickname`, `--assetIpAddress`, `--publicFacing`, `--virtualAsset`, `--manufacturer`, `--modelNumber`, `--serialNumber`, `--OsIosFwVersion`, `--memorySizeType`, `--location`, `--approvalStatus`, and `--criticalAsset`. Run `emasser put hardware help update` before changing optional fields.

---

## Software Assets
Update an existing software asset.

```bash
emasser put software update \
  -s <systemId> \
  -S <softwareId> \
  -V "<softwareVendor>" \
  -N "<softwareName>" \
  -v "<version>"
```

Common optional fields include `--approvalDate`, `--softwareType`, `--parentSystem`, `--network`, `--hostingEnvironment`, license/cost fields, lifecycle dates, `--approvalStatus`, `--criticalAsset`, `--location`, and `--purpose`. Run `emasser put software help update` before changing optional fields.

---

## Getting Help

```bash
emasser put help
emasser put controls help update
emasser put poams help update
emasser put milestones help update
emasser put artifacts help update
```
