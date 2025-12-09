# HALO Product Requirements Document - Phase 3
## EU Launch + HALO Track + Geofencing + Global Compliance

**Company:** HALO (halo-wear.com)  
**Product:** HALO Lytt + HALO Track (Comprehensive family safety ecosystem)  
**Version:** 3.0  
**Status:** Approved for Engineering  
**Last Updated:** December 8, 2025  
**Author:** Product Team  
**Target Launch:** August 15, 2026 (Track) + September 15, 2026 (EU)

---

## Phase 3 Principle

**Compliance standardization, ecosystem integration, and AI-ready architecture.** Phase 3 unifies HALO Lytt and HALO Track into a comprehensive family safety platform with 10,000+ total devices globally, GDPR compliance globally, geofencing for location safety, and a data warehouse foundation for machine learning algorithms. Build the foundation for future intelligence while ensuring global regulatory compliance.

---

## Executive Summary

**Phase 3 Goals:**
1. Launch HALO Track GPS + cellular wearable device
2. Add geofencing with Zones (entry/exit alerts)
3. Expand to EU market (Ireland AWS region, GDPR-compliant)
4. Implement global GDPR compliance
5. Build data warehouse foundation for AI algorithms
6. Support multi-device per persona
7. Scale to 10,000+ devices globally

**Key Metrics (Global Ecosystem):**
- Track Public Launch: August 15, 2026
- EU Launch: September 15, 2026
- Phase 1 Devices (ME): 2,000 units
- Phase 2 Devices (US): 5,000 units
- Phase 3 Devices (EU): +3,000 units
- Total Global: 10,000+ devices by Sep 30, 2026
- Total Families: 1,500-2,500 globally
- Geofencing accuracy: >98% (entry/exit detection)
- Global MRR: ~5-10k from paid roles

**In Scope (Phase 3 Additions):**
- HALO Track hardware support (GPS + cellular, wearable pendant)
- Track location API + ingestion (real-time + historical)
- Geofencing: Zones (CRUD, entry/exit, alerts)
- Map UI (live location, history, heatmap)
- EU data residency (Ireland AWS region, GDPR-compliant)
- GDPR full implementation (user rights, DPA, breach notification)
- Multi-device per persona UI (Lytt + Track simultaneously)
- Data warehouse foundation (Aurora exports, S3 data lake, query layer)

**Phase 1 & 2 Features Continue (Unchanged):**
- All HALO Lytt vitals, Ghost Mode, alerts, offline mode
- BLE pairing + all enhancements
- Health Scoring v1.0 MVP + activity-linked location tracking
- Monetization (paid Admin/Viewer)
- Multi-language Arabic + English RTL
- COPPA compliance (US)
- Multi-region ops (ME + US + EU)

**Explicitly Out of Scope (Phase 3):**
- Snowflake integration
- School/partner integrations
- Multi-language beyond AR/EN
- Telemedicine integration
- Predictive algorithms (Phase 4)

---

## Product Vision & Scope

### 3.1 Global Device Distribution

| Region | Phase 1 | Phase 2 | Phase 3 | Phase 3 Total |
|--------|---------|---------|---------|---------------|
| Middle East | 2,000 | - | - | 2,000 |
| USA | - | 5,000 | - | 5,000 |
| EU | - | - | 3,000+ | 3,000+ |
| **GLOBAL TOTAL** | 2,000 | 5,000 | 3,000+ | 10,000+ |

### 3.2 HALO Track Hardware

**Device Type:** GPS + Cellular wearable pendant

| Spec | Details |
|------|---------| 
| Form Factor | Pendant (child-friendly, 50g) |
| GPS | A-GPS (assisted), accuracy +/- 10m typical |
| Cellular | 4G LTE + fallback to 2G |
| Battery | 72-96hr (location sync every 10 min) |
| Charging | USB-C, 1hr charge |
| Water Resistance | IP67 (splash-proof) |
| Compatibility | iOS 15+, Android 11+ |

### 3.3 EU Market Requirements

- **Compliance:** GDPR full implementation
- **Data Residency:** Ireland AWS region only
- **Languages:** English (multi-language Phase 3.5)
- **Regulations:** EDPB guidelines, ePrivacy Directive
- **Time Zone:** Europe/Dublin
- **Currency:** EUR pricing

---

## Functional Requirements

### 4.1 HALO Track Hardware Integration (P0)

**FR-TRACK-001:** Track device pairing via BLE (P0).
- Track discovered via BLE scan (special UUID)
- One-tap pairing: stores device credentials
- Pairing persists across app restarts
- Track firmware version tracked in DB

**FR-TRACK-002:** Real-time location sync (P0).
- Track sends GPS coordinates every 10 minutes
- Coordinates include: latitude, longitude, accuracy, timestamp
- Sync via cellular LTE (fallback to 2G)
- Failed syncs queued on device, retry on next window
- Max 7 days offline storage on Track

**FR-TRACK-003:** Location history retention (P0).
- Free tier: last 30 days of locations
- Paid tier: last 90 days of locations
- Query >retention window returns error + upsell
- Archival to S3 data lake after retention

**FR-TRACK-004:** Battery status (P0).
- App displays Track battery % in real-time
- Low battery alert when <20%
- Battery data synced with location updates

**FR-TRACK-005:** Device handover for Track (P1).
- Owner/Admin can factory reset Track (remote wipe)
- Reset clears all data on device
- Track becomes unassigned, ready for re-pairing

---

### 4.2 Location Tracking API (P0)

**FR-LOCATION-001:** Real-time location ingestion (P0).
- POST /locations/ingest receives device_id, lat, lon, accuracy, source: 'TRACK', timestamp
- Coordinates stored in locations hypertable (TimescaleDB)
- Geospatial index created (PostGIS)
- Returns 200 on success, 400 on validation error

**FR-LOCATION-002:** Historical location queries (P0).
- GET /locations/{persona_id}?days=7 returns last 7 days
- Default return: last 24h (10-min intervals = 144 points)
- Respects retention limits (30-day free, 90-day paid)
- Returns GeoJSON format

**FR-LOCATION-003:** Last-known location (P0).
- GET /locations/{persona_id}/latest returns most recent GPS point
- Includes: latitude, longitude, timestamp, accuracy, address (reverse-geocoded)

---

### 4.3 Geofencing & Zones (P0)

**FR-GEOFENCE-001:** Zone CRUD (P0).
- POST /zones creates zone: family_id, name, center_lat, center_lon, radius_meters
- GET /zones/{family_id} lists all zones
- PATCH /zones/{zone_id} updates zone
- DELETE /zones/{zone_id} deletes zone
- Max 50 zones per family

**FR-GEOFENCE-002:** Entry/exit detection (P0).
- Backend calculates distance from Track location to zone center
- Hysteresis (50m) prevents boundary jitter
- OUTSIDE-INSIDE = ENTRY event
- INSIDE-OUTSIDE = EXIT event

**FR-GEOFENCE-003:** Geofence alerts (P0).
- Zone owner configures: ENTRY, EXIT, or BOTH
- ENTRY alert: "Child arrived at [Zone Name]"
- EXIT alert: "Child left [Zone Name]"
- Push + email notification (per preference)

**FR-GEOFENCE-004:** Zone history (P0).
- GET /zones/{zone_id}/history returns entry/exit events
- Includes: event_type (ENTRY/EXIT), timestamp, accuracy
- History searchable by date range
- Max 30 days retained

---

### 4.4 Global GDPR Compliance (P0)

**FR-GDPR-001:** User right to access data (P0).
- POST /gdpr/export initiates data export (ZIP file)
- Includes: user profile, family data, vitals (60/90/365 days), locations, alerts
- Exported within 24 hours (async job)
- Email with download link sent to user (link valid 7 days)

**FR-GDPR-002:** User right to delete data (P0).
- DELETE /users/{user_id} initiates deletion
- 30-day grace period (soft delete)
- After 30 days: permanent deletion (no recovery)
- Dependent accounts: parent must approve deletion
- Audit log preserved

**FR-GDPR-003:** User right to rectify data (P0).
- User can edit profile: email, name, phone, DOB
- Changes logged in audit trail (old to new)
- No deletion within rectification (update only)

**FR-GDPR-004:** Data Processing Agreement (P0).
- DPA template available at signup (all regions)
- User must accept DPA before using service
- DPA includes: data processing activities, sub-processors, retention
- Acceptance timestamp logged in DB

**FR-GDPR-005:** Breach notification process (P0).
- Breach detected - internal escalation within 24h
- Legal + IT notification process
- Authorities notified within 72h (if risk to rights)
- Affected users notified with details + remediation
- Breach log maintained (GDPR Article 33)

**FR-GDPR-006:** Retention policies enforced (P0).
- Vitals: 60-day free, 90-day paid, 365-day max archived
- Locations: 30-day free, 90-day paid
- Alerts: 90 days
- Users deleted: data purged after 30-day grace
- Breach logs: 5 years (regulatory requirement)

---

### 4.5 Data Warehouse Foundation (P0)

**FR-WAREHOUSE-001:** Data export pipeline (P0).
- Aurora operational DB - S3 data lake (daily ETL)
- Nightly exports: de-identified vitals + locations + alerts
- Partitioned by region + date for query efficiency
- Retention: compliant with GDPR (auto-delete honored)

**FR-WAREHOUSE-002:** Query layer (P0).
- SQL query capability on S3 data lake (Athena-ready)
- Support for: vital trends, location patterns, alert frequency
- De-identification: PII stripped, pseudo-anonymized IDs only
- Performance: queries <30s for 90-day datasets

**FR-WAREHOUSE-003:** Audit & compliance logging (P0).
- All data warehouse queries logged
- Purpose: "user requested export", "fraud investigation", etc.
- Access control: authorized personnel only (Data Science team)
- Retention: 5 years (regulatory)

---

## Architecture & Infrastructure

### 5.1 Three-Region Architecture (Global)

```
HALO Lytt + Track (BLE + GPS)
        |
        v
React Native App (region detection)
        |
        v
Region-Specific API Gateway + Endpoints
    |           |           |
    v           v           v
ME Users     US Users      EU Users
api-me      api-us        api-eu
    |           |           |
    v           v           v
AWS Dubai   AWS Virginia  AWS Ireland
(me-south)  (us-east-1)   (eu-west-1)
    |           |           |
    +-----+-----+-----+-----+
          |
          v
Global Data Warehouse (Central)
- Aurora (operational - S3 ETL)
- S3 Data Lake (partitioned by region + date)
- Athena Query Layer (ML-ready)
```

### 5.2 Geospatial Database (PostGIS)

- PostGIS Extension: Enabled on all RDS instances
- Spatial Indexes: GIST/BRIN on coordinates (locations table)
- Queries: ST_Distance, ST_Contains, ST_DWithin <500ms
- Hypertable: Locations remain TimescaleDB hypertable (time-series + spatial)

---

## Database Schema Changes

### 6.1 New Tables (Phase 3)

#### zones

```sql
CREATE TABLE zones (
  zone_id UUID PRIMARY KEY,
  family_id UUID NOT NULL REFERENCES families(family_id),
  name VARCHAR(100) NOT NULL,
  center_point GEOGRAPHY(POINT, 4326) NOT NULL,
  radius_meters INT NOT NULL,
  alert_on_entry BOOLEAN DEFAULT TRUE,
  alert_on_exit BOOLEAN DEFAULT TRUE,
  created_by UUID NOT NULL REFERENCES users(user_id),
  created_at TIMESTAMP,
  updated_at TIMESTAMP,
  deleted_at TIMESTAMP NULL,
  INDEX idx_family_id (family_id),
  SPATIAL INDEX idx_center_point (center_point)
);
```

#### zone_events

```sql
CREATE TABLE zone_events (
  event_id BIGSERIAL PRIMARY KEY,
  zone_id UUID NOT NULL REFERENCES zones(zone_id),
  persona_id UUID NOT NULL REFERENCES users(user_id),
  event_type ENUM('ENTRY', 'EXIT') NOT NULL,
  triggered_at TIMESTAMP NOT NULL,
  alert_sent_at TIMESTAMP NULL,
  INDEX idx_zone_persona (zone_id, persona_id),
  INDEX idx_triggered_at (triggered_at DESC)
);
```

#### gdpr_exports

```sql
CREATE TABLE gdpr_exports (
  export_id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(user_id),
  status ENUM('PENDING', 'PROCESSING', 'COMPLETED', 'ERROR') DEFAULT 'PENDING',
  s3_file_path VARCHAR(500) NULL,
  download_link VARCHAR(500) NULL,
  created_at TIMESTAMP,
  completed_at TIMESTAMP NULL,
  expires_at TIMESTAMP,
  error_message VARCHAR(500) NULL,
  INDEX idx_user_id (user_id),
  INDEX idx_status (status)
);
```

#### gdpr_deletions

```sql
CREATE TABLE gdpr_deletions (
  deletion_id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(user_id),
  requested_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  soft_deleted_at TIMESTAMP,
  permanent_deleted_at TIMESTAMP NULL,
  deletion_status ENUM('REQUESTED', 'GRACE_PERIOD', 'COMPLETED') DEFAULT 'REQUESTED',
  grace_period_expires_at TIMESTAMP,
  INDEX idx_user_id (user_id),
  INDEX idx_grace_period_expires_at (grace_period_expires_at)
);
```

#### breach_log

```sql
CREATE TABLE breach_log (
  breach_id UUID PRIMARY KEY,
  reported_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  description TEXT,
  affected_users INT,
  severity ENUM('LOW', 'MEDIUM', 'HIGH', 'CRITICAL') NOT NULL,
  authorities_notified BOOLEAN DEFAULT FALSE,
  users_notified BOOLEAN DEFAULT FALSE,
  authorities_notified_at TIMESTAMP NULL,
  created_at TIMESTAMP,
  INDEX idx_reported_at (reported_at)
);
```

---

## Non-Functional Requirements

### 7.1 Performance

- API Latency (p95): <2 seconds for multi-region
- Geospatial Query: <500ms for zone entry/exit detection
- Location Ingestion: <10s from Track - alert sent
- Map Rendering: <3s for 30-day history (100+ points)
- Multi-Region Failover: <5 min RTO per region

### 7.2 Reliability & Uptime

- Target Availability: 99% uptime per region
- RTO: <5 minutes per region
- RPO: <1 minute data loss tolerance
- Geofence Accuracy: >98% (entry/exit detection)

### 7.3 Scalability

- Concurrent Users: 10,000+ devices globally
  - ME: 2,000 devices
  - US: 5,000 devices
  - EU: 3,000+ devices
- Location Ingest: 10k Track locations/min
- Geofence Queries: 1k zone evaluations/sec
- Data Warehouse: Support Phase 4+ ML algorithms

---

## Success Metrics

| Metric | Target | Definition |
|--------|--------|------------|
| **App Crash Rate** | <0.1% | CloudWatch + Firebase Crashlytics |
| **Uptime (All Regions)** | 99% | ME + US + EU combined |
| **Geofence Accuracy** | >98% | Entry/exit detection accuracy |
| **Location Sync Success** | >95% | Track locations ingested |
| **GDPR Compliance** | 100% | All user rights operational |
| **Devices Shipped (Phase 3)** | 3,000+ units | EU expansion |
| **Families Onboarded (Phase 3)** | 1,500-2,500 | Global combined |

---

## Timeline & Milestones

### Critical Path

```
Jun 1: Phase 2 Monetization live
Jun 1 - Jul 15: Phase 3 development (Track + Geofencing + GDPR)
Jul 15: Beta testing begins (Track + geofencing)
Aug 1: GDPR compliance audit
Aug 15: Track GA LAUNCH (Track app update)
Aug 15 - Sep 15: EU launch preparation
Sep 15: EU GA LAUNCH + GDPR live
```

---

## Risk Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------| 
| Track hardware issues | Medium | High | Extended beta testing by Jul 15 |
| Geofencing accuracy | Medium | High | Load test 100+ zones by Jul 30 |
| GDPR audit findings | Medium | High | Legal + product audit by Aug 1 |
| Data warehouse performance | Medium | Medium | Query optimization by Aug 10 |
| EU regulatory delays | Low | High | Legal coordination by Jun 1 |

---

## Launch Checklist

**Before Track GA (Aug 15):**
- Track hardware finalized + tested
- BLE + cellular + GPS functional
- Location ingestion pipeline working
- Battery status tracking accurate
- Cloud sync reliable (7-day offline buffer)
- Track app update submitted to stores
- Beta testing with 50+ Track devices
- <0.1% crash rate in beta
- >95% location sync success

**Before EU GA (Sep 15):**
- GDPR compliance fully implemented
- All 3 regions operational (ME + US + EU)
- Geofencing tested (50+ zones)
- Data warehouse live + queries working
- Multi-device UI tested (Lytt + Track together)
- EU App Store/Play Store approved
- Legal review completed (GDPR + ePrivacy)
- EU data center transfers validated
- 3,000 device inventory ready
- EU shipping logistics validated

---

## Changelog

| Version | Date | Changes |
|---------|------|---------| 
| 3.0 | Dec 8, 2025 | Phase 3 PRD with HALO Track hardware, geofencing, GDPR compliance, global 3-region architecture, data warehouse foundation |

---

**Document Owner:** Product Team  
**Last Reviewed:** December 8, 2025  
**Next Review:** June 1, 2026 (post-Phase 2 GA)

---

*This is the roadmap for global expansion to EU with Track hardware, geofencing, GDPR compliance, and ML-ready data warehouse.*
