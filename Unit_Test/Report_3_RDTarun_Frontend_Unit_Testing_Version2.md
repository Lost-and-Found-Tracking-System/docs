# Unit Testing Report — RD Tarun
## Frontend Unit Tests: Notifications, Claims, Profile & Admin Pages

**Project:** AI-Powered Lost and Found Tracking System
**Module:** Frontend (React + Vite)
**Test Framework:** Vitest + React Testing Library
**Date:** 2026-02-10
**Total Tests:** 28 | **Passed:** 28 | **Failed:** 0

---

### Command to Run
```bash
cd frontend
npx vitest run src/test/Unit_Testing/Notifications.test.jsx src/test/Unit_Testing/MyClaims.test.jsx src/test/Unit_Testing/Profile.test.jsx src/test/Unit_Testing/AdminClaims.test.jsx src/test/Unit_Testing/admin/ --reporter=verbose
```

---

## 1. Notifications.test.jsx — Notification Center (10 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Renders sidebar | Positive | Checks sidebar data-testid | Sidebar found | PASS |
| 2 | Renders Notifications header | Positive | Checks "Notifications" heading | Heading found | PASS |
| 3 | Shows unread count | Positive | Checks GlitchText data-testid displays unread count | Count displayed | PASS |
| 4 | Fetches notifications on mount | Positive | Asserts api.get('/v1/notifications') called | API called | PASS |
| 5 | Renders All, Unread, and Read filter buttons | Positive | Checks all 3 filter pill texts | All 3 found | PASS |
| 6 | Shows correct count on filter pills | Positive | Waits for data load, checks count badges | Count badges found | PASS |
| 7 | Displays notification titles | Positive | Checks "Claim Approved", "New Item Reported", "Claim Rejected" | All 3 titles displayed | PASS |
| 8 | Displays notification messages | Positive | Checks full message text | Message found | PASS |
| 9 | Shows empty state when no notifications | Negative | Mocks empty array response | "No notifications" shown | PASS |
| 10 | Calls API when Mark all read is clicked | Positive | Clicks "Mark all read", asserts api.put('/v1/notifications/read-all') | API called | PASS |

---

## 2. MyClaims.test.jsx — Claims Management (9 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Renders My Claims header | Positive | Checks "My Claims" heading | Heading found | PASS |
| 2 | Fetches and displays claims | Positive | Asserts api.get('/v1/claims/user/my-claims'), checks "Electronics" and "Books" | Claims displayed | PASS |
| 3 | Displays claim status badges | Positive | Checks "Pending Review" and "Contact admin for pickup" | Badges shown | PASS |
| 4 | Renders all filter options | Positive | Checks "All Claims" and "Pending" pills | Filters found | PASS |
| 5 | Filters claims when filter is clicked | Positive | Checks filter buttons are interactive | Filter exists | PASS |
| 6 | Shows empty message when no claims | Negative | Mocks empty array | "No claims found" shown | PASS |
| 7 | Shows Browse Items button in empty state | Negative | Mocks empty array | "Browse Items" CTA shown | PASS |
| 8 | Renders View Item button for each claim | Positive | Checks 2 "View Item" buttons (1 per claim) | 2 buttons found | PASS |
| 9 | Shows pickup message for approved claims | Positive | Checks "Contact admin for pickup" | Message shown | PASS |

---

## 3. Profile.test.jsx — User Profile (5 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Renders sidebar | Positive | Checks sidebar testid | Sidebar found | PASS |
| 2 | Displays user full name via GlitchText | Positive | Checks GlitchText contains "John Doe" | Name displayed | PASS |
| 3 | Displays user email | Positive | Checks "john@amrita.edu" text | Email shown | PASS |
| 4 | Displays user role via NeonText | Positive | Checks NeonText contains "student" | Role shown | PASS |
| 5 | Fetches user stats on mount | Positive | Asserts api.get('/v1/dashboard/stats') called | API called | PASS |

---

## 4. AdminClaims.test.jsx — Admin Claims & Reports Management (4 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Renders admin sidebar | Positive | Checks admin sidebar component | Sidebar found | PASS |
| 2 | Renders tab buttons for Claims and Reports | Positive | Checks "Claims" and "Reports" tabs | Both tabs found | PASS |
| 3 | Shows Item Reports tab by default | Positive | Checks default active tab | Reports tab active | PASS |
| 4 | Switches to Claims tab when clicked | Positive | Clicks Claims tab, checks data loads | Claims data displayed | PASS |

---

## Summary

| Test File | Tests | Passed | Failed |
|-----------|-------|--------|--------|
| Notifications.test.jsx | 10 | 10 | 0 |
| MyClaims.test.jsx | 9 | 9 | 0 |
| Profile.test.jsx | 5 | 5 | 0 |
| AdminClaims.test.jsx | 4 | 4 | 0 |
| **TOTAL** | **28** | **28** | **0** |

Positive Tests: 25 | Negative Tests: 3