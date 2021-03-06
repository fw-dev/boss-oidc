Django Authentication OpenID Connect plugin for the Boss SSO
============================================================

This package configured the django-oidc (jhuapl-boss fork) and drf-oidc-auth Django
authentication plugins for use with the Boss Keycloak authentication server providing
single sign-on (SSO) capability for the larger Boss infrastructure.

While boss-oidc used the OpenID Connect (OIDC) protocol for talking with the Keycloak
Auth server, there may be some Keycloak specific implementation details that are also
captured in the code. Testing with other OIDC providers has not been tested.


Quickstart
----------

Install bossoidc:

```sh
pip install git+https://github.com/jhuapl-boss/django-oidc.git
pip install git+https://github.com/jhuapl-boss/drf-oidc-auth.git
pip install git+https://github.com/jhuapl-boss/boss-oidc.git
```

Configure authentication for Django and Django REST Framework in settings.py:

```py
INSTALLED_APPS = [
    # ...
    'bossoidc',
    'djangooidc',
]

AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
    'bossoidc.backend.OpenIdConnectBackend',
    # ...
]

REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
    'DEFAULT_AUTHENTICATION_CLASSES': (
        # ...
        'rest_framework.authentication.SessionAuthentication',
        'oidc_auth.authentication.BearerTokenAuthentication',
    ),
}

# (Optional) A function used to process a user's roles for the application
#            It also provides a helpful hook for each time a user logs in
# Function Args:
#   user (User object): The user that is logging in
#   roles (list of string): List of the roles the user is currently assigned
LOAD_USER_ROLES = 'path.to.function'

# NOTE: The following two rules are automatically applied to all user account during
#       the login process to allow bootstrapping admin / superuser accounts.
# The user will be assigned Django staff permissions if they have a 'admin' or 'superuser' role in Keycloak
# The user will be assigned Django superuser permissions if they have a 'superuser' role in Keycloak

auth_uri = "https://auth.theboss.io/auth/realms/BOSS"
client_id = "<auth client id>" # Client ID configured in the Auth Server
public_uri = "http://localhost:8000" # The address that the client will be redirected back to
                                     # NOTE: the public uri needs to be configured in the Auth Server
                                     #       as a valid uri to redirect to

from bossoidc.settings import *
configure_oidc(auth_uri, client_id, public_uri)
```

Add the required URLs to the Django project in urls.py:

```py
url(r'openid/', include('djangooidc.urls')),
```

Run the following migration to create the table for storing the Keycloak UID

```sh
$ python manage.py makemigrations bossoidc
$ python manage.py migrate
```

You may now test the authentication by going to (on the development server) http://localhost:8000/openid/login or to any
of your views that requires authentication.


Features
--------

* Ready to use Django authentication backend
* Fully integrated with Django's internal accounts and permission system
* Stores Keycloak UID to improve Keycloak - Django account association
* Support for OpenID Connect Bearer Token Authentication


Legal
-----


Use or redistribution of the Boss system in source and/or binary forms, with or without modification, are permitted provided that the following conditions are met:
 
1. Redistributions of source code or binary forms must adhere to the terms and conditions of any applicable software licenses.
2. End-user documentation or notices, whether included as part of a redistribution or disseminated as part of a legal or scientific disclosure (e.g. publication) or advertisement, must include the following acknowledgement:  The Boss software system was designed and developed by the Johns Hopkins University Applied Physics Laboratory (JHU/APL). 
3. The names "The Boss", "JHU/APL", "Johns Hopkins University", "Applied Physics Laboratory", "MICrONS", or "IARPA" must not be used to endorse or promote products derived from this software without prior written permission. For written permission, please contact BossAdmin@jhuapl.edu.
4. This source code and library is distributed in the hope that it will be useful, but is provided without any warranty of any kind.

