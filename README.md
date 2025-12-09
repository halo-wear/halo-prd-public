# HALO Roadmap Overview

HALO is a three-phase roadmap that launches a health and safety platform in the Middle East, then scales to the US and EU with added hardware, compliance, and intelligence features.

## Phase 1 – Middle East MVP

Phase 1 delivers the HALO Lytt wearable and mobile app for families in the Middle East. It focuses on reliable vitals monitoring, core alerts, and basic health insights. Scope includes BLE pairing, family roles, vitals collection, simple charts, offline mode, MVP scoring (Sleep, Activity, Stress, Overall Health), and activity-linked GPS during workouts. Infrastructure runs in a single region (Dubai) but is parameterized for future regions.

## Phase 2 – US + Monetization

Phase 2 adds a second region (US), introduces monetization (paid Admin/Viewer roles), and implements COPPA for children under 13. Work covers multi-region operations, Stripe and in-app purchases, subscription entitlements, and map visualization for the activity GPS data captured in Phase 1. Architecture becomes dual-region with independent failover and shared product behavior.

## Phase 3 – EU, Track, and Geofencing

Phase 3 launches the HALO Track GPS/cellular device, geofencing zones, and an EU region with full GDPR compliance. It adds real-time and historical location for Track, entry/exit alerts around defined zones, and a central data warehouse (S3 data lake plus query layer). This sets the foundation for future ML-based health and safety intelligence across the global device ecosystem.

