# AI Cluster Comparison Instructions

You are comparing a telco-hub OpenShift cluster against the reference configuration found at `telco-hub/configuration/reference-crs`.

## Core Principles
- Ignore all metadata fields that are cluster-managed (UIDs, timestamps, resourceVersions, ownerReferences, finalizers)
- Focus on configuration differences that matter for compliance with the telco-hub reference design
- Be precise about required vs optional components
- Understand semantic equivalence (e.g., `2Gi` equals `2048Mi`)

## Component Grouping Rules

Components can have different validation rules:

### **All-or-None Components**
These components must either have ALL their CRs present or NONE of them present:
- If any CR from the component is found, then ALL CRs for that component must be present
- If no CRs are found, this is acceptable (component is optional)

### **Required Components**  
These components must have ALL their CRs present in the cluster:
- All CRs for the component must be present and correctly configured
- Missing any CR from a required component is a compliance failure

### **Optional Components**
These components may or may not be present:
- If present, all CRs must be correctly configured
- If absent, no compliance impact

*See the "Hub Components" section below for specific component details and their CR lists.*

## Component Scope and Definition

### **Component Scope Restriction**

**CRITICAL**: Only analyze the components explicitly defined in the "Hub Components" section below. Do NOT analyze any other components found in the cluster.

**Forbidden Analysis**: Do not analyze components such as:
- GitOps (unless explicitly listed in Hub Components)
- Registry/Image Registry
- Any operator or component not listed in the Hub Components section

**Why this restriction exists**: The Hub Components section defines the complete scope of telco-hub compliance. Any additional components found in the cluster are outside the telco-hub reference design scope and should not influence compliance assessment.

## Hub Components

### Local Storage Operator (LSO)
**Component Type**: Optional
**Validation Rule**: If any LSO CR is found in the cluster, then ALL must be present. If none are found, this is acceptable.

**Component CRs**:
- `lsoNS.yaml` (Namespace: openshift-local-storage)
- `lsoOperatorGroup.yaml` (OperatorGroup for LSO)
- `lsoSubscription.yaml` (Subscription to install LSO)
- `lsoLocalVolume.yaml` (LocalVolume configuration)

### Cluster Logging Operator
**Component Type**: Optional
**Validation Rule**: If any Logging CR is found in the cluster, then ALL must be present. If none are found, this is acceptable.

**Component CRs**:
- `clusterLogNS.yaml` (Namespace: openshift-logging)
- `clusterLogOperGroup.yaml` (OperatorGroup for cluster logging)
- `clusterLogSubscription.yaml` (Subscription to install cluster logging)

### OpenShift Data Foundation (ODF) Internal
**Component Type**: Optional
**Validation Rule**: If any ODF CR is found in the cluster, then ALL must be present. If none are found, this is acceptable.

**Component CRs**:
- `odfNS.yaml` (Namespace: openshift-storage)
- `odfOperatorGroup.yaml` (OperatorGroup for ODF)
- `odfSubscription.yaml` (Subscription to install ODF)
- `storageCluster.yaml` (StorageCluster configuration)

### Topology Aware Lifecycle Manager (TALM)
**Component Type**: Required
**Validation Rule**: All TALM CRs must be present and correctly configured (TALM is required for telco-hub).

**Component CRs**:
- `talmSubscription.yaml` (Subscription to install TALM)

### Advanced Cluster Management (ACM)
**Component Type**: Required
**Validation Rule**: All ACM CRs must be present and correctly configured (ACM is required for telco-hub).

**Component CRs**:
- `acmNS.yaml` (Namespace: open-cluster-management)
- `acmOperGroup.yaml` (OperatorGroup for ACM)
- `acmSubscription.yaml` (Subscription to install ACM)
- `acmMCH.yaml` (MultiClusterHub configuration)
- `acmMCE.yaml` (MultiClusterEngine configuration)
- `acmProvisioning.yaml` (Provisioning configuration)
- `acmMirrorRegistryCM.yaml` (Mirror registry ConfigMap)
- `acmAgentServiceConfig.yaml` (Agent service configuration)
- `observabilityNS.yaml` (Observability namespace)
- `observabilityMCO.yaml` (MultiClusterObservability)
- `observabilityOBC.yaml` (ObjectBucketClaim)
- `observabilitySecret.yaml` (Pull secret for observability)
- `pullSecretPolicy.yaml` (Pull secret copy policy)
- `pullSecretPlacement.yaml` (Pull secret placement)
- `pullSecretPlacementBinding.yaml` (Pull secret placement binding)
- `pullSecretMCSB.yaml` (Pull secret ManagedClusterSetBinding)
- `thanosSecretPolicy.yaml` (Thanos secret policy)
- `thanosSecretPlacement.yaml` (Thanos secret placement)
- `thanosSecretPlacementBinding.yaml` (Thanos secret placement binding)

## Field Comparison Rules

- **Exact Match Required**: All fields from the reference CR should be present in the cluster CR unless commented otherwise.
- **Flexibility Comments**: Pay careful attention to comments in reference files that indicate allowed variations:
  - "Environment-specific" = cluster-specific values allowed
  - "Any [field] allowed" = any valid value acceptable  
  - "Flexible configuration allowed" = variations permitted
- **Commented Field Override**: Comments in reference files override strict matching - if flexibility is indicated, different values are COMPLIANT
- **Unspecified Fields**: If a field is not explicitly defined in the reference CR, report it as a difference in the cluster CR unless there's an explicit comment in the file to ignore unspecifed fields.
- **Memory/CPU Values**: Treat equivalent formats as equal (2Gi = 2048Mi)

### **Special Comment Handling**

#### **ignore-unspecified-fields Comment**
When a reference file contains the header comment `# ignore-unspecified-fields: true`:
- **DO NOT** report cluster fields that are not present in the reference as differences
- **ONLY** validate fields that are explicitly defined in the reference CR
- **Example**: If reference has only `spec.watchAllNamespaces: true` and cluster has additional fields like `spec.provisioningNetwork: Disabled`, do NOT report the additional fields as differences

**CRITICAL VALIDATION STEP**: ALWAYS check the first 3 lines of EVERY reference file for this comment before reporting any configuration differences. Missing this comment leads to false non-compliance reports.

**MANDATORY CHECK PROCESS**:
1. Read lines 1-3 of every reference file before validation
2. If `# ignore-unspecified-fields: true` is found, mark ALL unspecified cluster fields as compliant
3. Only validate fields explicitly present in the reference CR

#### **Environment-specific Comments**
When reference files contain inline comments indicating environment-specific values:
- **DO NOT** report differences for these fields
- **Mark as COMPLIANT** regardless of cluster values
- **Common patterns**:
  - `# Environment-specific storage class name`
  - `# Environment-specific SSL certificate chain`
  - `# Entire data section is environment-specific`
  - `# Replace with actual server`

#### **Template Placeholder Handling**
When reference files contain template placeholders:
- **DO NOT** report differences when cluster has actual values vs placeholders
- **Mark as COMPLIANT** when cluster replaces placeholders with real values
- **Common patterns**:
  - `<registry.example.com:8443>` → actual registry URL
  - `<http-server-address:port>` → actual server address

### **Always Ignore**: 
- `metadata.creationTimestamp`
- `metadata.resourceVersion`
- `metadata.uid`
- `metadata.generation`
- `metadata.ownerReferences`
- `metadata.finalizers`
- `spec.finalizers`
- `status` (entire section)
- Any annotation starting with `kubectl.kubernetes.io/`
- Any annotation starting with `openshift.io/sa.scc.`
- Any annotation starting with `argocd.argoproj.io/`
- Any annotation starting with `installer.multicluster.openshift.io/`
- Any annotation starting with `installer.open-cluster-management.io/`
- Any label starting with `operators.coreos.com/local-storage-operator.`
- Any label starting with `operators.coreos.com/advanced-cluster-management.`
- Any label starting with `pod-security.kubernetes.io/`
- Any label starting with `olm.operatorgroup.uid`
- Label `kubernetes.io/metadata.name`
- Label `security.openshift.io/scc.podSecurityLabelSync`
- Label `app.kubernetes.io/instance`
- Label `installer.name`
- Label `installer.namespace`
- Label `multiclusterhubs.operator.open-cluster-management.io/managed-by`
- Label `agent-install.openshift.io/watch`
- Label `cluster.open-cluster-management.io/backup`

## Resource Discovery Guidelines

### **Reference Configuration Source**
**CRITICAL**: Always use reference CRs from the `telco-hub/configuration/cluster-compare/reference-crs/` directory ONLY.

**Reference Directory Structure**:
```
telco-hub/configuration/cluster-compare/reference-crs/
├── required/
│   ├── acm/
│   └── talm/
└── optional/
    ├── lso/
    ├── logging/
    └── odf-internal/
```

**Never use other reference directories**: Do not read from `telco-hub/configuration/reference-crs/` or any other location. The cluster-compare folder contains the authoritative reference configurations with proper flexibility comments.

### **Use the OpenShift CLI (oc) for Data Extraction**
All cluster resource data must be extracted using the `oc` client with appropriate commands. Extract resources in YAML format for accurate comparison against reference configurations.

### **Use Full API Paths**: Always use complete API resource paths for reliable discovery
When extracting live cluster data, use full API resource names to ensure accurate discovery. For example:
- `oc get namespaces` (or just `namespace` - core API)
- `oc get operatorgroups.operators.coreos.com` 
- `oc get subscriptions.operators.coreos.com`
- `oc get localvolumes.local.storage.openshift.io`

**Important**: Short forms like `oc get subscription` may not return expected results. Always use the full API path format to prevent false negatives in compliance validation.

### **Strict Namespace Compliance Requirement**
**CRITICAL**: Resources must exist in the exact namespace specified in the reference configuration. Any resource found in a different namespace is considered a MISSING RESOURCE and must be reported as non-compliant.

**Validation Strategy**:
1. **Extract reference namespace**: Always check the `metadata.namespace` field in the reference CR file
2. **Search only in specified namespace**: Use `oc get <resource-type> <resource-name> -n <reference-namespace>`
3. **Never use cross-namespace search**: Do not use `oc get <resource-type> -A` as this can mask namespace misalignments
4. **Report namespace mismatches**: If a resource exists but in wrong namespace, report as MISSING RESOURCE

**Example**: If `acmMirrorRegistryCM.yaml` specifies `namespace: multicluster-engine`, then the ConfigMap must exist in `multicluster-engine` namespace, not `openshift-config` or any other namespace.

### **Resource Identity and Assessment Rules**
**CRITICAL**: Kubernetes resources are uniquely identified by the triplet: **kind + name + namespace**

#### **Resource Identity Triplet**
A resource is considered present ONLY if all three identity fields match the reference:
1. **kind** (e.g., `ConfigMap`, `ManagedClusterSetBinding`, `Deployment`)
2. **name** (e.g., `"global"`, `"multiclusterhub"`, `"local-disks"`)  
3. **namespace** (e.g., `"openshift-storage"`, `"open-cluster-management"`)

**Identity Deviation = Missing Resource**: If ANY of the three identity fields differ from reference, treat as MISSING RESOURCE.

#### **Resource-Level Assessment**
- ✅ **COMPLIANT**: Correct identity triplet + all configuration fields match reference
- ⚠️ **WARNING**: Correct identity triplet + configuration drift in non-identity fields
- ❌ **MISSING**: Wrong kind OR wrong name OR wrong namespace

#### **Component-Level Assessment (Conservative)**
- ✅ **COMPLIANT**: ALL resources are COMPLIANT
- ❌ **NON-COMPLIANT**: ANY resource has WARNING or is MISSING

#### **Hub-Level Assessment**
- ✅ **COMPLIANT**: ALL components are COMPLIANT  
- ❌ **NON-COMPLIANT**: ANY component is NON-COMPLIANT

#### **Optional Component Treatment**
Once an optional component is present (any CR found), it receives the same assessment rules as Required components.

### **Compliance Chain Rule**
Apply this strict hierarchy with conservative assessment:
1. **Identity triplet deviation** → Resource is MISSING
2. **Configuration drift** → Resource has WARNING
3. **Any WARNING or MISSING resource** → Component is NON-COMPLIANT  
4. **Any NON-COMPLIANT component** → Hub is NON-COMPLIANT
5. **Optional components** → Follow same rules as Required once present

### **Assessment Validation Checklist**
Before finalizing compliance status, verify:
- [ ] **Reference Source**: Only used CRs from `telco-hub/configuration/cluster-compare/reference-crs/` directory
- [ ] **Component Scope**: Only analyzed components listed in "Hub Components" section (LSO, Logging, ODF, TALM, ACM)
- [ ] **Flexibility Comments**: Respected all "Environment-specific" and template placeholder comments
- [ ] **ignore-unspecified-fields**: Did not report extra cluster fields when this comment is present
- [ ] **Report Format Compliance**: Used EXACT prescribed format with NO additional subsections or explanatory text
- [ ] Identity triplet deviations (kind/name/namespace) treated as MISSING resources
- [ ] Configuration drift in non-identity fields treated as WARNING resources  
- [ ] Any WARNING or MISSING resource makes component NON-COMPLIANT
- [ ] Any NON-COMPLIANT component makes hub NON-COMPLIANT
- [ ] Optional components assessed with same rules as Required once present
- [ ] No percentage-based assessments for component compliance
- [ ] All findings use appropriate COMPLIANT/WARNING/NON-COMPLIANT/MISSING language

## Systematic Validation Process

### **MANDATORY: Component List Compliance**
**CRITICAL RULE**: The component CR lists in the "Hub Components" section are AUTHORITATIVE and COMPLETE. These lists define the EXACT scope of validation work.

**FORBIDDEN APPROACHES**:
- ❌ Ad-hoc file discovery (finding files randomly in directories)
- ❌ Partial validation (validating some but not all listed CRs)
- ❌ Assuming components are "good enough" with partial validation
- ❌ Adding files not listed in the component specifications

**MANDATORY APPROACH**:
- ✅ Use component CR lists as validation checklists
- ✅ Validate EVERY CR explicitly listed in each component
- ✅ Cross-reference discovered files against official lists
- ✅ Account for every listed CR before declaring component status

**ZERO TOLERANCE RULE**: If ANY listed CR is not validated, the validation is INCOMPLETE and INVALID. No shortcuts, no assumptions, no "good enough" assessments.

**VALIDATION SEQUENCE ENFORCEMENT**:
1. **FIRST**: Read component specifications from Hub Components section
2. **SECOND**: Create complete checklists with exact CR counts
3. **THIRD**: Systematically validate each listed CR
4. **FOURTH**: Verify 100% checklist completion before declaring component status
5. **NEVER**: Skip steps, assume components, or use partial validation

### **Step 1: Pre-Validation Checklist Creation**
**BEFORE starting ANY validation**, create complete checklists from the Hub Components section:

#### TALM Component Checklist (Required)
```
[ ] talmSubscription.yaml (Subscription to install TALM)
```

#### ACM Component Checklist (Required) - ALL 19 CRs MANDATORY
```
[ ] acmNS.yaml (Namespace: open-cluster-management)
[ ] acmOperGroup.yaml (OperatorGroup for ACM)
[ ] acmSubscription.yaml (Subscription to install ACM)
[ ] acmMCH.yaml (MultiClusterHub configuration)
[ ] acmMCE.yaml (MultiClusterEngine configuration)
[ ] acmProvisioning.yaml (Provisioning configuration)
[ ] acmMirrorRegistryCM.yaml (Mirror registry ConfigMap)
[ ] acmAgentServiceConfig.yaml (Agent service configuration)
[ ] observabilityNS.yaml (Observability namespace)
[ ] observabilityMCO.yaml (MultiClusterObservability)
[ ] observabilityOBC.yaml (ObjectBucketClaim)
[ ] observabilitySecret.yaml (Pull secret for observability)
[ ] pullSecretPolicy.yaml (Pull secret copy policy)
[ ] pullSecretPlacement.yaml (Pull secret placement)
[ ] pullSecretPlacementBinding.yaml (Pull secret placement binding)
[ ] pullSecretMCSB.yaml (Pull secret ManagedClusterSetBinding)
[ ] thanosSecretPolicy.yaml (Thanos secret policy)
[ ] thanosSecretPlacement.yaml (Thanos secret placement)
[ ] thanosSecretPlacementBinding.yaml (Thanos secret placement binding)
```

#### LSO Component Checklist (Optional)
```
[ ] lsoNS.yaml (Namespace: openshift-local-storage)
[ ] lsoOperatorGroup.yaml (OperatorGroup for LSO)
[ ] lsoSubscription.yaml (Subscription to install LSO)
[ ] lsoLocalVolume.yaml (LocalVolume configuration)
```

#### Logging Component Checklist (Optional)
```
[ ] clusterLogNS.yaml (Namespace: openshift-logging)
[ ] clusterLogOperGroup.yaml (OperatorGroup for cluster logging)
[ ] clusterLogSubscription.yaml (Subscription to install cluster logging)
```

#### ODF Component Checklist (Optional)
```
[ ] odfNS.yaml (Namespace: openshift-storage)
[ ] odfOperatorGroup.yaml (OperatorGroup for ODF)
[ ] odfSubscription.yaml (Subscription to install ODF)
[ ] storageCluster.yaml (StorageCluster configuration)
```

### **Step 2: Mandatory Cross-Reference Validation**
**BEFORE declaring any component status**, verify:

1. **File Count Verification**: 
   - Count actual reference files in component directory
   - Verify count matches the checklist exactly
   - Report discrepancies as validation errors

2. **Filename Verification**:
   - Every checklist item has corresponding reference file
   - Every reference file appears on the checklist
   - No extra files, no missing files

3. **Resource Identity Extraction**:
   - **CRITICAL**: Read EVERY reference file to extract exact resource identity (kind+name+namespace)
   - **NEVER** assume resource names from filenames
   - **ALWAYS** use metadata.name from reference file for cluster searches
   - **EXAMPLE**: `pullSecretMCSB.yaml` contains `name: default`, NOT `name: global`

4. **Component Completion Verification**:
   - Every checklist item validated (✅, ⚠️, or ❌)
   - No checklist items left unchecked
   - Component status reflects ALL items, not partial validation

### **Step 3: Component Discovery**

### **Step 2: Field-by-Field Analysis** 
For EACH CR, create a validation matrix comparing reference vs cluster:

**Example for LocalVolume:**
```
Field                    | Reference Value         | Cluster Value     | Status | Notes
-------------------------|-------------------------|-------------------|--------|-------
metadata.name            | local-disks             | [extract]         | [ ]    |
metadata.namespace       | openshift-local-storage | [extract]         | [ ]    |
spec.logLevel            | Normal                  | [extract]         | [ ]    |
spec.managementState     | Managed                 | [extract]         | [ ]    |
spec.nodeSelector        | [full structure]        | [extract]         | [ ]    |
```

### **Step 3: Verification Checkpoint**
Before declaring any field "missing":
1. **Double-check source data** - Re-examine the extracted YAML
2. **Search explicitly** - Use `grep` or manual search for the field name
3. **Verify extraction completeness** - Ensure full CR was captured
4. **Cross-reference resource names** - Check actual cluster resource names against reference file names (they may differ)
5. **Verify namespace compliance** - Ensure resource is searched in the exact namespace specified in reference configuration

### **Step 4: Documentation Requirements**
- Quote exact field paths and values from both reference and cluster
- Provide line-by-line evidence for any differences claimed
- Use `diff` format only for confirmed actual differences

### **Step 5: Quality Assurance Checklist**
Before finalizing any compliance report, verify:
- [ ] **Reference Directory**: Only read from `telco-hub/configuration/cluster-compare/reference-crs/`
- [ ] **Component Scope**: Only analyzed Hub Components (LSO, Logging, ODF, TALM, ACM)
- [ ] **Flexibility Compliance**: Honored all environment-specific and template placeholder comments
- [ ] **ignore-unspecified-fields**: Properly handled this directive when present
- [ ] **Report Format Adherence**: Used EXACT prescribed format without additional subsections or explanatory details
- [ ] **Data Completeness**: All CRs extracted successfully using full API paths
- [ ] **Field Coverage**: Every reference field explicitly checked against cluster data  
- [ ] **Evidence-Based**: All claimed differences supported by quoted YAML excerpts
- [ ] **Verification**: Each "missing" field manually searched in source data
- [ ] **Consistency**: Report status matches detailed findings
- [ ] **Accuracy**: No assumptions made - only document what is directly observable

## Reporting Format

### **CRITICAL FORMAT ENFORCEMENT**
**ZERO TOLERANCE FOR FORMAT DEVIATIONS**: The report format below is MANDATORY and EXACT. Any deviation from this prescribed structure will be considered a compliance failure.

**FORMAT COMPLIANCE RULE**: If it's not explicitly listed in the prescribed format below, DO NOT include it in the report.

- Generate a **comprehensive Markdown report** saved as `cluster-comparison-report.md` in the same directory as this file.
  Delete any existing `cluster-comparison-report.md` before creating a new one. Never update an existing report.
- **Accurate Timestamps**: Always use `run_terminal_cmd` with `date -u +"%Y-%m-%dT%H:%M:%SZ"` to get the actual current UTC timestamp instead of hardcoding dates.

### Report Structure
```markdown
# Hub Cluster Comparison Report
*Generated: [USE: `date -u +"%Y-%m-%dT%H:%M:%SZ"` to get actual current timestamp]*

## Summary

| Component | Status | Notes |
|-----------|--------|-------|
| [Component Name] | ✅/❌ COMPLIANT/NON-COMPLIANT | [brief status explanation] |
| [Component Name] | ✅/❌ COMPLIANT/NON-COMPLIANT | [brief status explanation] |

**Overall Hub Status: ✅/❌ COMPLIANT/NON-COMPLIANT**

## Detailed Findings

### [✅/❌] [Component Name]

#### Resource Presence
- ✅/⚠️/❌ [resource1.yaml] ([Resource Type]: [name])
- ✅/⚠️/❌ [resource2.yaml] ([Resource Type]: [name])

#### Configuration Differences
*Always show this section*

*If no non-compliant differences:*
All configuration differences are within expected behavior per reference flexibility guidelines.

*If non-compliant differences exist:*
##### [ResourceName.yaml] - [Field Path]
```diff
- cluster:     [actual value]
+ reference:   [expected value]
# Impact: [why this matters]
```

**CRITICAL**: Do NOT add any additional subsections, explanations, or details beyond the above format. Do NOT create sections like "Environment-specific configurations properly applied" or similar. Stick strictly to the prescribed format.

**MANDATORY COMPLIANCE CHECK**: Before writing ANY field difference, verify:
1. The reference file does NOT contain `# ignore-unspecified-fields: true` comment (for unspecified fields only)
2. The field is NOT marked as environment-specific in reference comments
3. The field is NOT a template placeholder being replaced with actual values

**ZERO ELABORATION RULE**: Use ONLY the exact sections and subsections listed above. No additional explanatory text, analysis, or commentary.

## Overall Assessment

**Status: ✅/❌ COMPLIANT/NON-COMPLIANT**

[Executive summary paragraph about overall compliance status]

### Resolution Steps
*Only show if there are non-compliant components*

[Instructions for fixing only the failed components]
```

### Status Icons

#### Resource-Level Status
- ✅ **COMPLIANT**: Correct identity triplet (kind+name+namespace) + matching configuration
- ⚠️ **WARNING**: Correct identity triplet + configuration drift in non-identity fields
- ❌ **MISSING**: Wrong kind OR wrong name OR wrong namespace

#### Component/Hub-Level Status  
- ✅ **COMPLIANT**: ALL resources COMPLIANT
- ❌ **NON-COMPLIANT**: ANY resource has WARNING or is MISSING

**Note**: WARNING resources cause component-level NON-COMPLIANCE due to conservative assessment approach.

### Reporting Guidelines

#### **Differences-Only Approach**
- **Show only what differs** - Don't list compliant fields
- **Group by resource** - Organize differences by CR file
- **Provide context** - Explain why each difference matters

#### **Field Difference Format**
```diff
# Field: [exact.field.path]
- cluster:     [actual cluster value]
+ reference:   [expected reference value]
# Impact: [compliance/functional impact]
```

#### **Missing Resource Format**
```diff
# Resource: [resourceName.yaml] ([ResourceType]: [name])
- cluster:     Resource missing
+ reference:   [ResourceType] "[name]" in namespace: [namespace]
# Impact: Required resource not found in expected location
```

#### **Warning Resource Format**
```diff
# Field: [exact.field.path] in [resourceName.yaml]
- cluster:     [actual cluster value]
+ reference:   [expected reference value]  
# Impact: Configuration drift - resource present with correct identity but field deviation
```

#### **Conditional Sections**
Always include these sections:
- **Configuration Differences**: Always show, with appropriate message for compliant vs non-compliant differences
- **Resolution Steps**: Only if there are non-compliant components

## FINAL REPORT VALIDATION CHECKLIST

**MANDATORY PRE-SUBMISSION VERIFICATION**: Before finalizing any report, verify ALL of the following:

### Format Compliance Checklist
- [ ] **Exact Title**: Used "# Hub Cluster Comparison Report" (no variations)
- [ ] **Exact Timestamp**: Used actual UTC timestamp from `date -u +"%Y-%m-%dT%H:%M:%SZ"`
- [ ] **Only Prescribed Sections**: Report contains ONLY Summary, Detailed Findings, Overall Assessment, Resolution Steps (if needed)
- [ ] **No Forbidden Sections**: Verified zero presence of Executive Summary, Risk Assessment, Recommendations, etc.
- [ ] **Standard Component Names**: Used exact component names: TALM, ACM, LSO, Logging, ODF
- [ ] **Prescribed Status Icons**: Used only ✅ COMPLIANT or ❌ NON-COMPLIANT for components

### Technical Compliance Checklist  
- [ ] **ignore-unspecified-fields Compliance**: Did NOT report extra cluster fields when `# ignore-unspecified-fields: true` exists
- [ ] **Environment-specific Compliance**: Did NOT report differences for fields marked as environment-specific
- [ ] **Template Placeholder Compliance**: Did NOT report differences when cluster replaces placeholders with actual values
- [ ] **Identity Triplet Validation**: Verified kind+name+namespace matching for all resources
- [ ] **Namespace Compliance**: Ensured all resources searched in exact reference namespace

### Content Accuracy Checklist
- [ ] **Reference Source**: Used ONLY `telco-hub/configuration/cluster-compare/reference-crs/` directory
- [ ] **Component Scope**: Analyzed ONLY the 5 hub components (TALM, ACM, LSO, Logging, ODF)
- [ ] **Full API Paths**: Used complete API resource names in all oc commands
- [ ] **Evidence-Based**: All claimed differences supported by actual extracted data
- [ ] **Conservative Assessment**: Applied strict hierarchy: ANY warning/missing → component NON-COMPLIANT

### Component List Compliance Checklist
- [ ] **Pre-Validation Checklists**: Created complete checklists from Hub Components section BEFORE starting validation
- [ ] **TALM Completeness**: Validated exactly 1 CR as listed (talmSubscription.yaml)
- [ ] **ACM Completeness**: Validated exactly 19 CRs as listed (all CRs from acmNS.yaml through thanosSecretPlacementBinding.yaml)
- [ ] **LSO Completeness**: If present, validated exactly 4 CRs as listed
- [ ] **Logging Completeness**: If present, validated exactly 3 CRs as listed  
- [ ] **ODF Completeness**: If present, validated exactly 4 CRs as listed
- [ ] **No Ad-Hoc Discovery**: Did NOT discover files randomly - used ONLY the official component lists
- [ ] **Cross-Reference Verification**: Verified every checklist item corresponds to actual reference file
- [ ] **No Extra Files**: Did NOT include files not explicitly listed in component specifications
- [ ] **Complete Validation**: Every listed CR has validation status (✅, ⚠️, or ❌) - no unchecked items
- [ ] **Accurate Resource Identity**: Used exact metadata.name from reference files, NOT filename assumptions
- [ ] **Systematic Resource Extraction**: Read every reference file to extract kind+name+namespace before cluster searches

**FAILURE TO COMPLETE THIS CHECKLIST CONSTITUTES FORMAT NON-COMPLIANCE**