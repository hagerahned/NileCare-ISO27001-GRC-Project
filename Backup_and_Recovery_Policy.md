# NileCare – Backup and Recovery Policy

| Field | Value |
|---|---|
| Document ID | NC-POL-003 |
| Version | 1.0 (draft) |
| Owner | IT Manager |
| Approved by | General Manager (fictional) |
| Review cycle | Yearly |
| ISO 27001:2022 controls | A.8.13, A.5.30, A.8.14 |

> Fictional policy created for a GRC practice project.

## 1. Purpose
Make sure NileCare can restore critical data and systems after loss, corruption, ransomware or disaster.

## 2. Scope
EMR and patient database, lab and imaging systems, Active Directory, file server, email configuration, firewall and switch configurations.

## 3. Recovery Targets (assumed for this exercise)
| System | Max data loss (RPO) | Max downtime (RTO) |
|---|---|---|
| EMR and patient database | 4 hours | 8 hours |
| Lab and imaging systems | 24 hours | 24 hours |
| Active Directory | 24 hours | 8 hours |
| File server | 24 hours | 48 hours |

## 4. Policy Statements
1. **3-2-1 rule:** keep 3 copies of data, on 2 different media, with 1 copy **offline or immutable** and stored away from the production network.
2. Critical databases are backed up at least every 4 hours (transaction logs) and fully every night.
3. Backups are **encrypted** and access to backup systems is limited to named administrators with MFA.
4. Backup jobs are monitored daily; failures are fixed within one working day.
5. **Restore tests:** a sample restore is tested every quarter; a full recovery of the EMR is tested yearly. Results are recorded.
6. Retention: daily backups 30 days, monthly backups 12 months, patient records according to legal retention rules.
7. Backup media is wiped securely before disposal.

## 5. Roles
| Role | Responsibility |
|---|---|
| IT Manager | Owns policy; reviews test results |
| Backup administrators | Run and monitor backups; perform restore tests |
| System owners | Confirm recovery targets and test results |

## 6. Related Risks
R-01 (ransomware), R-04 (unrecoverable backups), R-08 (server room outage).
