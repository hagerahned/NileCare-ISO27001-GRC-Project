# NileCare – Information Security Incident Response Policy

| Field | Value |
|---|---|
| Document ID | NC-POL-002 |
| Version | 1.0 (draft) |
| Owner | IT Manager |
| Approved by | General Manager (fictional) |
| Review cycle | Yearly, and after every major incident |
| ISO 27001:2022 controls | A.5.24, A.5.25, A.5.26, A.5.27, A.5.28, A.6.8 |

> Fictional policy created for a GRC practice project.

## 1. Purpose
Define how NileCare detects, reports, handles and learns from information security incidents, to limit harm to patients, operations and data.

## 2. Scope
All information security events and incidents affecting NileCare systems, data, staff or suppliers.

## 3. Definitions
- **Event:** an observable occurrence that may affect security (e.g. a failed login alert).
- **Incident:** an event confirmed or likely to harm confidentiality, integrity or availability.

## 4. Reporting
1. Any employee who suspects an incident reports it **immediately** to IT (security@nilecare.example or the IT hotline).
2. Examples: phishing email clicked, lost laptop, malware alert, patient data sent to the wrong person, unexpected system behavior.
3. Reporting in good faith will not lead to punishment.

## 5. Severity Levels
| Level | Description | Example | Response target |
|---|---|---|---|
| Critical | Patient safety or mass data exposure, key systems down | Ransomware on EMR | Start within 15 minutes |
| High | Confirmed compromise of a system or sensitive data | Compromised admin account | Within 1 hour |
| Medium | Contained issue with limited impact | Malware on one workstation | Same working day |
| Low | Minor policy violation, no impact | Weak password found | Within 3 working days |

## 6. Response Phases
1. **Detect and report:** event received, logged in the incident register.
2. **Triage:** IT Manager assigns severity and an Incident Lead.
3. **Contain:** isolate affected systems (network disconnect, disable accounts); do **not** power off before evidence is preserved unless required for safety.
4. **Investigate and preserve evidence:** collect logs, memory and disk images where needed; record who handled what and when.
5. **Eradicate and recover:** remove the cause, restore from verified clean backups, confirm normal operation.
6. **Notify:** see section 7.
7. **Review:** within 10 working days of closure, hold a lessons-learned meeting and track improvements.

## 7. Notification
| Who | When |
|---|---|
| Management | Critical and High incidents: immediately |
| Data protection authority and affected patients | When personal data is breached and notification is required under Egypt PDPL (Law 151 of 2020); decision made by Legal and Compliance |
| EG-CERT / law enforcement | For serious cyber attacks, as decided by management |
| Suppliers | When their service or data is involved |

## 8. Roles
| Role | Responsibility |
|---|---|
| Incident Lead (IT Manager or delegate) | Coordinates response and decisions |
| IT team | Technical containment and recovery |
| Head of Medical Operations | Clinical continuity (paper downtime procedures) |
| Legal and Compliance | Legal and notification decisions |
| Communications (General Manager) | Statements to patients and media |

## 9. Records and Testing
- All incidents are recorded: date, description, severity, actions, root cause, lessons.
- A tabletop exercise (ransomware scenario) is performed **once a year**.

## 10. Related Risks
R-01 (ransomware), R-03 (phishing), R-06 (vendor breach), R-07 (lost laptop), R-14 (PDPL non-compliance).
