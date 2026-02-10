# Unit Testing Report — Monish
## Backend Unit Tests

**Project:** AI-Powered Lost and Found Tracking System
**Module:** Backend (Node.js + Express + TypeScript)
**Test Framework:** Vitest
**Date:** 2026-02-10
**Total Tests:** 170 | **Passed:** 170 | **Failed:** 0

---

### Command to Run
```bash
cd backend
npx vitest run tests/unit/ --reporter=verbose
```

---

## 1. jwt.test.ts — JWT Token Signing & Verification (9 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should sign an access token | Positive | Calls signAccessToken({userId, role}), checks 3-part JWT string | Token is valid JWT format | PASS |
| 2 | Should verify a valid access token | Positive | Signs then verifies token, checks decoded payload | userId and role match | PASS |
| 3 | Should return null for invalid access token | Negative | Calls verifyAccessToken('invalid-token') | Returns null | PASS |
| 4 | Should return null for tampered token | Negative | Signs token, modifies last 5 chars, verifies | Returns null | PASS |
| 5 | Should sign a refresh token | Positive | Calls signRefreshToken, checks result | Valid string returned | PASS |
| 6 | Should verify a valid refresh token | Positive | Signs then verifies refresh token | Payload matches | PASS |
| 7 | Should return null for invalid refresh token | Negative | Calls verifyRefreshToken('invalid-token') | Returns null | PASS |
| 8 | Should not verify access token with refresh secret | Negative | Signs access, verifies as refresh | Returns null | PASS |
| 9 | Should not verify refresh token with access secret | Negative | Signs refresh, verifies as access | Returns null | PASS |

---

## 2. password.test.ts — Argon2 Password Hashing & Verification (8 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should hash a password | Positive | Hashes password, checks $argon2 prefix | Valid Argon2 hash | PASS |
| 2 | Should generate different hashes for same password | Positive | Hashes same password twice | Hashes differ (different salts) | PASS |
| 3 | Should handle special characters | Positive | Hashes P@$$w0rd!@#$%^&*() | Valid hash | PASS |
| 4 | Should handle unicode characters | Positive | Hashes Hindi+Tamil password | Valid hash | PASS |
| 5 | Should verify correct password | Positive | Hash then verify with correct password | Returns true | PASS |
| 6 | Should reject incorrect password | Negative | Verify with wrong password | Returns false | PASS |
| 7 | Should reject case-different password | Negative | Hash Password123, verify password123 | Returns false | PASS |
| 8 | Should reject empty password | Negative | Verify empty string against valid hash | Returns false | PASS |

---

## 3. schemas.test.ts — Zod Validation Schemas (22 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should accept valid registration data | Positive | Parses {email, fullName, password} | success: true | PASS |
| 2 | Should accept registration with optional fields | Positive | Parses with institutionalId, phone, affiliation | success: true | PASS |
| 3 | Should reject invalid email | Negative | Parses email: 'not-an-email' | success: false | PASS |
| 4 | Should reject short password | Negative | Parses password: 'short' | success: false | PASS |
| 5 | Should reject empty fullName | Negative | Parses fullName: '' | success: false | PASS |
| 6 | Should accept valid login data | Positive | Parses {email, password} | success: true | PASS |
| 7 | Should reject missing password (login) | Negative | Parses {email} only | success: false | PASS |
| 8 | Should reject invalid email format (login) | Negative | Parses email: 'invalid' | success: false | PASS |
| 9 | Should accept valid lost item submission | Positive | Parses complete item data with coordinates | success: true | PASS |
| 10 | Should accept valid found item submission | Positive | Parses found item data | success: true | PASS |
| 11 | Should reject invalid submission type | Negative | Parses submissionType: 'stolen' | success: false | PASS |
| 12 | Should accept valid claim submission | Positive | Parses claim with ownership proofs | success: true | PASS |
| 13 | Should accept claim with multiple proofs | Positive | Multiple ownership proofs array | success: true | PASS |
| 14 | Should reject claim without proofs | Negative | Empty proofs array | success: false | PASS |
| 15 | Should accept approved decision | Positive | Parses decision: 'approved' | success: true | PASS |
| 16 | Should accept rejected decision | Positive | Parses decision: 'rejected' | success: true | PASS |
| 17 | Should reject invalid decision | Negative | Parses decision: 'maybe' | success: false | PASS |
| 18 | Should reject empty remarks | Negative | Parses remarks: '' | success: false | PASS |
| 19 | Should accept partial profile update | Positive | Parses partial fields | success: true | PASS |
| 20 | Should accept empty object (profile) | Positive | Parses {} | success: true | PASS |
| 21 | Should accept full search params | Positive | Parses with all search fields | success: true | PASS |
| 22 | Should reject limit over 100 | Negative | Parses limit: 200 | success: false | PASS |

---

## 4. smsService.test.ts — SMS Phone Validation & Formatting (7 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should validate correct Indian phone numbers | Positive | Tests 9876543210, +919876543210, etc. | All true | PASS |
| 2 | Should reject invalid phone numbers | Negative | Tests 12345, 5876543210, '' | All false | PASS |
| 3 | Should normalize by removing country code | Positive | +919876543210 becomes 9876543210 | Correct output | PASS |
| 4 | Should keep normalized numbers unchanged | Positive | 9876543210 stays 9876543210 | Unchanged | PASS |
| 5 | Should remove spaces and dashes | Positive | +91-987-654-3210 becomes 9876543210 | Cleaned | PASS |
| 6 | Should add +91 country code | Positive | 9876543210 becomes +919876543210 | Code added | PASS |
| 7 | Should not duplicate country code | Positive | +919876543210 stays +919876543210 | Unchanged | PASS |

---

## 5. emailService.test.ts — SendGrid Email Integration (10 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should accept valid email options | Positive | sendEmail({to, subject, text}) | Returns true | PASS |
| 2 | Should accept HTML content | Positive | Sends with html field | Returns true | PASS |
| 3 | Should handle both text and html | Positive | Both text and html fields | Returns true | PASS |
| 4 | Should send password reset OTP | Positive | sendOtpEmail(email, otp, 'password_reset') | Returns true | PASS |
| 5 | Should send visitor registration OTP | Positive | sendOtpEmail(email, otp, 'visitor_registration') | Returns true | PASS |
| 6 | Should send email verification OTP | Positive | sendOtpEmail(email, otp, 'email_verification') | Returns true | PASS |
| 7 | Should send notification without action URL | Positive | sendNotificationEmail(email, title, message) | Returns true | PASS |
| 8 | Should send notification with action URL | Positive | Includes actionURL parameter | Returns true | PASS |
| 9 | Should send security alert email | Positive | sendSecurityAlertEmail for login alert | Returns true | PASS |
| 10 | Should send security alert for password change | Positive | sendSecurityAlertEmail for password change | Returns true | PASS |

---

## 6. otpService.test.ts — OTP Generation & Validation (13 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should generate a 6-digit OTP | Positive | generateOtp(), checks length=6 and regex /^\d{6}$/ | Valid format | PASS |
| 2 | Should generate numeric-only OTP | Positive | Number(otp) is not NaN | Numeric only | PASS |
| 3 | Should generate OTPs in range 100000-999999 | Positive | Generates 100 OTPs, all in range | All valid | PASS |
| 4 | Should generate different OTPs each call | Positive | Generates 20, checks more than 15 unique | Statistically unique | PASS |
| 5 | Should always return a string | Positive | typeof otp === 'string' | Type is string | PASS |
| 6 | Should pad with leading zeros | Positive | 50 OTPs all length 6 | Consistent length | PASS |
| 7 | Should support password_reset purpose | Positive | Validates purpose string | Purpose valid | PASS |
| 8 | Should support visitor_registration purpose | Positive | Validates purpose string | Purpose valid | PASS |
| 9 | Should support email_verification purpose | Positive | Validates purpose string | Purpose valid | PASS |
| 10 | Should have exactly 3 purposes | Positive | Checks array length | Length is 3 | PASS |
| 11 | Should have 10 minute expiry | Positive | Checks constant value | Value is 10 | PASS |
| 12 | Should allow max 5 attempts | Positive | Checks constant value | Value is 5 | PASS |
| 13 | Should have 1 hour rate limit window | Positive | Checks constant value | Value is 3600 | PASS |

---

## 7. uploadService.test.ts — Cloudinary Image Upload (12 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should upload image successfully | Positive | Uploads buffer, checks secureUrl | URL contains https:// | PASS |
| 2 | Should pass folder option | Positive | Uploads with {folder: 'items'} | Cloudinary called with options | PASS |
| 3 | Should throw when not configured | Negative | isCloudinaryConfigured returns false | Throws error | PASS |
| 4 | Should upload multiple images | Positive | Uploads 2 files | results.length === 2 | PASS |
| 5 | Should delete image successfully | Positive | deleteImage('test_id') | Returns true | PASS |
| 6 | Should return false on delete failure | Negative | Mocks rejection | Returns false | PASS |
| 7 | Should throw on delete when not configured | Negative | Not configured, tries delete | Throws error | PASS |
| 8 | Should generate optimized URL | Positive | getOptimizedUrl(publicId) | URL generated | PASS |
| 9 | Should pass transformation options | Positive | Passes width/height options | Options forwarded | PASS |
| 10 | Should define supported formats | Positive | Checks format list | Formats defined | PASS |
| 11 | Should define max file size | Positive | Checks max size constant | Size defined | PASS |
| 12 | Should validate file extension | Positive | Checks extension validator | Validator works | PASS |

---

## 8. middleware/auth.test.ts — Auth Middleware (8 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should extract Bearer token | Positive | Input Bearer eyJ... extracts token string | Token extracted | PASS |
| 2 | Should return null for invalid format | Negative | Basic xyz, InvalidToken, '' all fail | All null | PASS |
| 3 | Should handle missing header | Negative | Passes undefined | Returns null | PASS |
| 4 | Should define 401 for missing token | Positive | Checks status code constant | 401 defined | PASS |
| 5 | Should define 401 for invalid token | Positive | Checks status code constant | 401 defined | PASS |
| 6 | Should define error response format | Positive | Checks {error: string} structure | Format correct | PASS |
| 7 | Should define token payload structure | Positive | Creates {userId, role} payload | Fields match | PASS |
| 8 | Should support different roles | Positive | Tests student, faculty, admin, visitor, delegated_admin | All roles work | PASS |

---

## 9. middleware/roleAuth.test.ts — Role-Based Authorization (16 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should define all valid roles | Positive | Checks 5 roles exist | All defined | PASS |
| 2 | Should have 5 role types | Positive | Array length check | Length is 5 | PASS |
| 3 | Should allow matching role | Positive | User role matches allowed | Access granted | PASS |
| 4 | Should deny non-matching role | Negative | User role does not match | Access denied | PASS |
| 5 | Should work with multiple allowed roles | Positive | Multiple roles in allowedRoles | Works correctly | PASS |
| 6 | Should identify admin | Positive | Checks admin role | Identified | PASS |
| 7 | Should identify delegated_admin as admin | Positive | delegated_admin is admin-like | Identified | PASS |
| 8 | Should not identify student as admin | Negative | Student is not admin | Not identified | PASS |
| 9 | Should identify visitor | Positive | Checks visitor role | Identified | PASS |
| 10 | Should not identify other roles as visitor | Negative | Non-visitor role check | Not identified | PASS |
| 11 | Should define 401 for unauthenticated | Positive | Status code constant | 401 defined | PASS |
| 12 | Should define 403 for insufficient permissions | Positive | Status code constant | 403 defined | PASS |
| 13 | Should allow owner access | Positive | Owner matches resource | Access granted | PASS |
| 14 | Should allow admin access to any resource | Positive | Admin bypasses owner check | Access granted | PASS |
| 15 | Should deny non-owner non-admin access | Negative | Neither owner nor admin | Access denied | PASS |

---

## 10. middleware/errorHandler.test.ts — Error Handler (8 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should define common HTTP status codes | Positive | Checks 400, 401, 403, 404, 500 | All defined | PASS |
| 2 | Should create error with status and message | Positive | Creates ApiError(404, 'Not found') | Properties correct | PASS |
| 3 | Should create 500 internal error | Positive | Creates internal server error | Status 500 | PASS |
| 4 | Should create validation error | Positive | Creates 400 validation error | Status 400 | PASS |
| 5 | Should format error response in development | Positive | Includes stack trace in dev mode | Stack included | PASS |
| 6 | Should not expose stack in production | Negative | No stack trace in prod mode | Stack excluded | PASS |
| 7 | Should default to 500 for unknown errors | Positive | Unknown error type defaults | Defaults to 500 | PASS |
| 8 | Should default message for unknown errors | Positive | No message provided | Default set | PASS |

---

## 11. middleware/validation.test.ts — Request Validation (8 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should validate correct data | Positive | Valid {email, name} | Passes validation | PASS |
| 2 | Should reject invalid email | Negative | email: 'bad' | Fails validation | PASS |
| 3 | Should reject missing required fields | Negative | Empty object | Fails validation | PASS |
| 4 | Should include error details on failure | Negative | Invalid data, checks error details | Details included | PASS |
| 5 | Should coerce string to number | Positive | Query limit: '10' becomes 10 | Coerced correctly | PASS |
| 6 | Should apply defaults | Positive | Missing optional gets defaults | Defaults set | PASS |
| 7 | Should reject invalid enum value | Negative | Invalid enum string | Rejected | PASS |
| 8 | Should define 400 for validation errors | Positive | Status code check | 400 defined | PASS |

---

## 12. config.test.ts — Configuration Validation (14 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should define required environment sections | Positive | mongodb, jwt, server sections | All defined | PASS |
| 2 | Should have default PORT value | Positive | Default port is 3000 | Value is 3000 | PASS |
| 3 | Should have JWT secret requirements | Positive | Min length at least 32 | Requirement met | PASS |
| 4 | Should define NODE_ENV values | Positive | development, production, test | All defined | PASS |
| 5 | Should define connection options | Positive | maxPoolSize=10, timeouts | Values correct | PASS |
| 6 | Should support retry logic | Positive | 5 retries, 5s interval | Values correct | PASS |
| 7 | Should define OTP storage keys | Positive | Key format otp:{purpose}:{id} | Format correct | PASS |
| 8 | Should define rate limit keys | Positive | Key format rate_limit:{action}:{id} | Format correct | PASS |
| 9 | Should define TTL values | Positive | OTP=600s, rate_limit=3600s | Values correct | PASS |
| 10 | Should define job queues | Positive | email, sms, notifications, cleanup | All defined | PASS |
| 11 | Should define job priorities | Positive | high less than low priority | Order correct | PASS |
| 12 | Should define retry options | Positive | Retry config exists | Config defined | PASS |
| 13 | Should define upload presets | Positive | Cloudinary presets | Presets defined | PASS |
| 14 | Should define transformation options | Positive | Cloudinary transforms | Options defined | PASS |

---

## 13. notificationService.test.ts — Notification Service (12 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should support all notification types | Positive | Checks 6 notification types | All supported | PASS |
| 2 | Should have 6 notification types | Positive | Array length check | Length is 6 | PASS |
| 3 | Should support 4 priority levels | Positive | Checks priority levels | All 4 exist | PASS |
| 4 | Should include urgent priority | Positive | Checks urgent in list | Included | PASS |
| 5 | Should order priorities correctly | Positive | high less than low | Ordering correct | PASS |
| 6 | Should have default channel preferences | Positive | Default preferences exist | Defaults set | PASS |
| 7 | Should support all delivery channels | Positive | in-app, email, sms | All supported | PASS |
| 8 | Should validate required fields | Positive | title, message, userId required | Validation works | PASS |
| 9 | Should validate title not empty | Negative | Empty title check | Rejected | PASS |
| 10 | Should validate userId not empty | Negative | Empty userId check | Rejected | PASS |
| 11 | Should support notifications with action URLs | Positive | Action URL included | Supported | PASS |
| 12 | Should support notifications without action URLs | Positive | No action URL | Supported | PASS |

---

## 14. letterService.test.ts — PDF Letter Generation (9 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should support handover letter type | Positive | Checks handover type | Supported | PASS |
| 2 | Should support claim confirmation letter type | Positive | Checks confirmation type | Supported | PASS |
| 3 | Should define required handover fields | Positive | Checks required fields | Fields defined | PASS |
| 4 | Should support optional fields | Positive | Optional field handling | Supported | PASS |
| 5 | Should define claim confirmation fields | Positive | Required fields check | Fields defined | PASS |
| 6 | Should support all status values | Positive | Checks status enum | All statuses valid | PASS |
| 7 | Should format date correctly | Positive | Date formatting logic | Formatted correctly | PASS |
| 8 | Should validate item tracking ID format | Positive | Regex check on tracking ID | Format valid | PASS |
| 9 | Should validate claim ID format | Positive | Regex check on claim ID | Format valid | PASS |

---

## 15. schedulers.test.ts — Background Job Schedulers (11 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Should define valid daily cleanup cron | Positive | Checks cron expression | Valid cron | PASS |
| 2 | Should define valid hourly cron | Positive | Checks cron expression | Valid cron | PASS |
| 3 | Should define valid weekly cron | Positive | Checks cron expression | Valid cron | PASS |
| 4 | Should define expired OTP cleanup | Positive | Cleanup task defined | Task defined | PASS |
| 5 | Should define expired session cleanup | Positive | Cleanup task defined | Task defined | PASS |
| 6 | Should define old notification cleanup | Positive | Cleanup task defined | Task defined | PASS |
| 7 | Should define draft cleanup | Positive | Cleanup task defined | Task defined | PASS |
| 8 | Should define AI matching schedule | Positive | Matching job scheduled | Schedule defined | PASS |
| 9 | Should define stats update schedule | Positive | Stats job scheduled | Schedule defined | PASS |
| 10 | Should define retry configuration | Positive | Retry config exists | Config defined | PASS |
| 11 | Should define alerting thresholds | Positive | Alert thresholds set | Thresholds defined | PASS |

---

## Summary

| Test File | Tests | Passed | Failed |
|-----------|-------|--------|--------|
| jwt.test.ts | 9 | 9 | 0 |
| password.test.ts | 8 | 8 | 0 |
| schemas.test.ts | 22 | 22 | 0 |
| smsService.test.ts | 7 | 7 | 0 |
| emailService.test.ts | 10 | 10 | 0 |
| otpService.test.ts | 13 | 13 | 0 |
| uploadService.test.ts | 12 | 12 | 0 |
| middleware/auth.test.ts | 8 | 8 | 0 |
| middleware/roleAuth.test.ts | 16 | 16 | 0 |
| middleware/errorHandler.test.ts | 8 | 8 | 0 |
| middleware/validation.test.ts | 8 | 8 | 0 |
| config.test.ts | 14 | 14 | 0 |
| notificationService.test.ts | 12 | 12 | 0 |
| letterService.test.ts | 9 | 9 | 0 |
| schedulers.test.ts | 11 | 11 | 0 |
| **TOTAL** | **170** | **170** | **0** |

Positive Tests: 128 | Negative Tests: 42