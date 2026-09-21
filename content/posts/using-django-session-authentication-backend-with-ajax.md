---
title: "Using Django Session Authentication Backend With Ajax"
summary: "Using Django Session Authentication Backend With Ajax"
tags: ["test"]
date: 2026-09-19
draft: true
showToc: true
TocOpen: true
lightCode: true
comments: true
---

● Here's the AJAX login flow, Hielke.

  1. Get a CSRF cookie first
  web/src/api/api-client.ts:9-16 — fetchCsrfToken() does GET /elsra/user/csrf/. That path is not a real login/csrf view; it falls through to the catch-all route in api/src/main/urls.py:97:
  re_path(r"elsra/(?P<path>.*)$", ensure_csrf_cookie(lambda req, path: render(req, "index.html")))
  ensure_csrf_cookie forces Django to set the csrftoken cookie on the response, even though the body is just the app's index.html.

  2. Axios auto-attaches CSRF
  api-client.ts:18-24 configures axios with xsrfCookieName: "csrftoken" and xsrfHeaderName: "X-CSRFToken". Axios reads the cookie set in step 1 and adds it as a header on every later request.
  withCredentials: true makes the session cookie ride along too.

  3. Login POST is JSON, not a form
  web/src/routes/login/store/auth-actions.ts:11-18 — login thunk does api.post(LOGIN_PATH, { username, password }), sent as application/json. But the endpoint is Django's built-in LoginView
  (urls.py:62-65), which normally expects form-encoded request.POST.

  This is bridged by api/src/main/middleware.py — JSONMiddleware. It parses any application/json body and merges it into request.POST (for POST requests). So LoginView sees the JSON fields as
  if they were normal form data.

  4. Success or failure
  - Success: LoginView authenticates against AUTHENTICATION_BACKENDS (LDAP, then Django's model backend), creates a session, sets the sessionid cookie.
  - Failure: MyAuthenticationForm.get_invalid_login_error (authentication.py:26-28) raises AuthenticationFailed("Wrong password!"), caught by AuthMiddleware (authentication.py:38-42) → JSON
    401 response.

  5. Frontend reacts to 401
  The axios response interceptor (api-client.ts:37-48) catches any 401 anywhere in the app and calls navigate(LOGIN_ROUTE).

  6. Status check / logout
  - checkLogin thunk → GET /elsra/user/check-login/ → always 200, body has {authenticated: bool, username} (urls.py:46-56). Used to probe session state without triggering a 401 redirect.
  - `log
  ⎿  Interrupted · What should Claude do instead?
