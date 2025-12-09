# HALO Product Requirements Document - Phase 2
## US Launch + Monetization

**Company:** HALO (halo-wear.com)  
**Product:** HALO Lytt (Health wearable app)  
**Version:** 2.0  
**Status:** Approved for Engineering  
**Last Updated:** December 8, 2025  
**Author:** Product Team  
**Target Launch:** May 15, 2026 (US GA) + June 1, 2026 (Monetization)

---

## Phase 2 Principle

**Multi-region operations, compliance, and stability.** Phase 2 expands HALO Lytt to the US market with 5,000 additional devices, bringing total ecosystem to 7,000 devices across Middle East + USA. Maintain Dubai stability while enforcing strict data residency, adding monetization for revenue sustainability, and implementing COPPA compliance for child safety. Build the operations foundation that enables Phase 3 global expansion.

---

## Executive Summary

**Phase 2 Goals:**
1. Scale HALO Lytt to US market (N. Virginia AWS region, us-east-1)
2. Add 5,000 devices in USA (total ecosystem: 7,000 devices = 2,000 ME + 5,000 US)
3. Add COPPA compliance for US children <13
4. Launch monetization: 1 Owner (free) + 1 Admin (free) + additional Admins/Viewers (paid)
5. Maintain Dubai + US stability across 2 regions

**Key Metrics:**
- US Launch: May 15, 2026
- Monetization Go-Live: June 1, 2026
- Phase 1 Devices (ME): 2,000 units
- Phase 2 Devices (US): +5,000 units
- Total Ecosystem: 7,000 devices by June 30, 2026
- Target Families: 800-1,200 combined (ME + US)
- Monetization conversion: 15% families upgrading to paid
- MRR from subscriptions: ~2-3k by end of June

**In Scope (Phase 2 Additions):**
- US data residency (N. Virginia region, separate from Dubai)
- COPPA compliance (verifiable parental consent email + link, age verification)
- Payment processing (Stripe + local US payment aggregators)
- Subscription backend (Additional Admin 10/mo, Viewer 5/mo)
- Paywall UI + in-app purchase + web billing flows
- Multi-region architecture (geo-fenced data, independent region failover)
- Monitoring + alerting (CloudWatch, PagerDuty-ready)
- Map visualization for Activity-Linked Location (Phase 1 GPS data)
- Health Scoring enhancement (user segmentation, Phase 1.5 percentile launch)

**Phase 1 Features Continue (Unchanged):**
- All HALO Lytt vitals, Ghost Mode, alerts, offline mode
- BLE pairing + all enhancements
- Health Scoring v1.0 MVP + activity-linked location tracking
- Family graph (Owner, 1 free Admin, Member, Dependent, Viewer)
- Arabic + English, RTL support
- Explicit role consent logging

**Explicitly Out of Scope (Phase 2):**
- HALO Track hardware or location ingestion
- Geofencing zones
- Snowflake analytics
- Multi-language beyond AR/EN

---

## Product Vision & Scope

### 2.1 US Market Requirements

- **Compliance:** COPPA (Children's Online Privacy Protection Act) for users <13
  - Verifiable parental consent required (email link, parent clicks to confirm)
  - Age verification at signup
  - Limited data collection for children <13
  - Parental access to collected data
  - Account deletion on parental request
- **Payment:** US payment methods (Stripe primary, local aggregators)
- **Regulations:** CCPA (California Privacy Rights Act) reviewed
- **Time Zone:** Supports US time zones (EST, CST, MST, PST)
- **Currency:** USD pricing

### 2.2 Monetization Strategy

| Role | Free Tier | Paid Tier | Price |
|------|-----------|-----------|-------|
| Owner | Free (unlimited) | N/A | Free |
| Admin | 1 free per family | Additional at 10/mo | 10/month or 50/year |
| Viewer | N/A | Paid only at 5/mo | 5/month or 30/year |
| Member | Free (unlimited) | N/A | Free |
| Dependent Member | Free (unlimited) | N/A | Free |

**Payment Models:**
- In-app purchase (iOS App Store, Google Play)
- Web billing (Stripe checkout, US local aggregators)
- Annual discount: 17% off

### 2.3 Device Distribution

| Region | Phase 1 | Phase 2 | Phase 2 Total |
|--------|---------|---------|---------------|
| **Middle East** | 2,000 | - | 2,000 |
| **USA** | - | 5,000 | 5,000 |
| **TOTAL** | 2,000 | 5,000 | 7,000 |

---

## Functional Requirements

### 3.1 COPPA Compliance (P0)

**FR-COPPA-001:** Age verification at signup (P0).
- User selects age at signup
- If <13: parent email field required
- If >=13: proceeds normally
- Age stored in user profile

**FR-COPPA-002:** Verifiable parental consent (P0).
- System sends email to parent with consent form
- Email includes HALO privacy policy link
- Parent clicks email link (valid 7 days)
- Parent views consent text + HALO privacy policy
- Parent clicks "I Agree" to activate child account
- Consent timestamp + parent email + IP stored in parental_consents table
- Audit trail preserved for FTC compliance

**FR-COPPA-003:** Limited data collection for children <13 (P0).
- Child profile limited to: name, age, DOB, vitals only
- No location tracking for children <13
- No social features (messaging, sharing)
- No third-party data sharing (Stripe only for payments, PCI-compliant)

**FR-COPPA-004:** Parental rights (P0).
- Parent can export child's vitals as CSV (COPPA right to access)
- Parent can delete child account + all data (COPPA right to delete)
- Deletion is permanent within 7 days
- Deletion request must be confirmed via email
- Child cannot login after deletion request initiated

---

### 3.2 Payment Processing (P0)

**FR-PAYMENT-001:** In-app purchase (IAP) (P0).
- Admin role IAP: 10/mo, 50/yr displayed in app
- Viewer role IAP: 5/mo, 30/yr displayed in app
- iOS IAP: uses App Store In-App Purchases, receipt validation via Apple API
- Android IAP: uses Google Play Billing, receipt validation via Google API
- Purchase receipt stored in DB with token + timestamp + amount

**FR-PAYMENT-002:** Web billing (Stripe checkout) (P0).
- User clicks "Upgrade" - redirects to Stripe hosted checkout
- Stripe returns session ID for verification
- On success: subscription created in backend, entitlement granted <1s
- Receipt emailed to user
- Web link accessible from app settings

**FR-PAYMENT-003:** Subscription renewal + cancellation (P0).
- Subscriptions auto-renew on billing date
- Renewal failures trigger retry logic (3 attempts, 3 days apart)
- User can cancel subscription (access remains through current billing period)
- Cancellation effective on next renewal date
- User can reactivate subscription before expiry

**FR-PAYMENT-004:** Local payment aggregators (P1).
- Support 1-2 local US payment aggregators (e.g., 2Checkout)
- Country/region selector for payment method
- Backend routes payment to appropriate processor
- Webhook handlers for payment confirmations

**FR-PAYMENT-005:** Payment error handling (P0).
- Declined card: user notified, can retry
- Billing failures logged for support team
- User can manually trigger renewal if it fails
- Error messages don't leak sensitive payment info

---

### 3.3 Monetization Backend (P0)

**FR-MONETIZATION-001:** Entitlement system (P0).
- Backend tracks subscription status per user
- API endpoint: GET /users/{user_id}/entitlements
- Returns: active_roles (list of paid roles), expiry dates
- Used for UI paywall gating

**FR-MONETIZATION-002:** Paywall UI (P0).
- Paid role locked UI (blur + "Upgrade" button)
- Stripe checkout link embedded
- Upgrade flow: tap role - paywall - Stripe checkout - confirmation

**FR-MONETIZATION-003:** Revenue tracking (P0).
- All transactions logged in transactions table
- Amount, currency, payment method, timestamp
- Reconciliation with Stripe webhook events

---

### 3.4 Activity-Linked Location Map Visualization (P0)

**FR-LOCATION-MAPS-001:** Visual activity route map (P0).
- Display activity GPS waypoints on map
- Leverage Phase 1 GPS data (no new collection)
- Map shows: route line, start marker, end marker, distance
- Integration: Google Maps or OpenStreetMap

**FR-LOCATION-MAPS-002:** Interactive activity timeline (P0).
- Tap activity - see route on map + stats
- Timeline view: activity date, duration, HR metrics
- Swipe to view previous/next activities

---

### 3.5 Health Scoring Enhancement (P0)

**FR-HEALTH-SEGMENTATION-001:** User segmentation (P0).
- Segment users by: age group, region, gender
- Adjust scoring baseline per segment
- Example: "Teens (13-18) have baseline 6.0; you're at 7.0"

**FR-HEALTH-PERCENTILE-001:** Percentile ranking (P0).
- Phase 1.5 data ready: 30+ days of Phase 1 users' scores
- Percentile display: "You're in 75th percentile of health"
- Requires historical data warehouse query

---

## Architecture & Infrastructure

### 4.1 Two-Region Architecture (ME + US)

```
HALO Lytt + Track (BLE + GPS)
        |
        v
React Native App (region detection)
        |
        v
Region-Specific API Gateway + Endpoints
    |               |
    v               v
ME Users          US Users
api-me.halo.app   api-us.halo.app
    |               |
    v               v
AWS Dubai         AWS N. Virginia
(me-south-1)      (us-east-1)
    |               |
    +-------+-------+
            |
            v
Regional Infrastructure (Geo-Fenced)
- API Gateway
- Lambda
- RDS PostgreSQL + TimescaleDB
- Redis
- SQS
- S3
```

### 4.2 Payment Infrastructure

- Stripe API: Payment processing, subscriptions, webhooks
- Local aggregators: 2Checkout or regional payment processors
- Database: subscriptions table with renewal tracking
- Webhook handlers: Stripe events (charge.succeeded, customer.subscription.updated)

---

## Database Schema Changes

### 5.1 New Tables (Phase 2)

#### subscriptions

```sql
CREATE TABLE subscriptions (
  subscription_id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(user_id),
  role_type ENUM('ADMIN', 'VIEWER') NOT NULL,
  billing_cycle ENUM('MONTHLY', 'ANNUAL') NOT NULL,
  stripe_subscription_id VARCHAR(100) UNIQUE,
  status ENUM('ACTIVE', 'PAUSED', 'CANCELED', 'EXPIRED') DEFAULT 'ACTIVE',
  current_period_start TIMESTAMP NOT NULL,
  current_period_end TIMESTAMP NOT NULL,
  amount_cents INT NOT NULL,
  currency VARCHAR(3) DEFAULT 'USD',
  created_at TIMESTAMP,
  updated_at TIMESTAMP,
  INDEX idx_user_id (user_id),
  INDEX idx_stripe_subscription_id (stripe_subscription_id),
  INDEX idx_status (status)
);
```

#### parental_consents

```sql
CREATE TABLE parental_consents (
  consent_id UUID PRIMARY KEY,
  child_user_id UUID NOT NULL REFERENCES users(user_id),
  parent_email VARCHAR(255) NOT NULL,
  consent_text TEXT NOT NULL,
  accepted_at TIMESTAMP,
  ip_address VARCHAR(45),
  user_agent TEXT,
  created_at TIMESTAMP,
  INDEX idx_child_user_id (child_user_id),
  INDEX idx_parent_email (parent_email),
  INDEX idx_accepted_at (accepted_at)
);
```

#### transactions

```sql
CREATE TABLE transactions (
  transaction_id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(user_id),
  subscription_id UUID REFERENCES subscriptions(subscription_id),
  amount_cents INT NOT NULL,
  currency VARCHAR(3) DEFAULT 'USD',
  status ENUM('SUCCEEDED', 'PENDING', 'FAILED', 'REFUNDED') DEFAULT 'PENDING',
  stripe_charge_id VARCHAR(100) UNIQUE,
  payment_method VARCHAR(50),
  created_at TIMESTAMP,
  updated_at TIMESTAMP,
  INDEX idx_user_id (user_id),
  INDEX idx_stripe_charge_id (stripe_charge_id),
  INDEX idx_status (status)
);
```

---

## Success Metrics

| Metric | Target | Definition |
|--------|--------|------------|
| **Monetization Conversion** | 15% | % families with paid subscriptions |
| **MRR** | 2-3k | Monthly recurring revenue by June |
| **API Latency (p95)** | <2 seconds | Multi-region queries |
| **Uptime (Combined ME + US)** | 99% | Both regions operational |
| **Payment Success Rate** | >99% | Successful transactions |
| **COPPA Compliance** | 100% | All <13 users have verified consent |
| **Devices Shipped (Phase 2)** | 5,000 units | USA expansion |
| **Families Onboarded (Phase 2)** | 800-1,200 | Combined ME + US |

---

## Timeline & Milestones

### Critical Path

```
Feb 17: Phase 1 GA (ME)
Feb 18 - Apr 1: Phase 2 development (COPPA + Monetization)
Apr 1: Beta testing begins (Stripe sandbox)
Apr 15: Payment flow validated + approved
May 1: US App Store/Play submission
May 15: US GA LAUNCH
Jun 1: Monetization goes live
```

---

## Risk Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------| 
| Payment processor delays | Medium | High | Onboard 2 processors by Mar 1 |
| Stripe regulatory issues | Low | High | Legal review by Feb 1 |
| COPPA compliance gaps | Medium | High | Legal + product audit by Apr 1 |
| Regional API latency | Medium | Medium | Load test with cross-region data |
| Payment fraud spikes | Medium | Medium | Fraud detection system ready |

---

## Launch Checklist

**Before US GA (May 15):**
- All Phase 1 features stable in production
- COPPA compliance fully implemented
- Payment processing tested end-to-end
- Stripe production account configured
- Monetization paywall UI tested
- Legal review completed (CCPA + COPPA)
- US App Store/Play Store approved
- Multi-region failover tested
- Monitoring + alerting active for 2 regions
- 5,000 device inventory ready
- Shipping logistics for USA validated

**Before Monetization Go-Live (Jun 1):**
- 30 days of payment transaction history analyzed
- Subscription renewal logic tested
- Refund/cancellation workflows tested
- Revenue dashboard operational
- Finance team trained on reconciliation
- Support team trained on payment issues

---

## Changelog

| Version | Date | Changes |
|---------|------|---------| 
| 2.0 | Dec 8, 2025 | Phase 2 PRD with COPPA compliance, monetization (Stripe + in-app purchase), Activity Location maps, Health Scoring enhancements, multi-region ops (ME + US) |

---

**Document Owner:** Product Team  
**Last Reviewed:** December 8, 2025  
**Next Review:** March 1, 2026 (post-Phase 1 GA)

---

*This is the roadmap for US expansion with monetization and COPPA compliance.*
