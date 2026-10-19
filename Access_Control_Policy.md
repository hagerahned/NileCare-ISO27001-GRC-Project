# NileCare – Access Control Policy

| Field | Value |
|---|---|
| Document ID | NC-POL-001 |
| Version | 1.0 (draft) |
| Owner | IT Manager |
| Approved by | General Manager (fictional) |
| Review cycle | Yearly |
| ISO 27001:2022 controls | A.5.15, A.5.16, A.5.17, A.5.18, A.8.2, A.8.3, A.8.5 |

> Fictional policy created for a GRC practice project.

## 1. Purpose
Make sure only authorized people can access NileCare systems and patient data, and only to the level their job needs.

## 2. Scope
All employees, contractors and vendors who use NileCare systems: EMR, patient database, lab and imaging systems, email, file server, Active Directory, VPN, and the booking portal back office.

## 3. Principles
- **Least privilege:** give the minimum access needed for the role.
- **Need to know:** patient records are visible only to staff involved in the patient's care or billing.
- **Individual accountability:** every action must be traceable to one person.
- **Deny by default:** access is not granted unless approved.

## 4. Policy Statements

### 4.1 User accounts
1. Every user has a unique account linked to an HR record. Shared and generic accounts are **not allowed**.
2. Accounts are created only after a request from the line manager and approval from the system owner. The requester cannot approve their own request.
3. Accounts for leavers are disabled **on the last working day**; accounts unused for 90 days are disabled.

### 4.2 Authentication
1. Passwords: minimum 12 characters, no reuse of the last 10, no sharing.
2. **Multi-factor authentication (MFA) is mandatory** for email, VPN, remote access to the EMR, and all administrator accounts.
3. Default and vendor passwords must be changed before systems go live.

### 4.3 Role-based access
1. Access is granted through roles (e.g. Doctor, Nurse, Receptionist, Lab Technician, Billing, IT Admin), not to individuals.
2. Role definitions are owned by the Head of Medical Operations and approved by the system owner.
3. Access to patient records is logged. Unusual access (e.g. viewing many records, viewing records of colleagues) is reviewed.

### 4.4 Privileged access
1. Administrators use a **separate admin account**; admin accounts are not used for email or browsing.
2. Admin access is limited to named IT staff, approved by the IT Manager, and reviewed every quarter.
3. Admin actions on servers, AD and firewalls are logged.

### 4.5 Access reviews
1. System owners review user access **every quarter** and confirm or remove it in writing.
2. Role changes (transfers, promotions) trigger an immediate access update.

### 4.6 Third parties
Vendor access is time-limited, uses named accounts with MFA, and is approved and logged by IT.

## 5. Roles and Responsibilities
| Role | Responsibility |
|---|---|
| IT Manager | Owns this policy; ensures technical enforcement |
| System owners | Approve access; perform quarterly reviews |
| HR | Notify IT of joiners, movers and leavers |
| Line managers | Request and justify access for their staff |
| All users | Protect credentials; report suspected compromise |

## 6. Compliance
Breaches may lead to disciplinary action. Exceptions need written approval from the IT Manager and are recorded with an expiry date.

## 7. Related Risks
R-02 (unauthorized access to patient records), R-03 (credential theft), R-09 (privileged account compromise).
