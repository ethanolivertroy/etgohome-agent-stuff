---
name: emasser-delete
description: Execute eMASSer CLI DELETE commands to remove data from the eMASS API. Use when deleting POA&Ms, removing milestones, deleting artifacts, removing hardware/software assets, or clearing cloud resource and container scan results.
user-invocable: true
argument-hint: "[poams|milestones|artifacts|hardware|software|cloud_resource|container]"
---

# eMASSer DELETE Commands

You are an expert in using the eMASSer CLI DELETE endpoints to remove data from the eMASS API.

## Prerequisites
- eMASSer CLI installed and `.env` file configured (see emasser-setup skill)
- Valid eMASS API credentials and access
- IDs of the records to delete

## Warning
DELETE operations are irreversible. Always verify the IDs before running delete commands, get explicit user approval for the exact system ID and record identifier, and verify removal with a follow-up GET command.

---

## POA&Ms
Delete one or more POA&M items from a system.

```bash
emasser delete poams remove -s <systemId> -p <poamId>
```

```bash
# Example: Delete a single POA&M
emasser delete poams remove -s 100 -p 45
emasser get poams byPoamId -s 100 -p 45

# Check available options first
emasser delete poams help remove
```

---

## Milestones
Delete a milestone from a POA&M.

```bash
emasser delete milestones remove \
  -s <systemId> \
  -p <poamId> \
  -m <milestoneId>
```

```bash
# Example
emasser delete milestones remove -s 100 -p 45 -m 10
emasser get milestones byPoamId -s 100 -p 45
```

---

## Artifacts
Delete an artifact from a system.

```bash
emasser delete artifacts remove -s <systemId> -f <filename>
```

```bash
# Example
emasser delete artifacts remove -s 100 -f "access-control-policy.pdf"
emasser get artifacts forSystem -s 100 -f "access-control-policy.pdf"
```

---

## Hardware Assets
Delete a hardware asset from a system.

```bash
emasser delete hardware remove -s <systemId> -w <hardwareId1> [hardwareId2 ...]
emasser get hardware assets -s <systemId>
```

---

## Software Assets
Delete a software asset from a system.

```bash
emasser delete software remove -s <systemId> -w <softwareId1> [softwareId2 ...]
emasser get software assets -s <systemId>
```

---

## Cloud Resource Results
Delete cloud resource scan results from a system.

```bash
emasser delete cloud_resource remove -s <systemId> [options]
# Run for full options:
emasser delete cloud_resource help remove
```

---

## Container Scan Results
Delete container scan results from a system.

```bash
emasser delete container remove -s <systemId> [options]
# Run for full options:
emasser delete container help remove
```

---

## Getting Help

```bash
emasser delete help
emasser delete poams help remove
emasser delete milestones help remove
emasser delete artifacts help remove
emasser delete hardware help remove
emasser delete software help remove
emasser delete cloud_resource help remove
emasser delete container help remove
```
