# Threat Model: activerecord-multi-tenant

**Version:** 1.0
**Last Updated:** 2024-01-19
**Owner:** Security Team
**Reviewers:** Development Team, Operations Team

---

## Executive Summary

This document provides a comprehensive threat model for the `activerecord-multi-tenant` library, a Ruby gem that enables multi-tenant database support for ActiveRecord/Rails applications. The library is designed to work with distributed PostgreSQL databases, particularly Citus, to ensure tenant isolation and efficient query routing.

As a critical security component responsible for tenant isolation in multi-tenant applications, this library must be thoroughly evaluated for potential security vulnerabilities that could lead to data breaches, unauthorized access, or tenant data leakage.

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

**activerecord-multi-tenant** is a Ruby gem that provides multi-tenancy support for Rails applications using ActiveRecord ORM. It operates by:

1. **Tenant Context Management**: Maintains current tenant context through `MultiTenant.current_tenant` using `ActiveSupport::CurrentAttributes`
2. **Query Rewriting**: Automatically injects tenant_id clauses into SQL queries via Arel AST manipulation
3. **Model Extensions**: Extends ActiveRecord models with tenant-scoping behavior
4. **Controller Integration**: Provides before_action filters to set tenant context in Rails controllers
5. **Background Job Support**: Integrates with Sidekiq to preserve tenant context across async operations
6. **Migration Helpers**: Provides utilities for distributed table setup with Citus

### Data Flow

```
┌─────────────────┐
│  Web Request    │
└────────┬────────┘
         │
         ▼
┌─────────────────────────┐
│  Controller Filter      │
│  (set_current_tenant)   │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│  MultiTenant.with()     │
│  (Thread-local context) │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│  ActiveRecord Query     │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│  Query Rewriter         │
│  (Inject tenant_id)     │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│  PostgreSQL/Citus       │
│  (Distributed Query)    │
└─────────────────────────┘
```

### Critical Components

1. **MultiTenant.current_tenant**: Thread-local storage for tenant context
2. **Query Rewriter**: Arel visitor that modifies SQL AST
3. **Model Extensions**: Callbacks and validations for tenant enforcement
4. **Controller Extensions**: Request-scoped tenant setting
5. **Sidekiq Middleware**: Tenant serialization/deserialization for background jobs

---

## Assets and Data Classification

### Critical Assets

| Asset | Classification | Description | Impact if Compromised |
|-------|---------------|-------------|----------------------|
| **Tenant Context** | Critical | Current tenant ID stored in thread-local storage | Cross-tenant data exposure |
| **Database Queries** | Critical | SQL queries with tenant_id clauses | Unauthorized data access |
| **Tenant Foreign Keys** | Critical | tenant_id columns in database tables | Tenant isolation breach |
| **Sidekiq Job Payloads** | High | Serialized tenant information in job queues | Background job privilege escalation |
| **Model Callbacks** | High | Tenant enforcement logic in AR callbacks | Bypass of tenant isolation |
| **Session Data** | High | User session containing tenant identifiers | Session hijacking/fixation |

### Data Classification

- **Highly Sensitive**: Tenant isolation boundaries, tenant_id values, cross-tenant queries
- **Sensitive**: Application queries, model associations, controller actions
- **Internal**: Configuration settings, migration helpers

### Credentials and Secrets

- Database connection credentials (handled by Rails, not this gem)
- Session tokens containing tenant identifiers (application responsibility)
- Redis credentials for Sidekiq (if used)

---

## Trust Boundaries

### Boundary 1: Application Code → Library

- **Trust Level**: Partial trust
- **Assumptions**: Application correctly identifies and authenticates tenants
- **Risks**: Malicious/buggy application code setting wrong tenant context

### Boundary 2: Library → Database

- **Trust Level**: Full trust in query rewriting
- **Assumptions**: Generated SQL is correct and cannot be bypassed
- **Risks**: SQL injection, query rewriter bugs allowing tenant filter bypass

### Boundary 3: Web Request → Application

- **Trust Level**: Zero trust
- **Assumptions**: All user input is potentially malicious
- **Risks**: Session fixation, parameter tampering, authentication bypass

### Boundary 4: Sidekiq Jobs → Application

- **Trust Level**: Partial trust
- **Assumptions**: Job payloads haven't been tampered with
- **Risks**: Job queue manipulation, tenant context injection

### Boundary 5: Library Code → ActiveRecord/Rails

- **Trust Level**: Full trust
- **Assumptions**: Rails framework behaves as documented
- **Risks**: Framework vulnerabilities, unexpected behavior in callbacks

---

## Threat Analysis (STRIDE)

### Spoofing Identity

| Threat ID | Threat | Likelihood | Impact | Risk | Mitigation |
|-----------|--------|------------|---------|------|------------|
| S-01 | Attacker manipulates session data to impersonate different tenant | Medium | Critical | **High** | Application must implement secure tenant identification from authenticated user |
| S-02 | Background job contains forged tenant_id in Sidekiq payload | Low | Critical | **Medium** | Validate tenant exists before processing; use signed job payloads |
| S-03 | Controller bypass allows skipping tenant context setting | Medium | Critical | **High** | Enforce opt-in via `set_current_tenant_through_filter`; code review |
| S-04 | Multi-tenant model instantiated with arbitrary tenant_id | Medium | High | **Medium** | Tenant immutability checks in model extensions |

### Tampering

| Threat ID | Threat | Likelihood | Impact | Risk | Mitigation |
|-----------|--------|------------|---------|------|------------|
| T-01 | Direct SQL execution bypasses query rewriter | Medium | Critical | **High** | Query monitor to detect unscoped queries; documentation warnings |
| T-02 | Attacker modifies tenant_id after record creation | Low | Critical | **Medium** | `TenantIsImmutable` exception raised on change |
| T-03 | Raw SQL in migrations lacks tenant scoping | Medium | High | **Medium** | Migration helpers; code review of raw SQL |
| T-04 | Query rewriter bug allows WHERE clause injection | Low | Critical | **Medium** | Comprehensive test coverage; fuzz testing of Arel AST |
| T-05 | Monkey-patching disables tenant enforcement | Low | High | **Low** | Code review; runtime integrity checks |

### Repudiation

| Threat ID | Threat | Likelihood | Impact | Risk | Mitigation |
|-----------|--------|------------|---------|------|------------|
| R-01 | Cross-tenant data access cannot be traced to specific user | Medium | Medium | **Medium** | Application-level audit logging with tenant context |
| R-02 | MultiTenant.without() usage not logged | Medium | Medium | **Medium** | Wrap `without()` with audit logging; restrict usage |
| R-03 | Direct database access bypasses all logging | Low | High | **Medium** | Database audit logs; restrict direct DB access |

### Information Disclosure

| Threat ID | Threat | Likelihood | Impact | Risk | Mitigation |
|-----------|--------|------------|---------|------|------------|
| I-01 | Query rewriter failure exposes cross-tenant data | Low | Critical | **Medium** | Fail-safe defaults; extensive testing |
| I-02 | Error messages leak tenant_id values | Medium | Low | **Low** | Sanitize error messages; exception handling |
| I-03 | Unscoped queries in associations expose data | Medium | Critical | **High** | Association wrapping; query monitoring |
| I-04 | Race condition in tenant context switching leaks data | Low | Critical | **Medium** | Thread-local storage; comprehensive concurrency tests |
| I-05 | Cache poisoning causes tenant data bleed | Medium | High | **Medium** | Statement cache disabled for multi-tenant models |
| I-06 | Reflection methods expose tenant isolation logic | Low | Low | **Low** | Code obfuscation not recommended; accept risk |

### Denial of Service

| Threat ID | Threat | Likelihood | Impact | Risk | Mitigation |
|-----------|--------|------------|---------|------|------------|
| D-01 | Tenant context never cleared causing memory leak | Low | Medium | **Low** | Ensure/rescue blocks in `with()` method |
| D-02 | Malformed tenant_id causes query failures | Medium | Medium | **Medium** | Input validation; error handling |
| D-03 | Recursive tenant switching causes stack overflow | Low | Low | **Low** | Depth limiting not currently implemented |
| D-04 | Mass assignment of tenant_id on many records | Medium | Medium | **Medium** | Rate limiting at application level |

### Elevation of Privilege

| Threat ID | Threat | Likelihood | Impact | Risk | Mitigation |
|-----------|--------|------------|---------|------|------------|
| E-01 | MultiTenant.without() allows unrestricted data access | High | Critical | **Critical** | Restrict usage; code review; consider deprecation |
| E-02 | Write-only mode allows querying other tenants' data | Medium | Critical | **High** | Clear documentation; intended for migration only |
| E-03 | Setting tenant_id=0 bypasses tenant checks | Medium | High | **Medium** | Document special semantics; validate against abuse |
| E-04 | Privilege escalation via constantize() in Sidekiq | Low | High | **Medium** | Whitelist allowed tenant class names |
| E-05 | Application code can arbitrarily change current_tenant | High | Critical | **High** | Application responsibility; enforce authorization |

---

## Attack Vectors and Entry Points

### External Entry Points

1. **HTTP Requests**
   - **Vector**: Session manipulation, cookie tampering
   - **Entry Point**: Controller `before_action` filters
   - **Attack**: Session fixation to access different tenant
   - **Mitigation**: Secure session management, HTTPS only, HttpOnly cookies

2. **API Endpoints**
   - **Vector**: Parameter injection, authentication bypass
   - **Entry Point**: REST/GraphQL APIs using gem
   - **Attack**: API requests without proper tenant context
   - **Mitigation**: API authentication, rate limiting, input validation

3. **Background Job Queue**
   - **Vector**: Job payload manipulation
   - **Entry Point**: Sidekiq Redis queue
   - **Attack**: Inject jobs with arbitrary tenant_id
   - **Mitigation**: Signed job payloads, Redis authentication, network isolation

### Internal Entry Points

4. **ActiveRecord Queries**
   - **Vector**: Unscoped queries, raw SQL
   - **Entry Point**: Model methods, controllers
   - **Attack**: Direct queries bypassing query rewriter
   - **Mitigation**: Query monitoring, code review, linting

5. **Console/Rake Tasks**
   - **Vector**: Administrative access
   - **Entry Point**: Rails console, rake tasks
   - **Attack**: Running commands without tenant context
   - **Mitigation**: Audit logging, access controls, mandatory tenant context

6. **Database Migrations**
   - **Vector**: Raw SQL in migrations
   - **Entry Point**: Migration files
   - **Attack**: Data manipulation across tenants
   - **Mitigation**: Migration helpers, code review, testing

### Dependencies

7. **Rails Framework**
   - **Dependency**: Rails >= 6.0
   - **Attack**: CVEs in Rails affecting query generation
   - **Mitigation**: Keep Rails updated, monitor security advisories

8. **PostgreSQL/Citus**
   - **Dependency**: PostgreSQL database with optional Citus extension
   - **Attack**: Database vulnerabilities, SQL injection
   - **Mitigation**: Database security hardening, parameterized queries

9. **Sidekiq (Optional)**
   - **Dependency**: Sidekiq for background jobs
   - **Attack**: Job queue tampering, Redis vulnerabilities
   - **Mitigation**: Redis authentication, network isolation, job signing

---

## Existing Security Controls

### Preventive Controls

1. **Tenant Immutability**
   - **Control**: `TenantIsImmutable` exception raised when attempting to change tenant_id
   - **Location**: `model_extensions.rb` lines 100-102, 109-112
   - **Effectiveness**: High - Prevents accidental/malicious tenant changes

2. **Query Rewriting**
   - **Control**: Automatic injection of tenant_id into WHERE clauses
   - **Location**: `query_rewriter.rb`
   - **Effectiveness**: High - Enforces tenant isolation at query level

3. **Association Wrapping**
   - **Control**: Wraps association methods with `MultiTenant.with()`
   - **Location**: `model_extensions.rb` lines 170-189
   - **Effectiveness**: High - Ensures associations respect tenant context

4. **Statement Cache Disabling**
   - **Control**: Skips prepared statement caching for multi-tenant models
   - **Location**: `model_extensions.rb` lines 192-204
   - **Effectiveness**: High - Prevents cross-tenant cache poisoning

5. **Thread-Local Storage**
   - **Control**: Uses `ActiveSupport::CurrentAttributes` for tenant context
   - **Location**: `multi_tenant.rb` lines 5-8
   - **Effectiveness**: High - Prevents cross-thread contamination

6. **Tenant Context Cleanup**
   - **Control**: Ensure/rescue blocks in `with()` method
   - **Location**: `multi_tenant.rb` lines 106-116
   - **Effectiveness**: High - Prevents context leakage

### Detective Controls

7. **Query Monitoring (Optional)**
   - **Control**: `QueryMonitor` to detect unscoped queries
   - **Location**: `query_monitor.rb`
   - **Effectiveness**: Medium - Requires explicit enablement

8. **Model Registration**
   - **Control**: Registry of all multi-tenant models
   - **Location**: `multi_tenant.rb` lines 44-62
   - **Effectiveness**: Medium - Enables centralized tracking

### Response Controls

9. **Exception Handling**
   - **Control**: `MissingTenantError` and `TenantIsImmutable` exceptions
   - **Location**: `multi_tenant.rb` lines 155-159
   - **Effectiveness**: Medium - Alerts application to violations

10. **Callback-Based Enforcement**
    - **Control**: `around_save`, `around_update`, `around_destroy` callbacks
    - **Location**: `model_extensions.rb` lines 132-156
    - **Effectiveness**: High - Last-line defense for tenant context

---

## Threat Mitigation Recommendations

### Critical Priority (Immediate Action Required)

#### REC-01: Restrict MultiTenant.without() Usage
- **Threat**: E-01 (Elevation of Privilege - Critical Risk)
- **Action**:
  - Add runtime warnings when `without()` is called
  - Implement allowlist of authorized callers
  - Consider deprecating in favor of explicit super-admin tenant
  - Add audit logging for all `without()` calls
- **Effort**: Medium
- **Impact**: Critical

#### REC-02: Implement Query Monitoring by Default
- **Threat**: T-01, I-03 (Tampering, Information Disclosure - High Risk)
- **Action**:
  - Enable query monitor in development/test environments by default
  - Add configurable threshold for production warnings
  - Log all unscoped queries with stack traces
  - Add CI/CD checks for unscoped query detection
- **Effort**: High
- **Impact**: High

#### REC-03: Secure Sidekiq Tenant Class Resolution
- **Threat**: E-04 (Elevation of Privilege - Medium Risk)
- **Action**:
  - Whitelist allowed tenant class names
  - Validate tenant class inheritance
  - Add configuration option to restrict `constantize()` usage
  - Sign/encrypt Sidekiq job payloads
- **Effort**: Medium
- **Impact**: Medium

### High Priority (Within 1 Quarter)

#### REC-04: Add Tenant Context Validation Hooks
- **Threat**: S-01, E-05 (Spoofing, Elevation of Privilege - High Risk)
- **Action**:
  - Provide callback hooks for tenant context validation
  - Allow applications to register tenant authorization checks
  - Add middleware to verify tenant access on each request
- **Effort**: Medium
- **Impact**: High

#### REC-05: Implement Comprehensive Audit Logging
- **Threat**: R-01, R-02 (Repudiation - Medium Risk)
- **Action**:
  - Add optional audit logging module
  - Log tenant context changes with timestamps
  - Track `without()` usage and multi-tenant operations
  - Integrate with application audit systems
- **Effort**: High
- **Impact**: Medium

#### REC-06: Enhance Error Message Sanitization
- **Threat**: I-02 (Information Disclosure - Low Risk)
- **Action**:
  - Review all exception messages for sensitive data
  - Implement sanitization for production environments
  - Add configuration for error detail levels
- **Effort**: Low
- **Impact**: Low

### Medium Priority (Within 2 Quarters)

#### REC-07: Write-Only Mode Security Review
- **Threat**: E-02 (Elevation of Privilege - High Risk)
- **Action**:
  - Add prominent security warnings to documentation
  - Implement automatic expiration of write-only mode
  - Add metrics to detect prolonged write-only usage
  - Consider requiring explicit opt-in per environment
- **Effort**: Low
- **Impact**: Medium

#### REC-08: Fuzz Testing for Query Rewriter
- **Threat**: T-04 (Tampering - Medium Risk)
- **Action**:
  - Implement fuzz testing for Arel AST manipulation
  - Test edge cases in complex query structures
  - Add regression tests for reported issues
- **Effort**: High
- **Impact**: Medium

#### REC-09: Add Security Linting Rules
- **Threat**: Multiple (General Security Hardening)
- **Action**:
  - Create RuboCop rules for common anti-patterns
  - Detect direct SQL usage without tenant context
  - Flag `without()` calls for review
  - Identify missing tenant declarations
- **Effort**: Medium
- **Impact**: Medium

### Low Priority (Ongoing/As Needed)

#### REC-10: Security Documentation Enhancement
- **Threat**: Multiple (Awareness and Training)
- **Action**:
  - Create security best practices guide
  - Document common pitfalls and anti-patterns
  - Provide secure coding examples
  - Add security considerations to README
- **Effort**: Low
- **Impact**: Low

#### REC-11: Runtime Integrity Checks
- **Threat**: T-05 (Tampering - Low Risk)
- **Action**:
  - Implement checksums for critical methods
  - Detect monkey-patching of tenant enforcement
  - Add warnings in development mode
- **Effort**: High
- **Impact**: Low

---

## Security Testing Requirements

### Unit Tests

- [ ] Tenant immutability enforcement
- [ ] Query rewriter correctness for all query types
- [ ] Association wrapping with correct tenant context
- [ ] Exception handling for missing tenant
- [ ] Thread-local storage isolation

### Integration Tests

- [ ] End-to-end tenant isolation in full Rails app
- [ ] Controller filter integration
- [ ] Sidekiq job tenant context preservation
- [ ] Migration helper security
- [ ] Cache isolation between tenants

### Security-Specific Tests

- [ ] **Tenant Isolation**: Verify no cross-tenant data leakage
- [ ] **Bypass Attempts**: Test for ways to circumvent tenant filters
- [ ] **SQL Injection**: Parameterized queries prevent injection
- [ ] **Race Conditions**: Concurrent tenant switching
- [ ] **Privilege Escalation**: `without()` and `with()` security
- [ ] **Input Validation**: Malformed tenant_id handling

### Penetration Testing

- [ ] Attempt session fixation attacks
- [ ] Test for SQL injection in tenant_id
- [ ] Analyze Sidekiq job payload manipulation
- [ ] Verify cross-tenant data access prevention
- [ ] Test for timing attacks on tenant identification

### Dependency Scanning

- [ ] Automated CVE scanning for Rails dependencies
- [ ] Monitor security advisories for PostgreSQL
- [ ] Track Sidekiq security updates
- [ ] Review transitive dependencies

---

## Threat Model Review Cadence

### Regular Review Schedule

- **Quarterly Reviews**: Security team reviews threat model for new threats
- **Release Reviews**: Update threat model before major version releases
- **Incident-Triggered**: Immediate review after security incidents
- **Annual Audit**: Comprehensive third-party security assessment

### Triggers for Unscheduled Review

1. New major feature addition (e.g., new database support)
2. Reported security vulnerability
3. Significant architectural changes
4. New integration points (e.g., new job queue systems)
5. Regulatory compliance requirements
6. Major Rails/Ruby version upgrades

### Review Process

1. **Security Team Review**: Evaluate new threats and attack vectors
2. **Development Team Input**: Identify architectural changes
3. **External Review**: Optional third-party security audit
4. **Update Documentation**: Revise threat model and mitigation strategies
5. **Communication**: Notify users of security-relevant changes

### Stakeholders

- **Owner**: Security Team Lead
- **Contributors**: Development Team, DevOps, QA
- **Reviewers**: External Security Consultants (annual)
- **Approvers**: CTO, Security Officer

---

## References

### Security Documentation

- [Rails Security Guide](https://guides.rubyonrails.org/security.html)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [STRIDE Threat Modeling](https://docs.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats)
- [CWE Top 25](https://cwe.mitre.org/top25/)

### Related Documentation

- [README.md](./README.md) - Installation and usage guide
- [CHANGELOG.md](./CHANGELOG.md) - Version history and changes
- [CONTRIBUTING.md](./docs/source/contributing.rst) - Contribution guidelines
- [LICENSE](./LICENSE) - MIT License terms

### Security Policies

- **Vulnerability Reporting**: Report security issues to engage@citusdata.com
- **Security Updates**: Monitor [GitHub Security Advisories](https://github.com/citusdata/activerecord-multi-tenant/security/advisories)
- **Dependency Updates**: Dependabot enabled for automated security patches

### Incident Response

- **Internal Contact**: Security team via secure channels
- **External Contact**: engage@citusdata.com (PGP key available on request)
- **Response SLA**:
  - Critical: 24 hours
  - High: 72 hours
  - Medium: 1 week
  - Low: Best effort

### Compliance

- **GDPR**: Tenant isolation supports data segregation requirements
- **SOC 2**: Audit logging capabilities for compliance
- **HIPAA**: Can be configured for healthcare data isolation

---

## Appendix: Common Attack Scenarios

### Scenario 1: Session Hijacking Leading to Tenant Access

**Attack Chain**:
1. Attacker steals session cookie via XSS
2. Uses cookie to authenticate as victim user
3. Application sets tenant based on compromised session
4. Attacker accesses victim tenant's data

**Countermeasures**:
- Application-level session security (HttpOnly, Secure, SameSite)
- Short session timeouts
- IP binding (with caution for mobile users)
- MFA implementation

### Scenario 2: SQL Injection in Tenant Identification

**Attack Chain**:
1. Application uses unsanitized input to determine tenant
2. Attacker injects SQL in tenant identifier
3. Bypass tenant filtering via SQL injection
4. Access cross-tenant data

**Countermeasures**:
- Never use user input directly for tenant_id
- Always use parameterized queries
- Validate tenant_id format (integer/UUID)
- Use ORM methods, not raw SQL

### Scenario 3: Background Job Privilege Escalation

**Attack Chain**:
1. Attacker gains access to Redis/job queue
2. Modifies Sidekiq job payload to include admin tenant_id
3. Job processes with elevated tenant privileges
4. Performs unauthorized actions

**Countermeasures**:
- Redis authentication and network isolation
- Sign/encrypt job payloads
- Validate tenant exists and user has access
- Job-level authorization checks

### Scenario 4: Race Condition in Concurrent Requests

**Attack Chain**:
1. User makes simultaneous requests for different tenants
2. Race condition in tenant context switching
3. Query executed with wrong tenant context
4. Brief data leakage window

**Countermeasures**:
- Thread-local storage (already implemented)
- Comprehensive concurrency testing
- Request ID tracking and correlation
- Audit logs for debugging

---

**Document Classification**: Internal Use
**Distribution**: Security Team, Development Team, DevOps
**Next Review Date**: 2024-04-19 (Quarterly)

---

*This threat model is a living document and should be updated as the system evolves. All team members are encouraged to contribute potential threats and mitigation strategies.*
