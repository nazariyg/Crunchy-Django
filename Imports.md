# Django Imports

## Alphabetically

    django.conf.settings.AUTH_USER_MODEL
    django.conf.urls.include
    django.conf.urls.url
    django.contrib.admin.ModelAdmin
    django.contrib.admin.site
    django.contrib.admin.StackedInline
    django.contrib.admin.TabularInline
    django.contrib.auth.authenticate
    django.contrib.auth.decorators.login_required
    django.contrib.auth.decorators.user_passes_test
    django.contrib.auth.get_user_model
    django.contrib.auth.login
    django.contrib.auth.models.AbstractBaseUser
    django.contrib.auth.models.BaseUserManager
    django.contrib.auth.models.Group
    django.contrib.auth.models.Permission
    django.contrib.auth.models.PermissionsMixin
    django.contrib.auth.models.User
    django.contrib.contenttypes.models.ContentType
    django.core.exceptions.ObjectDoesNotExist
    django.core.exceptions.PermissionDenied
    django.core.urlresolvers.NoReverseMatch
    django.core.urlresolvers.reverse
    django.db.models.Avg
    django.db.models.Count
    django.db.models.F
    django.db.models.Manager
    django.db.models.Max
    django.db.models.Min
    django.db.models.Model
    django.db.models.Q
    django.db.models.Sum
    django.db.transaction.atomic
    django.db.transaction.non_atomic_requests
    django.http.Http404
    django.http.HttpRequest
    django.http.HttpResponse
    django.http.HttpResponseRedirect
    django.shortcuts.get_list_or_404
    django.shortcuts.get_object_or_404
    django.shortcuts.redirect
    django.shortcuts.render
    django.template.loader
    django.template.RequestContext
    django.test.TestCase
    django.utils.decorators.method_decorator
    django.utils.timezone
    django.utils.translation.ugettext
    django.views.decorators.http.require_GET
    django.views.decorators.http.require_http_methods
    django.views.decorators.http.require_POST
    django.views.decorators.http.require_safe
    django.views.generic.DetailView
    django.views.generic.ListView
    django.views.generic.RedirectView
    django.views.generic.TemplateView
    django.views.generic.View

## Snippets

### URL Routing

    from django.conf.urls import url
    from django.conf.urls import url, include

### Models and Forms

    import django.db.models as m
    from django.db.models import F
    from django.db.models import Q
    from django.core.exceptions import ObjectDoesNotExist
    from django.db.models import Manager
    from django.db.models import Count
    from django.db.models import Sum
    from django.db.models import Avg
    from django.db.models import Min
    from django.db.models import Max
    from django.db import transaction

### Views

    from django.http import HttpResponse
    from django.http import HttpResponseRedirect
    from django.http import JsonResponse
    from django.shortcuts import render
    from django.shortcuts import redirect
    from django.shortcuts import get_object_or_404
    from django.shortcuts import get_list_or_404
    from django.template import RequestContext, loader
    from django.http import Http404
    from django.core.urlresolvers import reverse
    from django.views import generic
    from django.views.generic import View
    from django.views.generic import TemplateView
    from django.views.generic import RedirectView
    from django.utils.decorators import method_decorator
    from django.views.decorators.http import require_http_methods
    from django.views.decorators.http import require_safe
    from django.views.decorators.http import require_GET
    from django.views.decorators.http import require_POST
    from django.core.exceptions import PermissionDenied

### Forms

    import django.forms as f

### Authentication & Authorization

    from django.contrib.auth.models import User
    from django.contrib.auth import authenticate
    from django.contrib.auth import authenticate, login
    from django.contrib.contenttypes.models import ContentType
    from django.contrib.auth.models import Permission
    from django.contrib.auth.models import Group
    from django.contrib.auth import get_user_model
    from django.contrib.auth.decorators import login_required
    from django.contrib.auth.decorators import user_passes_test
    from django.contrib.auth.models import AbstractBaseUser
    from django.contrib.auth.models import BaseUserManager
    from django.contrib.auth.models import PermissionsMixin

### Administration

    from django.contrib import admin

### Testing

    from django.test import TestCase
    from django.test.utils import setup_test_environment
    from django.test import Client
