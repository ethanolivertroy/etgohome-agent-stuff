---
name: emasser-artifacts
description: Read, inspect, export, and upload eMASS artifacts with the eMASSer CLI. Use when an agent needs to find local evidence files, validate them, upload artifacts, export existing artifacts, or verify uploaded evidence in eMASS.
user-invocable: true
argument-hint: "[read|inspect|upload|export|verify]"
---

# eMASSer Artifact Read/Upload Workflow

You are an expert at using the eMASSer CLI for artifact evidence workflows. The default tool is always `emasser`; do not bypass it with raw API calls unless the user explicitly asks for direct API work or the CLI cannot perform the action.

## Core Rule

Agents must be able to read local evidence, upload it to eMASS, export it back out, and verify what happened.

Always follow this loop:

1. Identify the target eMASS system ID.
2. Inspect the local file(s) before upload.
3. Confirm eMASSer is installed and configured.
4. Run the correct `emasser` command.
5. Verify the artifact exists in eMASS after upload.
6. Record the command, system ID, artifact filename, and verification result.

## Prerequisites

- eMASSer CLI installed and configured (`emasser-setup`)
- Valid `.env` containing eMASS credentials and certificate paths
- Target `systemId`
- Local artifact path(s), or the exact filename to export from eMASS

Quick checks:

```bash
command -v emasser
emasser get test connection
emasser get system byId -s <systemId>
```

## Read Local Evidence Before Upload

Before uploading, inspect the file so the agent knows what it is sending.

```bash
# Confirm the path exists and is a regular file
test -f /path/to/evidence.pdf

# Inspect metadata
file /path/to/evidence.pdf
stat /path/to/evidence.pdf

# Check size in MiB; eMASS artifacts are commonly limited to 30 MB
python3 - <<'PY'
from pathlib import Path
p = Path('/path/to/evidence.pdf')
print(f"name={p.name}")
print(f"size_bytes={p.stat().st_size}")
print(f"size_mib={p.stat().st_size / 1024 / 1024:.2f}")
PY
```

For text-like evidence, read enough to verify content without dumping secrets:

```bash
python3 - <<'PY'
from pathlib import Path
p = Path('/path/to/evidence.txt')
print(p.read_text(errors='replace')[:4000])
PY
```

For binary files such as PDF, DOCX, XLSX, PNG, ZIP, CKL, or Nessus files, do not print raw bytes. Use metadata, filename, size, and content-specific tools when available.

## Upload Artifacts

Use `emasser post artifacts upload` for evidence files attached to a system.

```bash
emasser post artifacts upload \
  -s <systemId> \
  --no-isTemplate \
  -t <type> \
  -c <category> \
  -f /absolute/path/to/file
```

Common values:

- Type: `Procedure`, `Diagram`, `Policy`, `Labor`, `Document`, `Image`, `Other`, `Scan Result`, `Auditor Report`
- Category: `Implementation Guidance`, `Evidence`

Examples:

```bash
# Policy implementation guidance
emasser post artifacts upload -s 100 \
  --no-isTemplate \
  -t Policy \
  -c "Implementation Guidance" \
  -f /work/evidence/access-control-policy.pdf

# Assessment evidence
emasser post artifacts upload -s 100 \
  --no-isTemplate \
  -t Document \
  -c Evidence \
  -f /work/evidence/ac-2-user-review.xlsx

# Scan result artifact
emasser post artifacts upload -s 100 \
  --no-isTemplate \
  -t "Scan Result" \
  -c Evidence \
  -f /work/scans/ac-2-results.pdf
```

For multiple files, pass them after `-f` if supported by the installed CLI version:

```bash
emasser post artifacts upload -s 100 \
  --no-isTemplate \
  -t Document \
  -c Evidence \
  -f /work/evidence/file1.pdf /work/evidence/file2.xlsx
```

For bulk upload, package evidence into a zip if your eMASS workflow expects bulk artifact upload:

```bash
zip -r artifacts.zip evidence/
emasser post artifacts upload -s 100 \
  --no-isTemplate \
  -t Document \
  -c Evidence \
  --isBulk \
  -f /work/artifacts.zip
```

## Upload Device and Scan Evidence

Use scan-specific endpoints when the file is a structured scan result, not a general artifact.

```bash
# STIG Viewer CKL/CKLB
emasser post device_scans add -s <systemId> \
  -f /path/to/results.ckl \
  -t disaStigViewerCklCklb

# ACAS Nessus
emasser post device_scans add -s <systemId> \
  -f /path/to/scan.nessus \
  -t acasNessus

# SCAP Compliance Checker
emasser post device_scans add -s <systemId> \
  -f /path/to/scap-results.xml \
  -t scapComplianceChecker
```

If the user says "upload this scan", prefer the scan endpoint. If they say "attach this as evidence", prefer artifacts upload.

## Read Existing Artifacts from eMASS

List artifacts attached to a system:

```bash
emasser get artifacts forSystem -s <systemId>
```

Filter by filename or control when needed:

```bash
emasser get artifacts forSystem -s <systemId> -f "access-control-policy.pdf"
emasser get artifacts forSystem -s <systemId> -a "AC-1, AC-2"
```

Export an artifact from eMASS:

```bash
emasser get artifacts export -s <systemId> -f "access-control-policy.pdf"
```

Print to stdout only when the content is safe and reasonably small:

```bash
emasser get artifacts export -s <systemId> -f "evidence.txt" --printToStdout
```

Use `EMASSER_DOWNLOAD_DIR` to control where downloaded artifacts are written:

```bash
export EMASSER_DOWNLOAD_DIR="$PWD/eMASSerDownloads"
emasser get artifacts export -s <systemId> -f "security-plan.pdf"
```

## Verify Uploads

After every upload, query eMASS for the artifact by filename.

```bash
emasser get artifacts forSystem -s <systemId> -f "$(basename /path/to/file.pdf)"
```

Verify:

- The filename appears in the response.
- The system ID matches the target system.
- The artifact type/category match the intended use.
- Control acronyms or assessment procedures are associated when the workflow requires them.

## Update Metadata or Delete Artifacts

Update metadata for an existing artifact. In eMASSer PUT, `-f/--filename` is the exact filename already in eMASS, not a replacement file path. To upload a new file, use `post artifacts upload`.

```bash
emasser put artifacts update \
  -s <systemId> \
  -f "existing-filename.pdf" \
  --no-isTemplate \
  -t Document \
  -c Evidence
```

Delete only with explicit user approval and exact filename:

```bash
emasser delete artifacts remove -s <systemId> -f "filename.pdf"
```

## Safety Rules

- Never upload secrets, private keys, client certificates, `.env` files, or API keys as artifacts.
- Never print eMASS credentials or certificate contents in logs.
- Always use absolute paths for uploads when possible.
- Always inspect file size before upload; split or compress large evidence sets.
- Always verify after upload with `emasser get artifacts forSystem`.
- For destructive actions, confirm the exact system ID and filename first.

## Validate Against Official eMASS OpenAPI Docs

Use the MITRE Swagger UI renderer as the reference for endpoint semantics when checking a command or writing a test plan:

- Swagger UI: https://mitre.github.io/emass_client/docs/renderer/
- Hosted Prism mock server: https://stoplight.io/mocks/mitre/emasser/32836028
- OpenAPI YAML: https://raw.githubusercontent.com/mitre/emass_client/main/docs/eMASSRestOpenApi.yaml

For mock validation, use either normal headers or the Prism `Prefer` header:

```bash
curl -H 'api-key: 123' -H 'user-uid: 123' \
  https://stoplight.io/mocks/mitre/emasser/32836028/api/systems/100/artifacts

curl -H 'Prefer: code=200' \
  https://stoplight.io/mocks/mitre/emasser/32836028/api/systems/100/artifacts
```

The stock `emasser` CLI cannot directly target the hosted mock URL because it strips path components from `EMASSER_HOST_URL`. Use direct mock calls for OpenAPI validation and the CLI for authorized eMASS environments.

Artifact endpoint mapping:

| eMASSer CLI | OpenAPI endpoint |
|---|---|
| `emasser get artifacts forSystem` | `GET /api/systems/{systemId}/artifacts` |
| `emasser get artifacts export` | `GET /api/systems/{systemId}/artifacts-export` |
| `emasser post artifacts upload` | `POST /api/systems/{systemId}/artifacts` |
| `emasser put artifacts update` | `PUT /api/systems/{systemId}/artifacts` |
| `emasser delete artifacts remove` | `DELETE /api/systems/{systemId}/artifacts` |
| `emasser post device_scans add` | `POST /api/systems/{systemId}/device-scan-results` |

Renderer-based validation checks:

1. Confirm `systemId` is the path parameter.
2. Confirm artifact export requires `filename` as a query parameter.
3. Confirm artifact upload is `multipart/form-data` and uses the binary `filename` field.
4. Confirm artifact upload supports the `isBulk` query parameter for zip uploads.
5. Confirm device scan upload uses `scanType` and binary `filename`.
6. Confirm PUT updates artifact metadata by filename; it does not upload a replacement binary file.

## Troubleshooting

```bash
# Show artifact command help
emasser post artifacts help upload
emasser get artifacts help export
emasser put artifacts help update
emasser delete artifacts help remove

# Enable verbose eMASSer behavior when debugging
export EMASSER_DEBUGGING='true'
emasser get test connection
```

If upload fails:

1. Re-run the command-specific help and compare required flags.
2. Confirm file exists, size is under the configured limit, and path is readable.
3. Confirm `.env` is loaded from the directory where `emasser` runs.
4. Confirm the API key and client certificate are authorized for the target system.
5. Query the system and artifacts list to ensure the system ID is correct.
