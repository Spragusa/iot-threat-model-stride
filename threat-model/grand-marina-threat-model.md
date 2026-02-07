# Grand Marina – IoT Threat Model (Simulated)

## 1) Executive Summary
Hydroficient enables the Grand Marina hotel to monitor water usage in real time and remotely control water flow across key zones (guest rooms, kitchen, pool/spa). Because the system is internet-connected and managed via a cloud dashboard, it introduces security risks that could disrupt operations or cause real-world impact. If an attacker gained unauthorized access, they could shut off water to critical areas, manipulate readings to hide leaks, or disable visibility during an emergency.  
This threat model documents the system, identifies the most likely/high-impact threats using STRIDE, and recommends practical mitigations to reduce risk.

## 2) System Overview
**Environment (simulated):**
- 3 HYDROLOGIC devices (Device 01: Main Building, Device 02: Pool/Spa, Device 03: Kitchen/Laundry)
- Devices publish sensor readings (pressure, flow, gate positions) to a cloud system via MQTT
- Cloud API processes/stores data and serves a web dashboard
- Operators use the dashboard to view status and send remote commands (including emergency shutoff)

**High-value capability:** Remote shutoff / remote gate control (physical-world impact).

## 3) Assets and CIA Priorities (1–5)
| Asset | Confidentiality | Integrity | Availability | Why it matters |
|---|---:|---:|---:|---|
| Sensor readings (pressure/flow) | 2 | 5 | 4 | Bad data can hide leaks or trigger false actions |
| Command topics (gate/shutoff) | 3 | 5 | 5 | Commands directly change the physical system |
| Dashboard credentials/session | 5 | 5 | 4 | Stolen access = full control/visibility |
| MQTT broker + routing | 3 | 5 | 5 | Central path for data + commands |
| Cloud API / database | 4 | 5 | 5 | System-wide compromise impacts all zones |
| Alerting/notifications | 3 | 5 | 5 | Missed alerts = damage + downtime |

## 4) Data Flows (Text)
**Sensor path:**  
HYDROLOGIC device → publishes MQTT topic (e.g., `.../device-01/pressure/upstream`) → broker → cloud ingestion/API → database → dashboard subscription/display.

**Command path:**  
Operator dashboard action → command message created → MQTT topic (e.g., `.../commands/device-01/shutoff`) → broker → device receives → actuator executes.

## 5) STRIDE Threat Analysis (Key Components)

### Component A: Web Dashboard
| Threat | Scenario | Likelihood | Impact | Risk | Mitigation |
|---|---|---:|---:|---:|---|
| Spoofing | Stolen operator creds used to log in | High | Critical | Critical | MFA, strong password policy, phishing-resistant auth |
| Tampering | Attacker changes gate settings/shutoff | Medium | Critical | Critical | RBAC, approvals for shutoff, audit logs, confirmations |
| Repudiation | Operator denies triggering shutoff | Medium | High | High | Immutable audit trail, time-stamped logs |
| Info Disclosure | Attacker views usage patterns/controls | Medium | High | High | RBAC, least privilege, session controls |
| DoS | Dashboard/API flooded, operators locked out | Medium | High | High | WAF/rate limiting, caching/CDN, monitoring |
| Elevation | Viewer → admin via misconfig/bug | Low | Critical | High | Strict role separation, secure SDLC, periodic access reviews |

### Component B: MQTT Broker / Topic Access
| Threat | Scenario | Likelihood | Impact | Risk | Mitigation |
|---|---|---:|---:|---:|---|
| Spoofing | Fake device publishes “normal” readings | Medium | High | High | mTLS device certs, device allowlist |
| Tampering | MITM modifies messages in transit | Medium | High | High | TLS, cert validation, message signing (where feasible) |
| Repudiation | No proof who published command | Medium | Medium | Medium | Broker logging + centralized audit trail |
| Info Disclosure | Unencrypted MQTT traffic sniffed | High | High | Critical | TLS everywhere, network segmentation |
| DoS | Broker flooded, real messages delayed/dropped | Medium | High | High | Rate limits, quotas, autoscaling, alerts |
| Elevation | Read-only client publishes commands | Low | Critical | High | Separate publish/subscribe ACLs per topic |

### Component C: Cloud API / Data Storage
| Threat | Scenario | Likelihood | Impact | Risk | Mitigation |
|---|---|---:|---:|---:|---|
| Spoofing | Fake endpoint collects data | Low | High | Medium | Strict TLS + endpoint validation |
| Tampering | DB records altered (savings/alerts) | Low | High | Medium | IAM least privilege, integrity checks, backups |
| Repudiation | “We never got that alert” disputes | Medium | Medium | Medium | Delivery receipts + retention logs |
| Info Disclosure | Data breach exposes all properties | Low | Critical | High | IAM hardening, encryption at rest, key mgmt |
| DoS | Cloud outage takes down monitoring | Medium | High | High | Redundancy, graceful degradation, runbooks |
| Elevation | Over-permissive IAM role exploited | Medium | High | High | IAM audits, SCPs, least privilege, alerts |

## 6) Top Risks (What to Fix First)
1. **Dashboard account takeover** (Spoofing → full control)  
2. **Unencrypted MQTT traffic** (Info disclosure + tampering path)  
3. **Unauthorized publish to command topics** (Integrity/Availability impact)  
4. **Broker/API DoS leading to loss of monitoring/control**  
5. **Weak audit trail (repudiation) complicating incident response**

## 7) Recommended Mitigations (Practical)
**Immediate / low effort:**
- Enforce **MFA** on dashboard accounts
- Implement **RBAC** + least privilege (separate viewer/operator/admin)
- Add **rate limiting** + basic WAF protections for dashboard/API
- Turn on/verify **central logging** + alert routing and retention

**Medium effort:**
- Enforce **TLS for MQTT** and validate certificates
- Implement **topic ACLs**: separate read vs command publish permissions
- Add **command safeguards**: confirmations, approval flow for shutoff, anomaly checks

**Longer term:**
- **mTLS device identity** + device allowlists
- Resiliency: broker redundancy, fallback modes, outage runbooks
- Replay protections for commands (timestamps/nonces)

## 8) Scope / Notes
This is a simulated threat model created for portfolio purposes based on a realistic IoT water-management scenario and common IoT threat patterns. No proprietary internal data is included.

## 9) AI Usage Disclosure
This threat model was independently structured and analyzed by myself, Sam Sprague /author/. 
AI tools were used to refine language clarity, improve formatting, and validate completeness against STRIDE categories. 
All threat scenarios, risk ratings, and mitigation recommendations reflect the author's own reasoning and judgment.
