---
title: "Using 401 With Django Rest Framework"
summary: "Using 401 With Django Rest Framework"
tags: ["test"]
date: 2026-09-19
draft: true
showToc: true
TocOpen: true
lightCode: true
comments: true
---

The Django Rest Framework does not send the HTTP 401 status when using session
as the authentication framework. 
The reason for this is that technically no valid value can be set for 
"WWW-Authenticate".
While they might be right for saying that it not possible for how WWW-Authenticate is meant, and that indeed setting "WWW-Authenticate" is required for a HTTP 401. 
This is however unfortunate in that there is no difference in a HTTP 401 and a HTTP 403.

This piece of code will however configure your Django repository to send 401 errors on the session authentication backend.

```py
from django.contrib.auth.forms import AuthenticationForm
from django.http import JsonResponse
from rest_framework import authentication, status
from rest_framework.exceptions import AuthenticationFailed


def authenticate_header(request):
    return "Session"


class SessionAuthentication(authentication.SessionAuthentication):
    """
    This class is needed, because REST Frameswork's default SessionAuthentication does never return 401's,
    because they cannot fill the WWW-Authenticate header with a valid value in the 401 response. As a
    result, we cannot distinguish calls that are not unauthorized (401 unauthorized) and calls for which
    the user does not have permission (403 forbidden). See https://github.com/encode/django-rest-framework/issues/5968

    We do set authenticate_header function in SessionAuthentication, so that a value for the WWW-Authenticate
    header can be retrieved and the response code is automatically set to 401 in case of unauthenticated requests.
    """

    def authenticate_header(self, request):
        return authenticate_header(request)


class MyAuthenticationForm(AuthenticationForm):
    def get_invalid_login_error(self):
        raise AuthenticationFailed("Wrong password!")


class AuthMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        return self.get_response(request)

    def process_exception(self, request, exception):
        if isinstance(exception, AuthenticationFailed):
            headers = {"WWW-Authenticate": authenticate_header(request)}
            data = {"detail": exception.detail} if not isinstance(exception.detail, (list, dict)) else exception.detail
            return JsonResponse(data, status=status.HTTP_401_UNAUTHORIZED, headers=headers)
```

The `SessionAuthentication` with a custom `authenticate_header` is sufficient to get 401 already.
The override of the `AuthenticationForm` and settings of the `AuthMiddleware` however will give you the additional 
functionality that wrong passwords on the standard form endpoint give you a JSON error payload instead.
