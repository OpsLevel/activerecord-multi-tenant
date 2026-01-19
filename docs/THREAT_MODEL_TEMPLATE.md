# Threat Model Template

**Version:** [e.g., 1.0]
**Last Updated:** [YYYY-MM-DD]
**Owner:** [Team/Individual responsible]
**Reviewers:** [Comma-separated list of reviewers]

---

## Executive Summary

[Provide a brief overview of the system being modeled, its purpose, and why threat modeling is important for this component. Include a high-level summary of the most critical threats.]

---

## Table of Contents

1. [System Architecture Overview](#system-architecture-overview)
2. [Assets and Data Classification](#assets-and-data-classification)
3. [Trust Boundaries](#trust-boundaries)
4. [Threat Analysis (STRIDE)](#threat-analysis-stride)
5. [Attack Vectors and Entry Points](#attack-vectors-and-entry-points)
6. [Existing Security Controls](#existing-security-controls)
7. [Threat Mitigation Recommendations](#threat-mitigation-recommendations)
8. [Security Testing Requirements](#security-testing-requirements)
9. [Threat Model Review Cadence](#threat-model-review-cadence)
10. [References](#references)

---

## System Architecture Overview

### Component Description

[Describe the system/component being modeled. Include:]
- Primary purpose and functionality
- Key technologies and frameworks used
- Integration points with other systems
- User types and access patterns

### Data Flow

[Create a diagram or detailed description showing how data flows through the system. Include:]

```
[Example Data Flow Diagram]

┌─────────────────┐
│  External User  │
└────────┬────────┘
         │
         ▼
┌─────────────────────────┐
│  Entry Point            │
└────────┬────────────────┘
         │
         ▼
[Continue flow...]
```

### Critical Components

[List and describe the most security-critical components:]

1. **Component Name**: Brief description and security relevance
2. **Component Name**: Brief description and security relevance
3. [Add more as needed]

---

## Assets and Data Classification

### Critical Assets

[Use table format to catalog assets:]

| Asset | Classification | Description | Impact if Compromised |
|-------|---------------|-------------|----------------------|
| [Asset Name] | [Critical/High/Medium/Low] | [Description] | [Impact description] |
| [Asset Name] | [Critical/High/Medium/Low] | [Description] | [Impact description] |

### Data Classification

- **Highly Sensitive**: [Examples: PII, credentials, encryption keys]
- **Sensitive**: [Examples: Business logic, API schemas]
- **Internal**: [Examples: Configuration, logs]
- **Public**: [Examples: Documentation, public APIs]

### Credentials and Secrets

[List types of credentials and how they are managed:]
- [Credential type]: [Storage and management approach]
- [Credential type]: [Storage and management approach]

---

## Trust Boundaries

### Boundary 1: [Name]

- **Trust Level**: [Zero trust / Partial trust / Full trust]
- **Assumptions**: [What do we assume about this boundary?]
- **Risks**: [What happens if assumptions are violated?]

### Boundary 2: [Name]

- **Trust Level**: [Zero trust / Partial trust / Full trust]
- **Assumptions**: [What do we assume about this boundary?]
- **Risks**: [What happens if assumptions are violated?]

[Add more boundaries as needed]

---

## Threat Analysis (STRIDE)

### Spoofing Identity

| Threat ID | Threat | Likelihood | Impact | Risk | Mitigation |
|-----------|--------|------------|---------|------|------------|
| S-01 | [Threat description] | [Low/Medium/High] | [Low/Medium/High/Critical] | [Low/Medium/High/Critical] | [Mitigation approach] |
| S-02 | [Threat description] | [Low/Medium/High] | [Low/Medium/High/Critical] | [Low/Medium/High/Critical] | [Mitigation approach] |

### Tampering

| Threat ID | Threat | Likelihood | Impact | Risk | Mitigation |
|-----------|--------|------------|---------|------|------------|
| T-01 | [Threat description] | [Low/Medium/High] | [Low/Medium/High/Critical] | [Low/Medium/High/Critical] | [Mitigation approach] |
| T-02 | [Threat description] | [Low/Medium/High] | [Low/Medium/High/Critical] | [Low/Medium/High/Critical] | [Mitigation approach] |

### Repudiation

| Threat ID | Threat | Likelihood | Impact | Risk | Mitigation |
|-----------|--------|------------|---------|------|------------|
| R-01 | [Threat description] | [Low/Medium/High] | [Low/Medium/High/Critical] | [Low/Medium/High/Critical] | [Mitigation approach] |
| R-02 | [Threat description] | [Low/Medium/High] | [Low/Medium/High/Critical] | [Low/Medium/High/Critical] | [Mitigation approach] |

### Information Disclosure

| Threat ID | Threat | Likelihood | Impact | Risk | Mitigation |
|-----------|--------|------------|---------|------|------------|
| I-01 | [Threat description] | [Low/Medium/High] | [Low/Medium/High/Critical] | [Low/Medium/High/Critical] | [Mitigation approach] |
| I-02 | [Threat description] | [Low/Medium/High] | [Low/Medium/High/Critical] | [Low/Medium/High/Critical] | [Mitigation approach] |

### Denial of Service

| Threat ID | Threat | Likelihood | Impact | Risk | Mitigation |
|-----------|--------|------------|---------|------|------------|
| D-01 | [Threat description] | [Low/Medium/High] | [Low/Medium/High/Critical] | [Low/Medium/High/Critical] | [Mitigation approach] |
| D-02 | [Threat description] | [Low/Medium/High] | [Low/Medium/High/Critical] | [Low/Medium/High/Critical] | [Mitigation approach] |

### Elevation of Privilege

| Threat ID | Threat | Likelihood | Impact | Risk | Mitigation |
|-----------|--------|------------|---------|------|------------|
| E-01 | [Threat description] | [Low/Medium/High] | [Low/Medium/High/Critical] | [Low/Medium/High/Critical] | [Mitigation approach] |
| E-02 | [Threat description] | [Low/Medium/High] | [Low/Medium/High/Critical] | [Low/Medium/High/Critical] | [Mitigation approach] |

**Risk Calculation**: Risk = Likelihood × Impact
- **Critical Risk**: Requires immediate attention
- **High Risk**: Address within current sprint/iteration
- **Medium Risk**: Schedule for upcoming release
- **Low Risk**: Monitor and address as resources permit

---

## Attack Vectors and Entry Points

### External Entry Points

1. **[Entry Point Name]**
   - **Vector**: [How can this be attacked?]
   - **Entry Point**: [Where does the attack enter?]
   - **Attack**: [Specific attack scenario]
   - **Mitigation**: [How to prevent/detect/respond?]

2. **[Entry Point Name]**
   - **Vector**: [How can this be attacked?]
   - **Entry Point**: [Where does the attack enter?]
   - **Attack**: [Specific attack scenario]
   - **Mitigation**: [How to prevent/detect/respond?]

### Internal Entry Points

3. **[Entry Point Name]**
   - **Vector**: [How can this be attacked?]
   - **Entry Point**: [Where does the attack enter?]
   - **Attack**: [Specific attack scenario]
   - **Mitigation**: [How to prevent/detect/respond?]

### Dependencies

4. **[Dependency Name]**
   - **Dependency**: [Version/description]
   - **Attack**: [Potential vulnerability]
   - **Mitigation**: [Update strategy, monitoring]

---

## Existing Security Controls

### Preventive Controls

1. **[Control Name]**
   - **Control**: [Description of the control]
   - **Location**: [Where in the codebase/system]
   - **Effectiveness**: [High/Medium/Low - How well does it work?]

2. **[Control Name]**
   - **Control**: [Description of the control]
   - **Location**: [Where in the codebase/system]
   - **Effectiveness**: [High/Medium/Low - How well does it work?]

### Detective Controls

3. **[Control Name]**
   - **Control**: [Description of the control]
   - **Location**: [Where in the codebase/system]
   - **Effectiveness**: [High/Medium/Low - How well does it work?]

### Response Controls

4. **[Control Name]**
   - **Control**: [Description of the control]
   - **Location**: [Where in the codebase/system]
   - **Effectiveness**: [High/Medium/Low - How well does it work?]

---

## Threat Mitigation Recommendations

### Critical Priority (Immediate Action Required)

#### REC-01: [Recommendation Title]
- **Threat**: [Related threat ID(s) and description]
- **Action**:
  - [Specific action item 1]
  - [Specific action item 2]
  - [Specific action item 3]
- **Effort**: [Low/Medium/High]
- **Impact**: [Expected security improvement]

#### REC-02: [Recommendation Title]
- **Threat**: [Related threat ID(s) and description]
- **Action**:
  - [Specific action items]
- **Effort**: [Low/Medium/High]
- **Impact**: [Expected security improvement]

### High Priority (Within 1 Quarter)

#### REC-03: [Recommendation Title]
- **Threat**: [Related threat ID(s) and description]
- **Action**:
  - [Specific action items]
- **Effort**: [Low/Medium/High]
- **Impact**: [Expected security improvement]

### Medium Priority (Within 2 Quarters)

#### REC-04: [Recommendation Title]
- **Threat**: [Related threat ID(s) and description]
- **Action**:
  - [Specific action items]
- **Effort**: [Low/Medium/High]
- **Impact**: [Expected security improvement]

### Low Priority (Ongoing/As Needed)

#### REC-05: [Recommendation Title]
- **Threat**: [Related threat ID(s) and description]
- **Action**:
  - [Specific action items]
- **Effort**: [Low/Medium/High]
- **Impact**: [Expected security improvement]

---

## Security Testing Requirements

### Unit Tests

- [ ] [Specific security-focused unit test requirement]
- [ ] [Specific security-focused unit test requirement]
- [ ] [Add more as needed]

### Integration Tests

- [ ] [End-to-end security scenario]
- [ ] [Integration point security validation]
- [ ] [Add more as needed]

### Security-Specific Tests

- [ ] **[Category]**: [Specific test requirement]
- [ ] **[Category]**: [Specific test requirement]
- [ ] **[Category]**: [Specific test requirement]

### Penetration Testing

- [ ] [Specific pen test scenario]
- [ ] [Specific pen test scenario]
- [ ] [Add more as needed]

### Dependency Scanning

- [ ] [Automated scanning tool/process]
- [ ] [Monitoring approach]
- [ ] [Update process]

---

## Threat Model Review Cadence

### Regular Review Schedule

- **[Frequency]**: [Who reviews what and when]
- **[Frequency]**: [Who reviews what and when]
- **[Frequency]**: [Who reviews what and when]

### Triggers for Unscheduled Review

1. [Trigger event that requires immediate review]
2. [Trigger event that requires immediate review]
3. [Trigger event that requires immediate review]
4. [Add more as needed]

### Review Process

1. **[Step Name]**: [Description of review step]
2. **[Step Name]**: [Description of review step]
3. **[Step Name]**: [Description of review step]
4. **[Step Name]**: [Description of review step]

### Stakeholders

- **Owner**: [Role/Team]
- **Contributors**: [Roles/Teams]
- **Reviewers**: [Roles/Teams]
- **Approvers**: [Roles/Teams]

---

## References

### Security Documentation

- [Link to relevant security standard/guide]
- [Link to relevant security standard/guide]
- [Link to relevant security standard/guide]

### Related Documentation

- [Link to system documentation]
- [Link to API documentation]
- [Link to architecture diagrams]

### Security Policies

- **Vulnerability Reporting**: [Contact/process]
- **Security Updates**: [How to stay informed]
- **Dependency Updates**: [Update process]

### Incident Response

- **Internal Contact**: [Contact information]
- **External Contact**: [Contact information]
- **Response SLA**:
  - Critical: [Timeframe]
  - High: [Timeframe]
  - Medium: [Timeframe]
  - Low: [Timeframe]

### Compliance

- **[Regulation]**: [How this system supports compliance]
- **[Standard]**: [How this system supports compliance]
- **[Certification]**: [How this system supports compliance]

---

## Appendix: Common Attack Scenarios

### Scenario 1: [Attack Scenario Name]

**Attack Chain**:
1. [Step 1 of attack]
2. [Step 2 of attack]
3. [Step 3 of attack]
4. [Final impact]

**Countermeasures**:
- [Countermeasure 1]
- [Countermeasure 2]
- [Countermeasure 3]

### Scenario 2: [Attack Scenario Name]

**Attack Chain**:
1. [Step 1 of attack]
2. [Step 2 of attack]
3. [Step 3 of attack]
4. [Final impact]

**Countermeasures**:
- [Countermeasure 1]
- [Countermeasure 2]
- [Countermeasure 3]

[Add more scenarios as needed]

---

**Document Classification**: [Public/Internal/Confidential/Restricted]
**Distribution**: [Who should have access to this document]
**Next Review Date**: [YYYY-MM-DD]

---

## Instructions for Using This Template

1. **Fill in all bracketed placeholders** with specific information for your system
2. **Remove this instructions section** before finalizing the document
3. **Customize sections** as needed for your specific context
4. **Add/remove entries** in tables and lists based on your system's complexity
5. **Use STRIDE consistently** - ensure all threat categories are considered
6. **Prioritize realistically** - focus on threats with highest risk scores
7. **Make it actionable** - recommendations should be specific and measurable
8. **Keep it updated** - this is a living document that evolves with your system
9. **Get stakeholder buy-in** - ensure security and development teams align on priorities
10. **Link to other docs** - connect threat model to architecture, incident response, etc.

### Risk Scoring Guidelines

**Likelihood**:
- **Low**: Unlikely to occur, requires specific conditions
- **Medium**: Could occur with moderate effort or opportunity
- **High**: Likely to occur, easy to exploit

**Impact**:
- **Low**: Minimal damage, limited scope
- **Medium**: Moderate damage, affects specific users/data
- **High**: Significant damage, widespread impact
- **Critical**: Catastrophic damage, complete system compromise

**Risk = Likelihood × Impact**:
- Low × Low = Low Risk
- Low × Medium = Low Risk
- Low × High = Medium Risk
- Low × Critical = Medium Risk
- Medium × Low = Low Risk
- Medium × Medium = Medium Risk
- Medium × High = High Risk
- Medium × Critical = High Risk
- High × Low = Medium Risk
- High × Medium = High Risk
- High × High = Critical Risk
- High × Critical = Critical Risk

### STRIDE Reference

- **Spoofing**: Pretending to be someone/something else
- **Tampering**: Modifying data or code
- **Repudiation**: Denying actions or changes
- **Information Disclosure**: Exposing information to unauthorized parties
- **Denial of Service**: Making system unavailable
- **Elevation of Privilege**: Gaining unauthorized capabilities

---

*This is a template document. Customize it for your specific system and security requirements.*
