# HALO Product Requirements Document - Phase 1
## Middle East Launch

**Company:** HALO (halo-wear.com)  
**Product:** HALO Lytt (Health wearable app MVP)  
**Version:** 1.1  
**Status:** Approved for Engineering  
**Last Updated:** December 8, 2025  
**Author:** Product Team  
**Target Launch:** February 17, 2026 (ME GA)

---

## Phase 1 Principle

**Build for regional and product scalability from day one.** All infrastructure architected with parameterized regions and multi-device support so expansion to US (Phase 2), EU (Phase 3), and future device types requires configuration changes—not rewrites. Phase 1 focuses on Middle East market with Dubai-based servers and multi-country shipping.

---

## Executive Summary

**Phase 1 Goals:**
1. Launch HALO Lytt app to Middle East market (UAE, KSA, Bahrain, Qatar, Oman, Kuwait)
2. Ship 2,000 HALO Lytt wearable devices across Middle East
3. Build scalable architecture for US expansion (Phase 2: +5,000 devices)
4. Achieve <0.1% crash rate, >95% device sync success
5. Establish Dubai data center as operational hub

**Key Metrics:**
- ME Launch: February 17, 2026
- Target Devices (Phase 1): 2,000 units shipped by March 31, 2026
- Target Families (Phase 1): 300-500 families onboarded
- Target Phase 2 Devices: +5,000 (USA expansion)
- Crash Rate: <0.1%
- Device Sync Success: >95%
- API Latency (p95): <2 seconds

**In Scope (Phase 1):**
- HALO Lytt wearable device (HR, SpO2, Temp, Stress, HRV)
- React Native mobile app (iOS + Android)
- Family graph (Owner, Admin, Members, Dependents, Viewers)
- Vital tracking, display, and alerts
- Activity-Linked Location (GPS during activities + history with routes)
- Health Scoring v1.0 (Sleep/Activity/Stress scores - MVP formula)
- Offline mode (7-day buffer, sync on reconnect)
- Arabic + English UI (RTL support)
- Dubai AWS backend (multi-AZ failover)
- Multi-country shipping from Dubai warehouse
- Explicit role consent logging (compliance)
- BLE Pairing Enhancements (manual button, refresh, retry logic, notifications)
- Ghost Mode (moved to P1 - deferred)

**Out of Scope (Phase 1):**
- HALO Track hardware
- Map UI (text-only coordinates Phase 1, maps Phase 2)
- Monetization / paid roles
- Snowflake integration
- Multi-language beyond Arabic/English
- Custom alert thresholds
- Predictive algorithms

---

## Product Vision & Scope

### 2.1 Middle East Market

**Target Countries:** UAE, KSA, Bahrain, Qatar, Oman, Kuwait  
**Shipping Hub:** Dubai (all devices shipped from Dubai warehouse)  
**Servers:** Dubai AWS region (me-south-1) for all data residency  
**Languages:** Arabic (primary) + English (secondary), full RTL support  
**Time Zone:** Gulf Standard Time (GST, UTC+4)  
**Compliance:** Local data protection laws + regional privacy standards

### 2.2 Device Targets

| Milestone | Device Count | Timeline | Notes |
|-----------|--------------|----------|-------|
| **Phase 1 GA** | 500 units | Feb 17, 2026 | Launch shipment |
| **Phase 1 End** | 2,000 units | Mar 31, 2026 | 6 weeks post-launch |
| **Phase 2 (USA)** | +5,000 units | Jun 30, 2026 | US expansion |
| **Total by Phase 2** | 7,000 units | Jun 30, 2026 | Combined ME + US |

---

## Functional Requirements

### 3.1 Authentication (4 P0 + 1 P1)

**FR-AUTH-001:** Email/password signup (P0).
- User enters email + password at signup
- Confirmation email sent with 24-hour link
- Account activated on email confirmation
- Works offline: JWT cached locally

**FR-AUTH-002:** Social login (P0).
- OAuth 2.0 support: Google, Apple, Facebook
- One-tap signup for returning users
- Email verified through OAuth provider
- Already covers regional requirements

**FR-AUTH-003:** Password reset (P0).
- Email-based reset link, valid 24 hours
- User sets new password, auto-login after reset
- Rate limit: 3 requests/hour per user

**FR-AUTH-004:** JWT sessions (P0).
- Token issued on login, expires after 24 hours
- Refresh token for automatic renewal
- Logout clears cached JWT

**FR-AUTH-005:** Rate limiting (P1).
- Max 100 requests/min per user
- Returns 429 Too Many Requests when exceeded

---

### 3.2 Family Graph & Roles (6 P0)

**FR-FAMILY-001:** Owner role (P0).
- Creates family, one per family
- Can designate admins, remove members
- Full administrative control

**FR-FAMILY-002:** Admin role (P0).
- 1 free per family in Phase 1
- Can add/remove members
- Approves Ghost Mode requests
- Manages alerts + notifications

**FR-FAMILY-003:** Member role (P0).
- Read-only access to family vitals
- Can view assigned dependents' data
- Unlimited per family

**FR-FAMILY-004:** Dependent member role (P0).
- Person being monitored (wears HALO Lytt)
- Can request Ghost Mode
- Unlimited per family

**FR-FAMILY-005:** Viewer role (P0).
- Limited read-only access to vitals
- No admin capabilities
- Unlimited per family

**FR-FAMILY-006:** Explicit role consent (P0).
- Each role requires signed digital consent
- Consent timestamp logged in DB
- Audit trail for compliance
- Applies to all users, all countries (UAE, KSA, Bahrain, Qatar, Oman, Kuwait)
- Multi-country shipping context: Consent captures user's country of residence

---

### 3.3 Device Management (6 P0 + 3 P1)

**FR-DEVICE-001:** BLE auto-pairing (P0).
- HALO Lytt discovered via Bluetooth Low Energy
- One-tap pairing (AirPods-style UI)
- Credentials stored securely on phone

**FR-DEVICE-001a:** Manual Pairing Button (P0).
- User can manually enter device pairing code
- Backup method if auto-discovery fails
- Code displayed on HALO Lytt screen
- Fallback for unreliable BLE scenarios

**FR-DEVICE-001b:** Pull-to-Refresh Sync (P0).
- Pull-down gesture refreshes BLE connection
- Triggers fresh device scan + vitals sync
- Standard mobile UX pattern
- Works in offline mode with cached data

**FR-DEVICE-001c:** Auto-Retry Logic (P0).
- Pairing failures retry automatically (3 attempts)
- Exponential backoff (1s, 2s, 4s)
- Prevents timeout on first failure
- Improves emerging market reliability

**FR-DEVICE-001d:** Failure Notifications (P0).
- Alert user after multiple pairing failures
- Suggests troubleshooting steps
- Allows manual retry or device reset
- Reduces user frustration

**FR-DEVICE-002:** Device assignment (P0).
- Link HALO Lytt to dependent member persona
- Device.assigned_persona_id tracked in DB
- Required before vitals ingestion starts

**FR-DEVICE-003:** Battery tracking (P1).
- Display device battery % on dashboard
- Low-battery alert when <20%

**FR-DEVICE-004:** Factory reset (P1).
- Remote wipe for device handover
- Device becomes unassigned, ready for re-pairing

**FR-DEVICE-005:** Firmware OTA (P1).
- Automatic firmware updates via BLE
- Rollback on failure

---

### 3.4 Vitals Tracking & Display (5 P0 + 2 P1)

**FR-VITALS-001:** Collect 5 vitals (P0).
- HR, SpO2, Temperature, Stress, HRV collected every 5 minutes
- Device streams via BLE to app
- Sensor validation for accuracy

**FR-VITALS-002:** Vitals ingestion (P0).
- Backend receives vitals via SQS async processing
- Ensures scalability for 1,000+ devices
- No request blocking

**FR-VITALS-003:** 60-day retention (P0).
- Historical data stored in TimescaleDB
- Automatic archival after 60 days
- Compliance + analytics ready

**FR-VITALS-004:** Real-time dashboard (P0).
- Display current vitals: HR, SpO2, Temp, Stress, HRV
- Min/max/avg stats for 24h window
- Live updates (refresh every 10s)

**FR-VITALS-005:** Basic charts (P0).
- Simple line graphs for each vital type
- 24-hour default view
- Renders on dashboard

**FR-VITALS-006:** Advanced charts (P1).
- Date range picker, multi-view
- Export to CSV

**FR-VITALS-007:** Load target (P1).
- Validate 1,000 concurrent devices

---

### 3.5 Health Scoring Engine (4 P0)

**FR-HEALTH-001:** Sleep Score Calculation (P0).
- Based on HRV patterns + sleep duration
- Scale: 1-10
- Calculation: Consistency of HRV during sleep + total sleep hours
- Formula: (HRV_consistency_score + sleep_hours_factor) / 2

**FR-HEALTH-002:** Stress Score Calculation (P0).
- Based on stress level readings + HRV variability
- Scale: 1-10
- Inverse correlation: Lower HRV consistency = higher stress
- Formula: 10 - (avg_stress_reading / 10) - (HRV_variance_factor)

**FR-HEALTH-003:** Activity Score Calculation (P0).
- Based on movement patterns + HR intensity
- Scale: 1-10
- Calculation: Movement count + average HR during movement
- Formula: (movement_intensity + HR_elevation_factor) / 2

**FR-HEALTH-004:** Final Health Score (P0).
- Composite score from Sleep + Activity + Stress
- Scale: 1-10 (decimal to 1 place)
- Simple MVP formula: (Sleep + Activity + Stress) / 3 rounded to 1 decimal
- Example: (7 + 6 + 8) / 3 = 7.0/10
- Display: "Your Overall Health Score: 7.0/10"

**FR-HEALTH-005:** Percentile Ranking (P1).
- "You're in 95th percentile of health"
- Requires 30+ days of historical data
- Deferred to Phase 1.5

---

### 3.6 Ghost Mode (4 P0 → P1)

**FR-GHOST-001:** Request Privacy (P1).
- Dependent initiates Ghost Mode request
- Includes optional duration (hours/days)
- Notification sent to admin

**FR-GHOST-002:** Approval Flow (P1).
- Admin receives in-app notification + push
- Can approve with time limit or deny
- Dependent notified of decision

**FR-GHOST-003:** Hide Vitals (P1).
- During Ghost Mode: vitals not visible to family
- Cached locally on app
- Sync queued for after expiry

**FR-GHOST-004:** Retroactive Alerts (P1).
- Alerts always sent (override Ghost Mode)
- Emergency access for family safety
- Logged in audit trail

---

### 3.7 Alerts System (5 P0)

**FR-ALERT-001:** Heart Rate Alert (P0).
- HR >120 bpm for 2+ min triggers alert
- Hardcoded threshold
- Sent to all family members

**FR-ALERT-002:** SpO2 Alert (P0).
- SpO2 <92% for 1+ min triggers alert
- Hardcoded threshold
- Sent to all family members

**FR-ALERT-003:** Temperature Alert (P0).
- Temp >38°C for 1+ min triggers alert
- Hardcoded threshold
- Sent to all family members

**FR-ALERT-004:** Stress Alert (P0).
- Stress level >80 for 5+ min triggers alert
- HRV-derived, hardcoded
- Sent to all family members

**FR-ALERT-005:** Push Notifications (P0).
- iOS (APNs) delivery
- Android (FCM) delivery
- Multi-platform support
- Delivery target: <10 seconds

---

### 3.8 Activity-Linked Location Tracking (3 P0)

**FR-LOCATION-001:** Activity-Linked GPS Tracking (P0).
- Track phone GPS DURING recorded activities (walks, workouts)
- Activity type detected by HALO Lytt hardware (accelerometer + HR logic)
- GPS waypoints collected every 30 seconds
- Waypoints linked to activity_id in backend
- Location from phone GPS (not device GPS)
- Synced to backend with vitals + activity data

**FR-LOCATION-002:** Activity History with Location (P0).
- Show past activities with location route
- Display format: "5km walk | Dec 8, 3:30-4:17 PM | Route: [GPS coords] | Calories: 245"
- Queryable by date range
- Includes metrics: distance, duration, avg HR, calories
- Can view activity map (text coordinates Phase 1, visual maps Phase 2)

**FR-LOCATION-003:** Text-Only Location Display (P0).
- Show location as GPS coordinates (text format)
- Example: "40.7128 N, 74.0060 W"
- No map UI in Phase 1

---

### 3.9 Offline Mode (5 P0)

**FR-OFFLINE-001:** SQLite local cache (P0).
- Buffer vitals for 7 days when offline
- Stores: vitals, timestamps, device_id, persona_id
- Automatic cleanup after sync

**FR-OFFLINE-002:** Sync on reconnect (P0).
- Auto-sync queued vitals when internet restored
- Syncs in background without blocking UI
- Data recovery + no data loss

**FR-OFFLINE-003:** Offline banner (P0).
- Yellow "No Connection" banner at top of app
- User transparency + awareness

**FR-OFFLINE-004:** Manual refresh (P0).
- User-triggered sync button in settings
- User agency in offline mode
- Force sync without waiting for auto-detection

**FR-OFFLINE-005:** Deduplication (P0).
- Prevent duplicate vitals post-sync
- Check device_id + timestamp before insert
- Data integrity maintained

---

### 3.10 Push Notifications (3 P0)

**FR-PUSH-001:** iOS Integration (P0).
- APNs (Apple Push Notification service) setup
- Device token management
- Silent push support for vitals

**FR-PUSH-002:** Android Integration (P0).
- FCM (Firebase Cloud Messaging) setup
- Device token management
- Multi-language support

**FR-PUSH-003:** Notification Triggers (P0).
- Alerts (all 4: HR, SpO2, Temp, Stress)
- Ghost Mode requests
- Family invites
- Device connectivity changes
- Target delivery: <10 seconds

---

### 3.11 Localization & Multi-Language (2 P0)

**FR-LOCALE-001:** Arabic + English (P0).
- Full UI translation to both languages
- Launched in Arabic (primary market preference)
- English as secondary option

**FR-LOCALE-002:** RTL Support (P0).
- Full right-to-left layout for Arabic
- Text alignment, icon placement, form flow
- RTL CSS + i18n framework

---

### 3.12 Infrastructure (3 P0)

**FR-INFRA-001:** GitHub Actions (P0).
- Automated test + build on every push
- Runs unit tests, linting, build
- Blocks merge on test failure

**FR-INFRA-002:** AWS Dubai Region (P0).
- RDS PostgreSQL + TimescaleDB (time-series vitals)
- Lambda (stateless API)
- SQS (async vitals processing)
- Redis (caching)
- S3 (file storage, backups)
- Multi-AZ within region for failover

**FR-INFRA-003:** PostgreSQL + TimescaleDB (P0).
- Time-series optimized database
- Hypertable for vitals (automatic partitioning by time)
- Optimized queries for date-range searches
- PostGIS extension for location queries (Phase 2+)

---

## Priority Distribution

### P0 Features (46 - Launch Critical)

**Authentication (4):**
1. Email/Password Signup
2. Social Login (Google/Apple/Facebook)
3. Password Reset
4. JWT Sessions

**Family Graph (6):**
1. Owner Role
2. Admin Role (1 Free)
3. Member Role
4. Dependent Member Role
5. Viewer Role
6. Explicit Role Consent

**Device Management (6):**
1. BLE Auto-Pairing
2. Manual Pairing Button
3. Pull-to-Refresh Sync
4. Auto-Retry Logic
5. Failure Notifications
6. Device Assignment

**Vitals (5):**
1. Collect 5 Vitals
2. Vitals Ingestion
3. 60-Day Retention
4. Real-Time Dashboard
5. Basic Charts

**Health Scoring (4):**
1. Sleep Score Calculation
2. Stress Score Calculation
3. Activity Score Calculation
4. Final Health Score

**Activity-Linked Location (3):**
1. Activity GPS Tracking
2. Activity History with Location
3. Text-Only Display

**Alerts (5):**
1. Heart Rate Alert
2. SpO2 Alert
3. Temperature Alert
4. Stress Alert
5. Push Notifications

**Offline (5):**
1. SQLite Local Cache
2. Sync on Reconnect
3. Offline Banner
4. Manual Refresh
5. Deduplication

**Push (3):**
1. iOS Integration
2. Android Integration
3. Notification Triggers

**Localization (2):**
1. Arabic + English
2. RTL Support

**Infrastructure (3):**
1. GitHub Actions
2. AWS Dubai Region
3. PostgreSQL + TimescaleDB

---

### P1 Features (26 - Can Defer)

**Authentication (1):**
1. Rate Limiting

**Device (3):**
1. Battery Tracking
2. Factory Reset
3. Firmware OTA

**Vitals (3):**
1. Advanced Charts
2. Load Testing
3. Percentile Ranking

**Ghost Mode (6):**
1. Request Privacy
2. Approval Flow
3. Hide Vitals
4. Retroactive Alerts
5. Extension Request
6. Audit Log

**Alerts (2):**
1. Alert History
2. Acknowledge/Dismiss

**Location (2):**
1. General Location Logging
2. Location Privacy Settings

**Push (2):**
1. User Preferences
2. SMS/Email Alternatives

**Localization (2):**
1. Date Formats
2. Arabic Numerals

**Account (4):**
1. Profile Editing
2. Account Deletion
3. Data Export
4. Settings

**Infrastructure (2):**
1. Load Testing
2. Monitoring

---

## Feature Delivery by Sprint

### Sprint 0-1 (Dec 22 - Jan 5)
**Focus: Auth + Family Graph + Device Pairing + Core Vitals**

- Email/Social auth
- Family roles (all 6) + consent
- BLE pairing + manual button + all enhancements
- BLE auto-retry + failures
- Vitals ingestion (5 types)
- Dashboard (real-time)
- Push notifications (iOS/Android)
- Offline mode (cache + sync)
- Multi-language (AR/EN) + RTL
- AWS Dubai setup + GitHub Actions

### Sprint 2 (Jan 5-25)
**Focus: Health Scoring + Activity-Linked Location + Alerts**

- Health Scoring v1.0 (MVP)
- Activity GPS tracking
- Activity history display
- All alert thresholds (4)
- Pull-to-refresh + manual refresh
- Basic charts (line graphs)
- App Store/Play Store prep

### Sprint 3 (Jan 25 - Feb 17)
**Focus: Polish + QA + Beta + Launch**

- Bug fixes + stability
- Internal QA
- Beta testing (20-50 devices)
- GA launch (App Store/Play)

---

## Success Metrics

| Metric | Target | Definition |
|--------|--------|------------|
| **App Crash Rate** | <0.1% | CloudWatch + Firebase Crashlytics |
| **Device Sync Success** | >95% | Vitals successfully ingested to backend |
| **API Latency (p95)** | <2 seconds | Dashboard queries |
| **Startup Time** | <3 seconds | App launch to home screen |
| **Unit Test Coverage** | 80%+ | Critical path + auth + vitals |
| **Uptime** | 99.5%+ | Dubai region availability |
| **Critical Bugs at GA** | <5 | Severity: High or Critical |
| **User Adoption (Beta)** | 20-50 devices | Active testers before GA |
| **Devices Shipped (GA)** | 500 units | Feb 17, 2026 |
| **Devices Shipped (Phase 1)** | 2,000 units | Mar 31, 2026 |
| **Families Onboarded** | 300-500 | By Mar 31, 2026 |

---

## Architecture Readiness for Phase 2 + 3

### Regional Parameterization (Built-In Phase 1)

- API endpoint parameter: api-me.halo.app (Dubai) - Phase 2 adds api-us.halo.app (N. Virginia)
- Database region field: user.region = 'ME' - Phase 2 adds 'US'
- Environment variables: Region-specific config - No code changes for new regions
- Multi-AZ within region: Failover ready for Phase 2 expansion
- Consent system: Global-ready - Scales to COPPA (US Phase 2) + GDPR (EU Phase 3)

### Device Type Extensibility (Built-In Phase 1)

- Device table: device_type = 'HALO_LYTT' - Phase 3 adds device_type = 'HALO_TRACK'
- Location tracking: Text-only (phone GPS) - Phase 3 adds Track GPS + cellular
- Multi-device per persona: Architecture supports multiple devices per user
- Vitals schema: Generic enough for Track metrics (geo + vitals combined)

### Health Scoring Foundation (Built-In Phase 1)

- v1.0 MVP: Simple average formula (Sleep + Activity + Stress / 3)
- Phase 1.5: Add percentile ranking with historical data
- Phase 2: Enhance with user segmentation (age, region, gender)
- Phase 3: ML-ready data warehouse for Phase 4+ algorithms

---

## Risk Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------| 
| Hardware delays | Medium | High | 2nd vendor PO by Dec 5, 2-week buffer |
| BLE connectivity issues | Medium | High | Load test 50+ devices by Jan 15; include retry logic |
| Health scoring complexity | Low | Medium | Use simple MVP formula (avg of 3 scores) |
| API performance degradation | Medium | High | Redis caching + SQS buffering |
| App Store rejection | Medium | Medium | Legal pre-review by Jan 20 |
| Contractor leaves | Low | Medium | Backup contractor + documentation |
| Shipping delays from Dubai | Low | Medium | Warehouse stock by Jan 30 |

---

## Launch Checklist

**Before GA (Feb 17):**
- All 46 P0 features complete + tested
- BLE enhancements working
- Health Scoring v1.0 calculating correctly
- Activity-Linked Location tracking GPS during activities
- Activity history displaying routes (text coordinates)
- Beta testing with 20-50 devices (Feb 1-7)
- App Store/Play Store approved
- <0.1% crash rate in beta
- >95% device sync success in beta
- <5 critical bugs open
- Monitoring + alerting active
- Team trained on support + escalation
- Legal + privacy review signed off
- Shipping logistics ready (Dubai warehouse)
- 500 units shipped to customers (Feb 17 onwards)

**30-Day Post-Launch (Mar 17):**
- 1,000 devices shipped + operational
- <0.1% crash rate in production
- >95% device sync success in production

**End of Phase 1 (Mar 31):**
- 2,000 devices shipped + operational
- 300-500 families onboarded
- Ready for Phase 2 (US expansion) approval
- Documentation complete for Phase 2 transition

---

## Phase 2 Readiness Checklist

Once Phase 1 is complete:

- Phase 1 stability metrics confirmed (>99% uptime)
- Architecture audit: regional parameterization validated
- Consent system tested globally-ready
- Team trained on multi-region deployment
- N. Virginia AWS account provisioned
- Health Scoring foundation ready for Phase 2 enhancements
- Activity-Linked Location ready for Phase 2 map visualization
- Phase 2 PRD reviewed + approved

---

## Changelog

| Version | Date | Changes |
|---------|------|---------| 
| 1.1 | Dec 8, 2025 | BLE enhancements, Health Scoring v1.0 MVP, Activity-Linked Location, Ghost Mode deferred to P1, SMS/Email deleted, 46 P0 + 26 P1 = 72 total features, Feb 17 deadline achievable |
| 1.0 | Dec 5, 2025 | Initial Phase 1 PRD |

---

**Document Owner:** Product Team  
**Last Reviewed:** December 8, 2025  
**Next Review:** January 5, 2026 (post-beta)

---

*This is the roadmap for building Phase 1 with proper device reliability, health insights, activity tracking, and pragmatic prioritization.*
