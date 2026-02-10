# Unit Testing Report — Praveen S B
## Frontend Unit Tests: Authentication, Routing & Core Pages

**Project:** AI-Powered Lost and Found Tracking System
**Module:** Frontend (React + Vite)
**Test Framework:** Vitest + React Testing Library
**Date:** 2026-02-10
**Total Tests:** 76 | **Passed:** 76 | **Failed:** 0

---

### Command to Run
```bash
cd frontend
npx vitest run src/test/Unit_Testing/App.test.jsx src/test/Unit_Testing/Login.test.jsx src/test/Unit_Testing/Register.test.jsx src/test/Unit_Testing/VisitorRegister.test.jsx src/test/Unit_Testing/LandingPage.test.jsx src/test/Unit_Testing/main.test.jsx src/test/Unit_Testing/context/ src/test/Unit_Testing/services/ --reporter=verbose
```

---

## 1. App.test.jsx — Route Guards & Application Routing (9 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Shows loading spinner when loading | Positive | Mocks useAuth with loading: true, renders ProtectedRoute, checks .animate-spin class | Spinner element in DOM | PASS |
| 2 | Redirects to login if no user | Negative | Mocks useAuth with user: null, navigates to protected route | Redirects to Login page | PASS |
| 3 | Renders children if user is authenticated | Positive | Mocks authenticated student user, renders ProtectedRoute | "Protected Content" visible | PASS |
| 4 | Redirects to dashboard if user lacks required role | Negative | Student tries accessing admin-only route (allowedRoles=['admin']) | Redirects to Dashboard | PASS |
| 5 | Redirects to dashboard if user is logged in (PublicRoute) | Negative | Logged-in student visits /login via PublicRoute | Redirects to Dashboard | PASS |
| 6 | Redirects admin to admin dashboard if logged in | Positive | Admin user visits PublicRoute | Redirects to Admin Dashboard | PASS |
| 7 | Renders children if not logged in (PublicRoute) | Positive | No user authenticated, visits PublicRoute | "Login Content" visible | PASS |
| 8 | Renders Landing Page on root / | Positive | Navigates to /, checks landing renders | "Landing Page" visible | PASS |
| 9 | Navigates to Dashboard on /dashboard | Positive | Authenticated student visits /dashboard | "Dashboard Page" visible | PASS |

---

## 2. Login.test.jsx — Login Page (11 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Renders login form with email and password inputs | Positive | Checks placeholders you@amrita.edu and password dots | Both inputs found | PASS |
| 2 | Renders Sign In button | Positive | Checks "Sign In" text exists | Button found | PASS |
| 3 | Renders link to register page | Positive | Checks "Don't have an account" text | Text found | PASS |
| 4 | Renders Welcome Back heading | Positive | Checks "Welcome Back" heading | Heading found | PASS |
| 5 | Calls login function with correct credentials | Positive | Types email + password, submits form, verifies mockLogin called with {email, password} | Login called with correct args | PASS |
| 6 | Displays error message on login failure | Negative | Mocks login rejection with "Invalid email or password" | Error message displayed | PASS |
| 7 | Shows loading state while authenticating | Positive | Mocks login as pending promise, submits form | "Authenticating..." shown | PASS |
| 8 | Toggles password visibility | Positive | Clicks eye icon, checks input type changes password to text | Type toggled | PASS |
| 9 | Has link to register page | Positive | Checks anchor has href="/register" | Correct href | PASS |
| 10 | Has link to visitor registration | Positive | Checks "Visitor" link has href="/register-visitor" | Correct href | PASS |
| 11 | Has link to home page | Positive | Checks "Back to Home" link has href="/" | Correct href | PASS |

---

## 3. Register.test.jsx — Multi-Step Registration (12 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Renders personal info form fields (Step 1) | Positive | Checks placeholders: John Doe, you@amrita.edu, +91 9876543210 | All inputs found | PASS |
| 2 | Renders Continue button | Positive | Checks "Continue" button exists | Button found | PASS |
| 3 | Shows error for empty full name | Negative | Clicks Continue without entering name | "Full name is required" shown | PASS |
| 4 | Shows error for invalid email | Negative | Types valid name + invalid-email, clicks Continue | "Please enter a valid email" shown | PASS |
| 5 | Advances to step 2 with valid input | Positive | Types valid name + email, clicks Continue | Password field (8+ characters) appears | PASS |
| 6 | Renders password fields (Step 2) | Positive | Navigates to step 2, checks both password inputs | 8+ characters and Repeat password found | PASS |
| 7 | Shows Back button | Positive | Navigates to step 2, checks "Back" exists | Button found | PASS |
| 8 | Shows error for short password | Negative | Types 3-char password, clicks Continue | "Password must be at least 8 characters" shown | PASS |
| 9 | Shows error for mismatched passwords | Negative | Types password123 / different123, clicks Continue | "Passwords do not match" shown | PASS |
| 10 | Renders role selection options (Step 3) | Positive | Completes steps 1-2, checks role options | "Student" and "Faculty" visible | PASS |
| 11 | Renders Create Account button | Positive | Completes steps 1-2, checks button | "Create Account" found | PASS |
| 12 | Calls register with correct data | Positive | Fills all 3 steps, clicks "Create Account" | register called with {fullName, email, password, role} | PASS |

---

## 4. VisitorRegister.test.jsx — Visitor OTP Registration (11 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Renders VISITOR ACCESS header | Positive | Checks "VISITOR" and "ACCESS" text | Both texts found | PASS |
| 2 | Renders subtitle | Positive | Checks "Secure Temporary Registration" | Text found | PASS |
| 3 | Renders mobile number input | Positive | Checks placeholder +91 9876543210 | Input found | PASS |
| 4 | Renders Mobile Number label | Positive | Checks "Mobile Number" label | Label found | PASS |
| 5 | Renders Send Secure OTP button | Positive | Checks OTP button text | Button found | PASS |
| 6 | Renders progress indicators | Positive | Checks 3-step progress container | Container found | PASS |
| 7 | Renders Sign In link | Positive | Checks "Already have an account" text | Text found | PASS |
| 8 | Renders Home and Student Registration links | Positive | Checks navigation links | Links found | PASS |
| 9 | Updates phone input on change | Positive | Types 9876543210, checks input value | Value is 9876543210 | PASS |
| 10 | Calls API when form is submitted | Positive | Enters phone, clicks OTP button | api.post('/v1/auth/visitor/request-otp') called | PASS |
| 11 | Displays error message on API failure | Negative | Mocks API rejection with "Phone number invalid" | Error message displayed | PASS |

---

## 5. LandingPage.test.jsx — Landing/Home Page (18 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Renders hero section | Positive | Checks hero heading text | Heading found | PASS |
| 2 | Renders hero subtitle | Positive | Checks subtitle text | Text found | PASS |
| 3 | Renders CTA buttons | Positive | Checks "Get Started" and "Learn More" buttons | Buttons found | PASS |
| 4 | Get Started links to register | Positive | Checks href="/register" | Correct href | PASS |
| 5 | Learn More links to login | Positive | Checks href="/login" | Correct href | PASS |
| 6 | Renders feature cards | Positive | Checks feature section heading | Heading found | PASS |
| 7 | Renders Smart Tracking feature | Positive | Checks feature card title | Title found | PASS |
| 8 | Renders AI Matching feature | Positive | Checks feature card title | Title found | PASS |
| 9 | Renders Secure Claims feature | Positive | Checks feature card title | Title found | PASS |
| 10 | Renders Real-time Updates feature | Positive | Checks feature card title | Title found | PASS |
| 11 | Renders How It Works section | Positive | Checks section heading | Heading found | PASS |
| 12 | Renders step 1 — Report | Positive | Checks step title | Title found | PASS |
| 13 | Renders step 2 — Match | Positive | Checks step title | Title found | PASS |
| 14 | Renders step 3 — Claim | Positive | Checks step title | Title found | PASS |
| 15 | Renders step 4 — Recover | Positive | Checks step title | Title found | PASS |
| 16 | Renders stats section | Positive | Checks stats numbers | Numbers found | PASS |
| 17 | Renders footer | Positive | Checks footer text | Footer found | PASS |
| 18 | Renders navigation bar | Positive | Checks nav links | Nav found | PASS |

---

## 6. main.test.jsx — App Entry Point (1 test)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Renders App into root element | Positive | Imports main.jsx, verifies getElementById('root') + createRoot + render called | App mounted correctly | PASS |

---

## 7. context/AuthContext.test.jsx — Authentication State Management (6 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Starts with loading true | Positive | Sets token, mocks pending API call | "Loading..." shown | PASS |
| 2 | Checks for existing token on mount | Positive | Sets valid token, mocks profile API response | "Logged in as Existing User" shown | PASS |
| 3 | Handles invalid token on mount | Negative | Sets invalid token, mocks API rejection | "Not logged in" + token cleared from localStorage | PASS |
| 4 | Handles successful login | Positive | Clicks Login, mocks API success with accessToken | "Logged in as Test User" + token stored | PASS |
| 5 | Handles login failure | Negative | Mocks API rejection, clicks Login | Remains "Not logged in" | PASS |
| 6 | Handles logout | Positive | Sets up logged-in state, clicks Logout | "Not logged in" + token cleared | PASS |

---

## 8. services/api.test.js — Axios HTTP Service & Interceptors (8 tests)

| # | Test Case | Type | Description | Expected Result | Status |
|---|-----------|------|-------------|-----------------|--------|
| 1 | Creates axios instance with correct config | Positive | Checks baseURL, withCredentials: true, timeout: 15000 | Config matches | PASS |
| 2 | Adds Authorization header if token exists | Positive | Sets token in localStorage, runs request interceptor | Bearer test-token header added | PASS |
| 3 | Does not add Authorization header if no token | Negative | No token in localStorage, runs interceptor | Header is undefined | PASS |
| 4 | Rejects on request error | Negative | Passes error to error interceptor | Promise rejects | PASS |
| 5 | Returns response data on success | Positive | Passes mock response to success interceptor | Response returned unchanged | PASS |
| 6 | Handles 401 by clearing storage and redirecting | Negative | Mocks 401 from /dashboard | Token cleared + redirect to /login | PASS |
| 7 | Does NOT redirect on 401 if on auth page | Negative | Mocks 401 while on /login page | Token NOT cleared, no redirect | PASS |
| 8 | Handles network errors | Negative | No response object in error | Error message contains "Network error" | PASS |

---

## Summary

| Test File | Tests | Passed | Failed |
|-----------|-------|--------|--------|
| App.test.jsx | 9 | 9 | 0 |
| Login.test.jsx | 11 | 11 | 0 |
| Register.test.jsx | 12 | 12 | 0 |
| VisitorRegister.test.jsx | 11 | 11 | 0 |
| LandingPage.test.jsx | 18 | 18 | 0 |
| main.test.jsx | 1 | 1 | 0 |
| context/AuthContext.test.jsx | 6 | 6 | 0 |
| services/api.test.js | 8 | 8 | 0 |
| **TOTAL** | **76** | **76** | **0** |

Positive Tests: 55 | Negative Tests: 21