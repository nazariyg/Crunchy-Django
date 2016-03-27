## See Also

* [Installation & Maintenance](Installation & Maintenance.md)
* [Imports](Imports.md)
* [Third-Party Packages](Third-Party Packages.md)
* [Django REST Framework](Django REST Framework/Django REST Framework.md)

***

# Django (v1.8)

## General

* Django is a web framework written in Python
* normally runs in its own virtual environment created with `virtualenv`
* `<virtualenvdir>/bin/activate` can set environment variables for the Django instance, such as:

```

export PYTHONPATH=...
export DJANGO_SETTINGS_MODULE=...
export DJANGO_SECRET_KEY=...
export DJANGO_DB_PASSWORD=...

```

* a Django instance for a web app is created with `django-admin startproject <appname> .` while in the virtual environment
* `django-admin startproject <appname> .` generates in the current directory `manage.py` and the directory of the app's main module which is a Python package named `<appname>`
* the directory that contains `manage.py` is the project directory or `BASE_DIR` (settings variable) of the Django instance
* the app's main module contains the Django instance's settings, the gateway URL router `urls.py`, etc.
* initial directory structure of a Django instance:

```

appname/
    manage.py
    appname/
        __init__.py
        settings.py
        urls.py
        wsgi.py

```

* a Django instance is usually managed with `python manage.py ...`
* Django is modular
* for entertaining ambiguity, a Django module is called Django app or simply app; this concept is referred to as *subapp* henceforth
* a subapp is usually a functional component of a web app, such as news or blog
* a subapp is a Python package inside `BASE_DIR`
* subapps are often meant to be pluggable, even into other Django web apps
* subapps that the Django instance is aware of are listed in `INSTALLED_APPS` variable in the settings
* the main module (where the settings are located) is a subapp too (not listed in `INSTALLED_APPS`)
* `INSTALLED_APPS` list initially includes a number of built-in subapps located elsewhere
* a subapp is created with `python manage.py startapp subappname`
* Django's modus operandi is **MVT**: *model*, *view*, *template*
* the view in MVT is analogous to the controller in MVC
* inside a subapp, the key elements are `views.py` for the subapp's views, `models.py` for models, `urls.py` for URL (sub)routing, `tests.py`, `admin.py`, etc.
* `DATABASES` dictionary in the settings lists the default and, optionally, other databases to be known by the Django instance
* Django's *middleware* is a set of hooks into request/response processing; active middleware components are listed in `MIDDLEWARE_CLASSES` settings variable
* `django.middleware.common.CommonMiddleware` middleware component, which is normally always activated, can forbid access to user agents based on `DISALLOWED_USER_AGENTS` setting, perform URL rewriting based on `APPEND_SLASH` and `PREPEND_WWW` settings (if `APPEND_SLASH` is `True`, any requested but unmatched URL that doesn't end in a "/" is tried again with a trailing "/"), handle ETags automatically based on `USE_ETAGS` setting

## URL Routing

* the URL routing of a Django instance is a tree starting in `urls.py` inside the main module (where settings are located) and optionally branching to `urls.py` inside the subapps of the Django instance so that the gateway `urls.py` of the main module delegates subsequent URL routing to subapps
* `urls.py` are also known as *URLconf*
* the main responsibility of a `urls.py` is to set `urlpatterns` variable to a list of `django.conf.urls.url`
* each `django.conf.urls.url` takes as parameters the regex of the routed **URI** without leading "/" and without query string, the corresponding view of the route as an object or string (with Python module path relative to `BASE_DIR`) and, optionally, the name of the route (etc.), e.g. `django.conf.urls.url(r"^$", "appname.views.home", name="home")`
* naming a route lets refer to it unambiguously from elsewhere, especially from templates
* instead of pointing to a view, the second argument to `django.conf.urls.url` can point to the `urls.py` router of a subapp with `django.conf.urls.include` in order to branch the routing out, e.g. `django.conf.urls.include("blog.urls")`
* a branching route is often associated with a routing namespace to avoid route name collisions between subapps in e.g. `{% url %}` template tag; the namespace is usually named after the subapp into which the route is branching out, e.g. `django.conf.urls.url(r"^subappname/", include("subappname.urls", namespace="subappname"))`
* unless it's a branch, the regex of a route is anchored at both ends ("^...$")
* for path-like URIs, regex chaining from the gateway `urls.py` to subapps uses "/" at the end of any preceding regex as a delimiter and never at the beginning of the subsequent regex in the next `urls.py`
* for path-like URIs, any regex that leads to a view usually ends with a "/$"
* referring to a view or another router by string instead of by object lets avoid massive imports in the gateway `urls.py`; a string points to the route's destination relative to the `BASE_DIR`
* a regex in `django.conf.urls.url` can contain simple or named regex groups to pass captured strings to the view; any value passed from a URL route to a view is a string
* if the regex uses simple captures, values are passed to the target view as positional arguments; if it uses named captures, values are passed as keyword arguments
* using named regex groups allows for assigning default values to parameters in target view functions/methods
* only the gateway `urls.py` can contain `handler400`, `handler403`, `handler404`, and `handler500` variables to customize HTTP error views if the variable is set to the target view, possibly as a string
* instead of specifically pointing to a subapp's URL router, `django.conf.urls.include` can take as a parameter a list of `django.conf.urls.url`
* the string in a regex group captured in the gateway URL router can as well go as a keyword argument to the URL router of any subapp
* `django.conf.urls.url` can take an optional third argument which should be a dictionary of extra keyword arguments to pass to the view function/method; `django.conf.urls.include` works in the same way with its last optional argument
* `django.core.urlresolvers.reverse` can resolve URLs (or, more precisely, URIs) from their route names and application namespaces; rises `django.core.urlresolvers.NoReverseMatch` on fail
* for reverse-resolving of URLs when multiple instances of a subapp are running (e.g. multiple admin sites), two URL namespaces are used: application namespace and application instance namespace

## Models

* models are located in `models.py` of the subapp or in `models` package of the subapp
* any model class derives from `django.db.models.Model`
* in Django terms, model instance is called object
* model instances are compared to one another with `==`, which compares the instances' primary keys behind the scenes
* class attributes in the model's class correspond to fields in the model's table in the database, using attribute names for the table's field (column) names
* each field is represented by a subclass of `django.db.models.Field`
* `django.db.models.ForeignKey` is a field synonymous to many-to-one relationship
* a model often implements `__str__` method right away so that its instances could tell what they are when evaluated to string or listed in shell
* even though model fields are defined as class attributes, Django makes them available for model instances as instance attributes
* when creating a model instance, values for its fields are provided via keyword arguments
* after creation, a model instance can be saved to the database with `save` method; with `ModelClass.objects.create`, a model instance can be created and saved in a single call; a model instance can be deleted with `delete` method
* the ID (primary key) of a model instance is in its `id` attribute and is only available after the instance is saved
* when defining a field, the first positional argument can be a human-readable name of the field, called verbose name, except for relational field types that use `verbose_name` argument; by convention, the first letter of a verbose name is not capitalized; this can also be set with `verbose_name` keyword argument
* a field can have a default value **or** callable provided with **`default`** argument when defining the field
* when defining a field, **`null`** argument can specify whether NULL is an allowed value for the field in the database; default is `False`; may play an important role for many-to-one relationship fields
* any field that is not allowed to be NULL (default behavior) must have a value or a default value provided with `default` argument
* when defining a field, **`choices`** argument can specify an iterable with tuples where the first element is the string of how the choice stored (the choice's code) and the second is how the choice is displayed; `max_length` is often used too to specify the maximum code length; the display value can be obtained with `model_instance.get_<field_name>_display()`
* when defining a field, **`blank`** argument can specify whether the field's value is allowed to be blank during form validation; default is `False`
* when defining a field, **`unique`** argument can specify whether this field should be unique throughout the table
* when defining a field, **`primary_key`** argument can specify whether this field should be used as the primary key for the model instead of an automatically created `IntegerField`
* when defining a field, **`help_text`** argument can specify an extra help text to be displayed with the form widget
* also when defining a field, **`db_index`** argument can specify whether to create an index on the field, **`db_tablespace`** can specify a tablespace for the index, **`db_column`** argument can specify the name of the database column to use for the field, and **`editable`** can specify whether the field should be displayed in the admin site and included for validation; other keyword arguments are `error_messages`, `unique_for_date`, `unique_for_year`, `validators`
* model field summary (fuzzy-ordered by use frequency):

```python

max_length    # for CharField; for file-related fields, the default is 100
default       # the value used when field's value is not specified (None/NULL/null/"" is a value)
null          # in Django, textual fields should not be null=True and should default to "" instead (except if file-related);
              #   often used for ForeignKey; has no effect on ManyToManyField; if null=True, then typically blank=True too
blank         # blank=True if None/NULL/null/"" is an acceptable value in validation (if None/NULL/null is explicitly specified);
              #   if blank=True, then typically null=True too
choices       # often used with max_length
unique        # uniqueness across the model's table
db_index      # can improve the performance of querying on that field
verbose_name  # mostly for the admin site
on_delete     # for relationship fields

```

* common field types:

```python

CharField(max_length=...)   # short, limited-length text
TextField                   # arbitrary-length text
EmailField                  # email with validation
URLField
SlugField
BooleanField
NullBooleanField            # for boolean fields allowed to be NULL
IntegerField                # 4 bytes
BigIntegerField             # 8 bytes
FloatField
DateTimeField               # date and time; options & defaults: auto_now_add=False (creation), auto_now=False (modification)
DateField                   # just date (same options & defaults)
FileField                   # for file uploads (stored as file path relative to MEDIA_ROOT, default max_length is 100)
ImageField                  # like FileField but with validation

ForeignKey                  # many-to-one relationship
ManyToManyField             # many-to-many relationship
OneToOneField               # one-to-one relationship

```

* as a general rule, textual fields (which include file-related fields which store file paths) should not be created with `null=True` since the convention is to store empty textual data as empty strings, but it's ok to create textual fields with `blank=True`; for boolean, numeric, and time-related fields, it's ok to use `null=True` and `blank=True` (for such fields, `blank=True` usually means that `null=True` is needed too); for relationship fields, using `null=True` and `blank=True` is ok
* Django allows for custom field types
* a model class can be given metadata in the form of an inner class named `Meta`
* model metadata is anything that's not a field, such as ordering options with `ordering` class attribute of `Meta` e.g. `ordering = ["-pub_date", "author"]`, custom database table name with `db_table`, or human-readable singular and plural names of model instance(s) with `verbose_name` and `verbose_name_plural`
* among others, model metadata options also include `abstract` (model inheritance), `proxy` (model inheritance), `managed` (for controlling whether migrations should be performed for the model), `permissions` (custom permissions for the model), `unique_together` (field names that, taken together, must be unique), `index_together` (field names that, taken together, are indexed), etc.
* in addition to `__str__`, a model often implements `get_absolute_url` method to be used to calculate the URI for a model instance for templates, the admin site, etc. (the output must contain only ASCII characters and be URL-encoded where necessary); `django.utils.encoding.iri_to_uri` could be used
* `save` and `delete` methods of models are sometimes overridden to add custom logic when saving and deleting model instances but this may become problematic when performing operations on model instances in bulk
* a model instance can be copied (except relationship fields) by setting its `pk` attribute to `None` and then saving the instance, in which case the instance will be duplicated in the database with a new ID; however, to copy an instance of a model derived from another model, both `pk` and `id` attributes should be set to `None`; relationship fields can only be copied manually (by storing them in a temporary variable before performing basic copying)
* instead of overriding `__init__` method of a model class for custom initialization logic, it's preferred to either add a class method on the model class, e.g. `@classmethod def create(...`, or add a method on a custom manager
* when using PostgreSQL, `pg_stat_activity` tells what indexes are actually being used

### Model Relationships

* relationships between models are established by means of `ForeignKey` (many-to-one relationship), `ManyToManyField`, and `OneToOneField` field types; when defining a relationship field, the model on which the field is defined is the *origin* (*source*) model, and the *target* model is provided as the first argument to the field's definition
* `ForeignKey` is defined on the model that is on the "many" side of a many-to-one relationship
* the related model can be referenced by a Python object or, for models not yet defined, by a string with the class name of the related model, possibly prefixed with `"other_subapp."`
* model's relationship to itself can be specified with `"self"` as the target model
* automatic creation of an index for a relationship field can be disabled with `db_index=False`
* in the database, relationship field names are suffixed with `_id`
* when defining a relationship field, `on_delete` argument can be used to specify what should happen to the referencing model instance when the referenced model instance is deleted; the default is `django.db.models.CASCADE` (delete too), other options are `SET_NULL` (if NULL is allowed for the relationship field), `PROTECT` (prevent deletion raising an exception), `SET_DEFAULT` (if default was specified), `SET(value_or_callable)`, `DO_NOTHING`
* when defining a relationship field, `related_name` and `related_query_name` arguments can be used to customize the names by which the origin model is referenced as an attribute of a target model's instance or when writing fields lookups in target model's queries
* when defining a relationship field, `limit_choices_to` argument can limit available choices for the field when this field is rendered using a `ModelForm` or the admin, e.g. `{"is_staff": True}`
* to represent a many-to-many relationship, Django creates an intermediary table
* when defining a many-to-many relationship, `through` argument can specify the model to be used for the intermediary table; if specified and the intermediary model references any of the two related models more than once, thus creating ambiguity for Django, `through_fields` argument can specify the names of the origin and target `ForeignKey` fields in the intermediary table, as a tuple, otherwise the fields are determined automatically; this also applies to recursive relationships
* when defining a model's many-to-many relationship to itself, `symmetrical` argument controls whether the relationship is symmetrical so that there's no need to create an attribute for the "reverse" association; the default is `True`
* recursive relationships using an intermediary model must be defined as non-symmetrical with `symmetrical=False`
* `null` argument has no effect on `ManyToManyField`
* when defining a one-to-one relationship in a model that inherits from another model, `parent_link` argument can specify whether this field should be used as the link back to the parent class, rather than the extra `OneToOneField`
* it's suggested that the name of a `ForeignKey` is singular and the name of a `ManyToManyField` is plural
* for a many-to-many relationship, it's irrelevant which of the two models defines a `ManyToManyField`, but `ManyToManyField` often goes into the model that is most dependent or can be edited in a form
* when no intermediary model is used, a many-to-many relationship can be populated with `add(..., ...)` and `create` methods of the relationship field on the origin model; a reverse relationship field is created automatically on the target model; related model instances can also be given in an iterable and assigned directly if the `ManyToManyField` was defined with `null=True` (the model instances will be added otherwise); related model instances can be disassociated by `remove` and `clear` (remove all) methods
* when an intermediary model is used for a many-to-many relationship, an association between two models is established by creating an instance of the intermediary model
* when querying either side of a many-to-many relationship, fields of the intermediary model can be queried too referencing the intermediary model by its lowercased class name
* a one-to-one relationship is typically to extend one model with the additional fields of another model; instead of making a one-to-one relationship, the same effect is often achieved by means of non-abstract model inheritance (which creates an implicit one-to-one relationship)
* when modifying a `ForeignKey` field of a model instance, the instance needs to be saved afterwards for the changes to be propagated to the database, but when modifying a `ManyToManyField`, the changes are saved automatically
* if a `ForeignKey` field allows for NULL values, it can be assigned `None` to remove the association

### Querying

* in addition to code, the Django's database can be queried from the shell available via `python manage.py shell`
* Django provides a rich database lookup API that for a big part is driven by keyword arguments
* `objects` attribute of a model class manages its instances is called the model's *manager*; `objects` is just the default name and a model can have multiple such managers
* many-to-many relationship fields and auto-generated "reverse" relationship attributes (`<relatedmodelclass>_set`) are managers too and manage related model instances
* the retrieval of model instances from the database is **prepared** with *QuerySet* objects
* querysets are constructed with methods of the model's manager
* queries can combine and chain for possible optimizations before the resulting queryset is evaluated and any database hit is done (querysets are lazy)
* model -> manager -> one or more querysets -> evaluation (implicit) and database hit
* a queryset is iterable, and a queryset is evaluated the first time it's iterated or indexed
* querysets can be lazily combined with Python slices (`[:]`) to impose limitations (except for negative indexing); no subsequent query chaining can be done past a slice; if the slice has a step, it evaluates the query
* the results of the evaluation of a queryset is cached so database hits are minimized for going through the same evaluated queryset multiple times
* the evaluation of a queryset can be forced by calling `list(queryset)` on it

```python

ModelClass.objects.all()
ModelClass.objects.order_by("-created")[:5]
ModelClass.objects.filter(num_stars__gte=4).count()

ModelClass.objects.filter(text_field__startswith="prefix", ...).<anothermethod>...
#                  ^      ^                      ^
#                  |      |                      |
# manager/QuerySet method |                      |
# field lookup ------------                      |
# value/expression for lookup --------------------

```

#### Managers and Querysets

* the methods of a model's manager for initial querying **and** the QuerySet methods for query chaining:

```python

# methods that return querysets:

all()                  # all instances (in e.g. ModelClass.objects.filter this method is implicit), also used for queryset copying
order_by(..., ...)     # all instances ordered by fields of the given names, ascending, or descending with "-" in front
order_by("?")          # randomize at the database level
filter(...)            # only with instances matching given field lookups
exclude(...)           # without instances matching given field lookups
reverse()              # reverse order
distinct()             # distinct instances only (doesn't work in some cases involving order_by)
defer(...)             # retrieve all field values except for the fields with the given names, retrieve deferred fields on-demand
only(...)              # defer all but the fields with the given names
values(..., ...)       # (ValuesQuerySet) the values of all fields or of the fields with the given names, stored in dictionaries
values_list(..., ...)  # same but returning tuples
select_related(...)    # returns a queryset that will let "follow" ForeignKey fields without database hits; may increase performance
prefetch_related(...)  # similar to select_related but uses a different strategy
annotate(...)          # just for the query, creates pseudo-fields named after the given keyword arguments and with values computed by the given expressions
select_for_update()    # returns a queryset that will lock model instances at the row level until the end of the transaction

dates(...)
datetimes(...)
using(...)
raw()
none()

# methods that don't return querysets:

get(...)               # returns the model instance matching given field lookups
count(...)             # quantity
exists(...)            # True if the given field lookups result in any database records
create(...)            # for a model's manager, creates a new instance of the model and saves it
latest(...)            # returns the latest object in the table, using the field of the given name or get_latest_by model metadata option
earliest(...)          # returns the earliest object in the table, using the field of the given name or get_earliest_by model metadata option
aggregate(...)         # returns a dictionary of aggregate values (averages, sums, etc) calculated over the queryset
update(...)            # sets fields of matching database records in bulk
delete()               # deletes matching database records in bulk
get_or_create(...)     # creates a new instance of the model if such instance does not already exist

bulk_create(...)
in_bulk(...)
update_or_create(...)
first()
last()

```

* when using `get` to retrieve a single model instance and the model instance cannot be found, the query raises `ModelClass.DoesNotExist` exception (inherits from `django.core.exceptions.ObjectDoesNotExist`); conversely, when a `get` query results in more than one record, `ModelClass.MultipleObjectsReturned` exception is risen
* instead of values for field lookups, one can use query *expressions*, e.g. `django.db.models.F("counter_field") + 1`
* for better performance, fields of multiple model instances can be set in bulk with `update` method of QuerySet. e.g. `update(field_name=<newvalue>)`; only non-relationship fields and `ForeignKey` fields can be set in this way and only the model's table can be modified; bulk updates can use `django.db.models.F` expressions but the selection of fields for `django.db.models.F` is limited to the model's fields only
* similarly, when deleting multiple model instances, all the database records of the model instances of a queryset can be deleted in bulk with `delete` method invoked on the queryset
* `update`s and `delete`s in bulk may avoid race conditions
* race conditions can also be avoided by referencing field values directly in the query by means of expressions with `django.db.models.F`, annotations, and aggregations or by assigning such expressions to fields before `save`
* when accessing a model instance that is related to a model instance in the results of a queryset by many-to-one relationship, it hits the database but the hit is cached in the queryset so that subsequent accesses to the same related model instance don't make any new database hits
* when many model instances related by many-to-one relationship (`ForeignKey`) are expected to be accessed on the results of a queryset, database hits can be minimized by telling the queryset to recursively pre-cache model instances related to the future results with `select_related` method, which returns another queryset; the order of `filter` and `select_related` chaining isn't important
* expressions and a number of arithmetic and statistical functions can be performed on field values at the database level with `aggregate` and `annotate` manager/QuerySet methods; `aggregate` looks into field values of multiple records defined by the query and results in a single value stored in a dictionary, whereas `annotate` looks into field values of every individual record defined by the query and stores the result in a pseudo-field ready to be used for any other purpose, such as ordering
* the key name for `aggregate` and the pseudo-field's name for `annotate` are specified by the name of a keyword argument to the method with the argument's value being the expression to be computed
* an expression to be computed for `aggregate`/`annotate` can be `django.db.models.Count("field_name...")` for counting the number of related instances, `Sum("field_name...")` for summation, `Avg("field_name...")` for computing the average value, `Min("field_name...")` and `Max("field_name...")` for finding minimum and maximum values, etc.; fields can follow forward and reverse relationships; other expressions that are supported by methods such as `filter`, `exclude`, and `order_by` can be used as well, e.g. `django.db.models.F` for in-query field evaluation
* `aggregate` and `annotate` can be constrained to only particular model instances via preceding `filter` or `exclude`
* `aggregate` and `annotate` can take multiple arguments
* pseudo-fields created with `annotate` can be used in subsequent query chaining with methods such as `filter`, `exclude`, and `order_by`
* for annotations, aggregations, filters etc., Django supports database-level functions such as `django.db.models.functions.Coalesce`, `Concat`, `Length`, `Lower`, `Substr`, `Upper`
* in an `order_by` method with an expression instead of a value, the expression can be suffixed with `.asc()` or `.desc()`
* the output of an expressions can be aimed at a specific field type with `django.db.models.ExpressionWrapper` (where specifying the output field type is not supported natively)
* the related model in a one-to-one relationship also has access to a manager object, but that manager represents a single model instance
* a queryset indicates whether it was ordered in any way in its `ordered` attribute

#### Forward and Reverse Relationship Managers

* from an instance of a target model that is in a relationship with an origin model where `ForeignKey` or `ManyToManyField` is defined, related model instances can be accessed in the "reverse" way with automatically generated `<relatedmodelclass>_set` attribute (lowercased name of the reverse-related model suffixed with `_set`), e.g. `author.blogentry_set`
* the methods of a `ManyToManyField`'s manager for forward-related model instances and the methods of a manager for reverse-related instances in relationships defined with `ForeignKey` and `ManyToManyField`:

```python

rel_manager = # model_instance.relationship_field (forward)
# or
rel_manager = # model_instance.<relatedmodelclass>_set (reverse)

# methods that return querysets:

rel_manager.all()
rel_manager.order_by(..., ...)
rel_manager.order_by("?")
rel_manager.filter(...)
rel_manager.exclude(...)
rel_manager.distinct()
rel_manager.defer(...)
rel_manager.only(...)
rel_manager.values(..., ...)
rel_manager.values_list(..., ...)
rel_manager.select_related(...)
rel_manager.prefetch_related(...)
rel_manager.annotate(...)
rel_manager.select_for_update()
rel_manager.dates(...)
rel_manager.datetimes(...)
rel_manager.using(...)
rel_manager.raw()
rel_manager.none()

# methods that don't return querysets:

rel_manager.get(...)
rel_manager.count(...)
rel_manager.exists(...)
rel_manager.latest(...)
rel_manager.earliest(...)
rel_manager.aggregate(...)
rel_manager.update(...)
rel_manager.delete()
rel_manager.in_bulk(...)
rel_manager.first()
rel_manager.last()

rel_manager.clear()             # disassociates all (if allowed by null=True)

# and if it's not a many-to-many relationship through a custom model:

rel_manager.create(...)         # creates a new related instance and saves it
rel_manager.add(..., ...)       # associates one or more instances, possibly by their primary key values
rel_manager = [..., ...]        # same but with overriding; in case of ForeignKey, overridden only if null=True, added otherwise;
                                #   any iterable and its elements can be primary key values
rel_manager.remove(..., ...)    # disassociates one or more instances, possibly by their primary key values (if allowed by null=True)

rel_manager.bulk_create(...)
rel_manager.get_or_create(...)
rel_manager.update_or_create(...)

```

#### Field Lookups

* basic lookups keyword arguments take the form `field_name__lookup=value`
* multiple field lookups in a manager method combine with logical AND
* if the type of a field is `ForeignKey`, an additional field name is available for lookup, which is the field's name suffixed with `_id`
* common field lookups:

```python

exact        # exact match; assumed by default if no lookup is specified for a field name
iexact       # case-insensitive exact match
contains     # containment match
icontains    # case-insensitive containment match
startswith   # starts-with match
istartswith  # case-insensitive starts-with match
endswith     # ends-with match
iendswith    # case-insensitive ends-with match
regex        # regex match with the regex flavor is that of the database used
iregex       # regex match, case-insensitive
gt           # greater-than match
gte          # greater-than-or-equal match
lt           # less-than match
lte          # less-than-or-equal match
in           # in-match against iterable alternatives, which can be a queryset, possibly narrowed to values with values("field_name")
range        # match against a range given by two values, inclusive
year         # for date and datetime fields, a year match
month        # for date and datetime fields, a month match, 1 (January) to 12 (December)
day          # for date and datetime fields, a day match
week_day     # for date and datetime fields, a day-of-week match, 1 (Sunday) to 7 (Saturday)
hour         # for datetime fields, an hour match, 0 to 23
minute       # for datetime fields, a minute match, 0 to 59
second       # for datetime fields, a second match, 0 to 59
isnull       # NULL match, takes either True or False

```

* the field lookup API automatically follows relationships as far as indicated with the same `__` separator, e.g. `blog.author__name...` given that `author` is a `ForeignKey` on `Blog` model
* to refer a an instance of the origin model in a relationship from an instance of a target model during field lookup, the origin model's lowercased name is used (**without** `_set`), e.g. `Author.objects.filter(blog__name...)`
* if a relationship field does not lead to any modes instances, it's equivalent to NULL and so are subsequent lookups, which may be misleading if `isnull` lookup is involved at some point
* instead of a value, a field lookup can compare against the value of a field in the same model or another; such value is retrieved with a `django.db.models.F` expression, e.g. `django.db.models.F("field_name")`; `__` can also be used to "follow" relationships
* Django supports the use of addition, subtraction, multiplication, division, modulo, and power arithmetic with `django.db.models.F`, both with constants and with other `django.db.models.F`; for date and date/time fields, `timedelta` can be added or subtracted
* to combine field lookups with logical OR or with logical NOT, normally formatted lookups can be wrapped into `django.db.models.Q` objects and then combined with `&` (AND), `|` (OR), or `~` (NOT) to compose statements of arbitrary complexity
* normal lookups and `django.db.models.Q` objects are interchangeable: each manager method that takes keyword arguments as lookups (e.g. `filter`, `exclude`, `get`) can also be passed one or more `django.db.models.Q` objects as positional arguments; as with normal lookups, multiple `django.db.models.Q` objects will be AND-ed together

### Model Inheritance

* model inheritance in Django has 3 types that differentiate based on whether the base and derived models both have their separate tables in the database or not and on whether or not it only needs to be a Python class inheritance for additional functionality and with model fields untouched; for all the types, the Python's class inheritance syntax is used
* **abstract**: if only the derived model needs a database table, the base model class is made abstract (in Django terms) so that it only stores fields that are common to some models and propagates those fields to any derived model; no instances can be created of an abstract base model; a model class is marked as an abstract base model by putting `abstract = True` into its inner `Meta` class; model class metadata is inherited too if the derived model does not define any `Meta` inner class but `abstract` is implicitly set to `False`; this type of model inheritance is used most commonly
* **non-abstract** a.k.a. *multi-table* or *concrete*: if both base and derived models need to have tables, the to-be-derived model inherits from the base model normally; this typed of model inheritance creates an implicit one-to-one relationship between the two models; model class metadata is not inherited, with the exception of `ordering` and `get_latest_by`; a link to the derived model is created automatically in the base model and named after the derived models' class (lowercased); a link from the derived to parent model can be created by defining a `OneToOneField` with `parent_link=True`
* **proxy**: used to change the Python behavior of a model without adding new fields or tables; the to-be-derived model marked as proxy by putting `proxy = True` into its inner `Meta` class; one of the uses of proxy models is to change the default manager or redefine ordering
* as a good practice, non-abstract inheritance should be avoided

## Transactions

* Django’s default behavior is to run in autocommit mode: every query is immediately committed to the database, unless a transaction is active
* for HTTP requests, Django can be told to automatically wrap any view function/method of a response into a transaction by setting `ATOMIC_REQUESTS` option in the database's settings, which is `False` by default
* atomic requests can be inefficient for high traffic and don't behave well with streamed responses
* when `ATOMIC_REQUESTS` is `True`, individual view function/methods can be excluded from being wrapped into transactions by decorating them with `@django.db.transaction.non_atomic_requests`
* when `ATOMIC_REQUESTS` is `False` (default), individual view function/methods can be wrapped into transactions by decorating them with `@django.db.transaction.atomic`
* to wrap portions of code into a transaction, `django.db.transaction.atomic` can be used as a Python context manager:

```python

with django.db.transaction.atomic():
    ...

```

* a failed transaction results in a `django.db.IntegrityError` exception (or `django.db.DatabaseError`)
* no exception handling should take plane inside a `django.db.transaction.atomic()` block; if any, exception handling should happen around the block
* `django.db.transaction.atomic()` blocks can be nested; by default, entering any inner block creates a transaction savepoint
* transactions should be kept as short as possible

## Migrations

* a *migration* represents the changes to the models of one or more subapps as well as the changes that need to be correspondingly applied to the database
* migration is distinct from *migrating*, which is applying changes stored in migration files to the database
* migrations are a soft buffer between models and the database, allowing for greater flexibility and safety
* migrations are generated and stored in `migrations` subdirectory of the respective subapp with `python manage.py makemigrations` or `python manage.py makemigrations subappname`
* migration files are human-editable
* the name of a migration file starts with the migration's ID/number
* the SQL representation of a migration file can be derived from the migration with `python manage.py sqlmigrate subappname <migrationid>`
* Django appends "_id" to the foreign key field name
* before making migrations, the wellbeing of a Django instance can be checked with `python manage.py check`
* new migrations are applied to the database with `python manage.py migrate`
* for large tables in a PostgreSQL database, adding new fields with `null=True` avoids rewriting the whole table
* to merge multiple migrations into a single file, `python manage.py squashmigrations ...` can be used

## Views

* in Django, web pages and other content are delivered by views
* views are located in `views.py` of the subapp or main module
* Django views can be either *function-based* or *class-based*
* *generic views* are class-based views designed to simplify some common view patterns, such as rendering a model instance or a list of model instances
* each view is represented by a function or, in the case of class-based views, a method
* GET and POST query string values are in `request.GET` and `request.POST` dictionaries; these values are always strings
* a view is expected to return `django.http.HttpResponse` (or its subclass) or raise an exception, and nothing else
* a basic text response from a view can be made with `django.http.HttpResponse`
* when rendering a template, variables are communicated to it by means of a dictionary
* a more complex way to return a response would be to load and then render a template passing it a request context:

```python

...
template = django.template.loader.get_template("subappname/template_name.html")
context = django.template.RequestContext(request, {
        "context_var_name": some_local_var,
    })
return django.http.HttpResponse(template.render(context))

```

* with `django.shortcuts.render`, the above can be reduced to:

```python

...
return django.shortcuts.render(request, "subappname/template_name.html", {
        "context_var_name": some_local_var,
    })

```

* with `django.shortcuts.get_object_or_404`, `django.http.Http404` exception is risen automatically if a requested model instance does not exist; same goes for `django.shortcuts.get_list_or_404` if the filter returns an empty list

```python

django.shortcuts.get_object_or_404(ModelClass, ...)

```

* a view can return a `django.http.HttpResponseRedirect` passing it the target URI (with leading `/`) possibly reverse-resolved with `django.core.urlresolvers.reverse`, e.g. `django.core.urlresolvers.reverse("url_namespace:url_route_name", args=[..., ...], kwargs={...: ..., ...: ...})`
* there are subclasses of `django.http.HttpResponse` for a number of common HTTP status codes other than 200, e.g. `HttpResponseNotFound`; alternatively, the HTTP status code of the response can be passed to `HttpResponse` with `status` argument, e.g. `HttpResponse(status=201)`
* unlike `django.http.HttpResponseNotFound` response, which needs to be given a response body every time it's returned, the usage of `django.http.Http404` exception is to promote consistent 404 pages
* alternatively to `django.http.HttpResponseRedirect`, `django.shortcuts.redirect` can be used (which can accept a model instance as the address)
* any view is passed `django.http.HttpRequest` of the current request as the first positional argument
* some attributes and methods of `django.http.HttpRequest` are:

```python

# attributes:

user              # the user of the request
scheme            # protocol
path              # URI (without query string)
method            # HTTP method
body              # raw HTTP body
GET               # GET key-values
POST              # POST key-values (without file uploads)
COOKIES           # COOKIES key-values
FILES             # uploaded files as name-UploadedFile for key-values
META              # values of the available HTTP headers and WSGI environment variables:
  # HTTP headers:
  HTTP_ACCEPT
  HTTP_ACCEPT_ENCODING
  HTTP_ACCEPT_LANGUAGE
  HTTP_HOST
  HTTP_REFERER
  HTTP_USER_AGENT
  CONTENT_LENGTH  # exception
  CONTENT_TYPE    # exception
  ...
  # other:
  QUERY_STRING
  REMOTE_ADDR
  REMOTE_HOST
  REMOTE_USER
  REQUEST_METHOD
  SERVER_NAME
  SERVER_PORT
  ...
session           # current session's key-values, writable
current_app       # application namespace for URL resolving (e.g. in templates)
upload_handlers

# methods:

get_host()        # originating host inferred from various HTTP headers and WSGI info
get_full_path()   # URI with query string
is_secure()       # `True` if the request was made via HTTPS
is_ajax()         # `True` if the request was made with XMLHttpRequest
build_absolute_uri(...)
get_signed_cookie(...)
read(...)
readline()
readlines()
xreadlines()

```

* with a few exceptions, any HTTP headers in the request are converted to `META` keys by converting all characters to uppercase, replacing any hyphens with underscores and adding an "HTTP_" prefix to the name
* WSGI environment variables can also be found in `environ` attribute of a `django.http.HttpRequest`
* some attributes and methods of `django.http.HttpResponse` are:

```python

content
charset
status_code
reason_phrase
streaming
closed

response[...] = ...     # sets an HTTP header
del response[...]       # removes an HTTP header
has_header(...)
set_cookie(...)
set_signed_cookie(...)
delete_cookie(...)

```

* the initializer of `django.http.HttpResponse` accepts `content_type` argument for Content-Type header, `status` for the status code, etc.
* a `django.http.HttpResponse` can be used as a file-like object with corresponding methods
* for setting the Cache-Control and Vary header fields, it's recommended to use `django.utils.cache.patch_cache_control` and `django.utils.cache.patch_vary_headers`
* HTTP-related `django.http.HttpResponse` subclasses include `HttpResponseRedirect` (status code 302), `HttpResponsePermanentRedirect` (301), `HttpResponseNotModified` (304), `HttpResponseBadRequest` (400), `HttpResponseNotFound` (404), `HttpResponseForbidden` (403), `HttpResponseNotAllowed` (405), `HttpResponseGone` (410), `HttpResponseServerError` (500)
* other `django.http.HttpResponse` subclasses and response-like classes are `JsonResponse` for JSON responses, `StreamingHttpResponse` for streamed responses (not a subclass), `FileResponse` for file streaming (a subclass of `StreamingHttpResponse`)
* a file attachment for download can be set as follows:

```python

response = django.http.HttpResponse(file_data, content_type="filetype")
response["Content-Disposition"] = 'attachment; filename="..."'

```

* all class-based views ultimately inherit from `django.views.generic.View`; in the same module, `RedirectView` is for a simple HTTP redirect, and `TemplateView` extends the base class to make it also render a template
* in a `urls.py`, a class-based view is converted to a callable with `as_view` method
* class attributes of a class-based view, which can be set on the class itself, can also be passed as corresponding keyword arguments to `as_view` method right in the `urls.py`, e.g. `django.views.generic.TemplateView.as_view(template_name="...")` or `django.views.generic.RedirectView.as_view(url=...)`
* a class-based view has `http_method_names` attribute by default set to about 8 supported HTTP methods (need to be in lowercase when overridden); among others, `http_method_not_allowed` method is called if the view was called with a HTTP method it doesn't support, and `options` method handles responding to requests for the OPTIONS HTTP method
* method names of class-based views naturally correspond to the names of HTTP methods, lowercased
* if there is no method that matches the HTTP method of the request, `django.http.HttpResponseNotAllowed` is returned
* before a method of a class-based view is called, `as_view` calls `dispatch` method
* `get` is the method that `django.views.generic.View` subclasses implement by default
* generic views derive from classes in `django.views.generic`; in the URL routes of generic views, a target view is address as `views.ViewClass.as_view()`
* `django.views.generic.DetailView` expects its URL route to capture a regex group named `pk`, which is the ID of the model instance; the model of the view class is indicated by `model` class attribute; by default, a view derived from `DetailView` looks for its template as `subappname/<modelclass>_detail.html` (where `modelclass` is the lowercased name of the model class), which can be overridden with `template_name` class attribute; by default, the context variable containing the model instance of a detail view in the template is expected to be referenced as `<modelclass>`, which can be customized with `context_object_name` class attribute; the list of model instances from where the requested instance should be selected from can be specified with `get_queryset` method or `queryset` class attribute; how the model instance is retrieved can be customized in `get_object` method
* `django.views.generic.ListView` is similar to `DetailView` except that `pk` regex group is not used, the default template name is rather `subappname/<modelclass>_list.html`, the list to be displayed can be returned from `get_queryset` method of the view class or defined in `queryset` class attribute, the default name of the context variable containing model instances to be listed is `object_list`, which can be overridden with `context_object_name`, etc.
* the dictionary with context variables that are to be used when displaying a generic view can be customized by overriding `get_context_data` method in a subclass (the initial dictionary in an overridden method is obtained by calling `get_context_data` on the superclass)
* positional and keyword arguments from `urls.py` to a (customization) method of a generic view are also stored in `self.args` and `self.kwargs` respectively
* some other generic views are `django.views.generic.CreateView`, `UpdateView`, `DeleteView`, etc.
* the methods of a class-based view can be decorated via overriding `as_view` method in a mixin or by decorating the `dispatch` method:

```python

class Mixin():
    @classmethod
    def as_view(cls, **kwargs):
        view = super(Mixin, cls).as_view(**kwargs)
        return decorator(view)
class ViewClass(Mixin, ...):
    ...

```

```python

class ViewClass(...):
    @django.utils.decorators.method_decorator(decorator)
    def dispatch(self, *args, **kwargs):
        return super(ViewClass, self).dispatch(*args, **kwargs)
    ...

```

* view decorators:

```python

# in django.contrib.auth.decorators:

login_required (depends on LOGIN_URL setting)
user_passes_test

# in django.views.decorators.http:

condition(etag_func=..., last_modified_func=...)
etag(...)
last_modified(...)

# can return django.http.HttpResponseNotAllowed
require_http_methods([..., ...])   # HTTP methods should be in uppercase
require_safe                       # only GET and HEAD
require_GET
require_POST

# in django.db.transaction

non_atomic_requests                # don't wrap this view into a transaction

# in django.views.decorators.gzip:

gzip_page

# in django.views.decorators.vary:

vary_on_cookie(...)
vary_on_headers(...)

```

## Templates

* if `APP_DIRS` variable is `True` in `TEMPLATES` settings, Django by convention first looks for templates inside `templates` subdirectories in each of the `INSTALLED_APPS`, while the templates inside the directories in `DIRS` variable of `TEMPLATES` settings override subapps' templates
* Django sees template files globally so to avoid name collisions all templates of a subapp are stored in `subappname/templates/subappname`, so in `django.template.loader.get_template("subappname/index.html")` `subappname` refers to the name of a subdirectory in the subapp's `templates` directory and not to the subapp's actual directory
* the extension of a Django HTML template is usually ".html"
* when dot syntax is used on a context variable, the template renderer first tries to do a dictionary key lookup, then attribute lookup, and finally list index lookup
* URL hardcoding can be avoided with `{% url %}` template tag, e.g. `<a href="{% url 'url_route_name' target_view_arg %}">...</a>` or `<a href="{% url 'url_namespace:url_route_name' kwargname=target_view_arg %}">...</a>`

### Django Template Language

* the static content of a template, whether it's HTML code or any other, is interleaved with dynamic content by means of template *variables* and *tags*
* for template variables, the value of a context variable can be placed into a template with `{{ variable_name }}` syntax; template variables can also be used inside tags simply by their names
* in a template, an attribute of a variable (such as a filed) can be accessed and a callable can be called on a variable using the `.` syntax; for callables, no `()` are used and no arguments can be provided; the `.` syntax can be invoked multiple times for the attributes/callables of a variable
* using the same `.` syntax, a dictionary variable can return the value of one of its keys, e.g. `variable_name.key`, and an indexable variable (such as a list) can return one of its elements, e.g. `variable_name.0`
* for variables, the order of lookup that is used for the `.` syntax is: dictionary lookup, attribute or method lookup, numeric index lookup
* template tags typically add business logic to a template; the tag syntax is `{% tag_name %}` or `{% tag_name ...arguments... %}`, e.g. `{% if variable_name > 1 %}` or `{% for element in list_variable_name %}`
* the ending of a tag that logically takes in a portion of static content or code is indicated with `{% end<tag_name> %}`, e.g. `{% endif %}` or `{% endfor %}`; such tags are *block-like*
* single lines can be commented out with `{# ... #}`
* arguments to a tag are separated with spaces
* some of the available tags are:

```python

autoescape     # turns autoescaping `on` or `off`; block-like
block          # identifies a block of code; block-like
comment        # comments out multiple lines; block-like
csrf_token     # used for CSRF protection (in forms)
cycle          # returns one of its arguments each time the tag is encountered
debug          # outputs a whole load of debugging information
extends        # extends the template of the given name (can be a variable)
filter         # filters the contents of the block through one or more filters; block-like
firstof        # outputs the first argument variable that is not False
for            # for-loop similar to that of Python; reverse-able; block-like; sets forloop.<variable>:
  counter      # current index, 1-based
  counter0     # current index, 0-based
  revcounter   # current reverse index, 1-based
  revcounter0  # current reverse index, 0-based
  first        # True if first iteration
  last         # True if last iteration
  parentloop   # reference to the enclosing loop
for ... empty  # in a for-loop, `empty` clause containing what's to be displayed if the iterable is empty
if             # if-statement; clauses: `else`, `elif`; boolean operators: `and`, `or`, `not`;
               #   comparison operators: `==`, `!=`, `<`, `>`, `<=`, `>=`, `in`, `not in`; filter-enabled; block-like
ifchanged      # checks if a value has changed from the last iteration of a loop; block-like
include        # inserts the content of the template of the given name (can be a variable) into the current context
load           # loads one or more template tag sets
lorem          # displays random "lorem ipsum" text
now            # displays the current date and/or time, using a format according to the given string
regroup        # regroups a list of alike objects by a common attribute
spaceless      # removes whitespace between HTML tags; block-like
templatetag    # outputs one of the syntax characters used to compose template tags
url            # reverse-resolves a possibly namespaced URL route, optionally taking positional or keywords arguments;
               #   namespaces are separated by `:`; the output can be stored in a variable using `as` keyword;
               #   rises django.core.urlresolvers.NoReverseMatch on fail
verbatim       # stops the template engine from rendering the contents of this block tag; block-like
widthratio     # calculates the ratio of a given value to a maximum value, and then applies that ratio to a constant
with           # caches a computed variable under another name for repeated use; block-like

```

* more available tags are in `i18n`, `l10n`, and `tz` tag sets
* for modularity and content/code reuse, content/code portions named with `{% block block_name %}` tag in one template can be substituted with the same-named content/code portions present in another template by placing `{% extends "first_template_filename" %}` tag at the very beginning of the second template; this mechanism is known as template inheritance wherein the first template is the parent and the second template is a child
* in template inheritance, if a child template does not define a block, the parent's block is displayed
* in template inheritance, the parent's block can be referenced from a child's block with `{{ block.super }}`
* in template inheritance, any tag set loaded in the parent template is not automatically available in a child template and needs to be reloaded
* in addition to literal strings and variables, `{% include ... %}` tag can be passed an object with a `render` method that accepts a context (to insert a compiled template); additional context variables can be passed to an included template using keyword arguments preceded by `with`, e.g. `{% include "template_filename" with var1=... var2=... %}`; by appending `only` option, all other variables of the current context can be discarded for an included template
* text content in template variables is escaped automatically when rendered, as the concluding step after applying filters, if any
* auto-escaping can be turned off with `autoescape` tag for a portion of content/code or with `safe` filter for a particular variable
* in a template, a context variable can be put through a *filter*, which is analogous to a function, and is delimited from the variable with a `|`, e.g. `variable_name|length`
* a filter can take a single argument, delimited by `:`
* filter arguments are **not** auto-escaped
* numeric filter arguments can be provided unquoted
* some of the available filters are:

```python

add                 # arithmetic addition
addslashes          # adds slashes before quotes
capfirst            # capitalizes the first character of the value
center              # centers the value in a field of the given width
cut                 # removes all occurrences of the given string
date                # formats a date according to the given format
default             # if the value evaluates to False, uses the given default
default_if_none     # if and only if the value is None, uses the given default
dictsort            # takes a list of dictionaries and returns that list sorted by the given key
dictsortreversed    # same, only in the reverse order
divisibleby         # returns True if the value is divisible by the argument
escape              # escapes any special characters in the value, assuming HTML; always applied last
escapejs            # escapes any special characters in the value, assuming JavaScript
filesizeformat      # formats the value like a human-readable file size
first               # returns the first element in a list
floatformat         # rounds a floating-point value to one decimal place or custom quantity
force_escape        # escapes any special HTML characters in the value immediately where applied
get_digit           # returns the requested digit in a number
iriencode           # converts an IRI to a string that is suitable for including in a URL
join                # joins the elements in a list value with the given string
last                # returns the last element in a list
length              # for lists and strings, returns the length of the value
length_is           # returns True if the value's length is the given argument, or False otherwise
linebreaks          # replaces line breaks in a plain text with appropriate HTML, <br /> and <p>
linebreaksbr        # converts all newlines in a plain text to <br />
linenumbers         # displays text with line numbers
ljust               # left-aligns the value in a field of the given width
lower               # converts a string into all lowercase
make_list           # returns the value turned into a list
phone2numeric       # converts a phone number (possibly containing letters) to its numerical equivalent
pluralize           # returns a plural suffix if the value is not 1; by default, the suffix is "s"
pprint              # a wrapper around pprint.pprint
random              # returns a random item from the given list
rjust               # right-aligns the value in a field of the given width
safe                # marks a string as not requiring further HTML escaping prior to output
safeseq             # applies the safe filter to each element of a sequence
slice               # returns a slice of the list, e.g. slice:":5"
slugify             # slugifies the value
stringformat        # formats the value according to the argument, a string formatting specifier
time                # formats a time according to the given format
timesince           # formats a date as the time since the given date
timeuntil           # formats a date as the time until the given date
title               # title-cases the value
truncatechars       # truncates a string if it is longer than the given number of characters
truncatechars_html  # HTML-aware version of truncatechars
truncatewords       # truncates a string after a certain number of words
truncatewords_html  # HTML-aware version of truncatewords
unordered_list      # recursively takes a nested list and returns an HTML unordered list, without <ul>
upper               # converts a string into all uppercase
urlencode           # escapes a value for use in a URL
urlize              # converts URLs and email addresses in text into clickable links
urlizetrunc         # converts URLs and email addresses in text into clickable links, with truncation
wordcount           # returns the number of words
wordwrap            # wraps words at specified line length
yesno               # maps values for True, False, and (optionally) None, to the strings "yes", "no", "maybe", or a custom mapping

# with django.contrib.humanize activated, {% load humanize %}:

apnumber            # for numbers 1-9, returns the number spelled out, otherwise returns the number
intcomma            # converts an integer to a string containing commas every three digits
intword             # converts a large integer to a friendly text representation
naturalday          # for dates that are the current day or within one day, returns "today", "tomorrow" or "yesterday"
naturaltime         # for datetime values, returns a string representing how many seconds, minutes or hours away it was
ordinal             # converts an integer to its ordinal as a string, e.g. "2nd"

```

## Static Files

* static files are collected by Django from subapps with `python manage.py collectstatic`
* static files are placed into the directory specified by `STATIC_ROOT` settings variable as an absolute file system path; the path should normally end in a `/`
* the static files of a subapp are discovered and referenced by Django similar to how templates are discovered; a subapp usually stores its static files in `subappname/static/subappname` directory
* the template tags related to static files are loaded with `{% load staticfiles %}` tag at the top of the template
* static files are referenced with `{% static 'subappname/file.ext' %}`
* unlike with templates, a static file that needs to reference another static file uses a URI that is relative to the referencing file and not to the Django app as a whole

## File Uploads

* file uploads depend on `MEDIA_ROOT` and `MEDIA_URL` settings variables; `MEDIA_ROOT` is the absolute file system path to the parent directory for uploaded files and `MEDIA_URL` is the URL where the files to be served from `MEDIA_ROOT` are to be found
* by default, `MEDIA_ROOT` and `MEDIA_URL` settings variables are not specified; when specified, `MEDIA_URL` must end in a `/`, and `MEDIA_ROOT` normally should too
* when Django handles a file upload via POST with `multipart/form-data`, the file data ends up placed in `request.FILES`
* in `request.FILES` dictionary, a key is the value of the file's `name` (in the ID sense) in the originating upload form and the key's value is a `django.core.files.uploadedfile.UploadedFile`
* some attributes of `django.core.files.uploadedfile.UploadedFile` are `name` (in the file name sense), `size`, `content_type`, `charset`
* for direct access to the data of an uploaded file, `chunks` method of `UploadedFile` returns an iterable with the file's data chunks (for memory efficiency) and `read` method returns the entire data
* Django can save uploaded files automatically when `django.forms.ModelForm` is used
* in a `django.forms.Form`, file inputs are defined with `django.forms.FileField` (or `ImageField`, or other `FileField` subclasses) and uploaded files are stored in `request.FILES` by their field names
* a (model) form gets bound to the file uploads by passing `request.FILES` as an argument after `request.POST` argument
* when an uploaded file comes from a `django.db.models.FileField` (or a subclass) of a `django.forms.ModelForm`, it can be saved with `save` method of `ModelForm` (after `is_valid`) and the destination location will be the subdirectory (of `MEDIA_ROOT`) specified for `upload_to` argument when the field was defined in the form; `upload_to` can also be passed a callable
* `upload_to` argument for `FileField` (or a subclass) supports `strftime` formatting
* when a user uploads a file, Django passes off the file data to an upload handler; one of the default upload handlers is to handle small files by keeping them in memory before saving and the other one is to handle large files by temporarily keeping them on disk
* custom upload handlers can be provided to Django to e.g. enforce user-level quotas, compress data on the fly, render progress bars on the client's side, etc.
* the URL of a file uploaded via `FileField` (or a subclass) is available in `url` attribute of the field; some of the other attributes are `path` (for the URI), `name` (for the path relative to `MEDIA_ROOT`), and `size`
* `django.db.models.fields.files.FieldFile` acts as a proxy for `FileField` (which inherits from `django.core.files.File`)
* `ImageField` inherits all attributes and methods from `FileField`, but also validates that the uploaded object is a valid image; `width` and `height` attributes are available on the field; when defining an `ImageField`, `width_field` and `height_field` arguments can be used to specify fields to be auto-populated; using `ImageField` requires Pillow library

## Sessions

* by default, sessions are enabled for a Django instance and Django stores data associated with a user session on the server side, in the database; the user's client only needs to tell the server the session's ID via cookies
* Django's sessions support depends on `django.contrib.sessions.middleware.SessionMiddleware` being listed in `MIDDLEWARE_CLASSES` settings variable
* in addition to database, available backends are `django.contrib.sessions.backends.file`, `django.contrib.sessions.backends.cache`, `django.contrib.sessions.backends.cached_db`, `django.contrib.sessions.backends.signed_cookies`; default is `django.contrib.sessions.backends.db`
* the sessions' database table is created by the initial `python manage.py migrate` (after migrations are made)
* the default expiration time for a session is 2 weeks
* session data is accessible for reading and writing via `session` attribute of any request object passed as an argument to a view, which is a dictionary-like object
* a key in `request.session` should be a string and the key's value should be a serializeable
* among the methods available on `request.session` are `flush` (deletes the current session data from the session and deletes the session cookie), `set_expiry` (sets the expiration time for the session), `cycle_key` (creates a new session key while retaining the current session data), `clear_expired` (class method; removes expired sessions from the session store), `get_expiry_age`, `get_expiry_date`, `get_expire_at_browser_close`, `set_test_cookie`, `test_cookie_worked`, `delete_test_cookie`
* session dictionary keys that begin with an underscore are reserved for internal use by Django
* by default, Django only saves a session data to the session storage when the session dictionary itself has been modified, but not when an object stored as a value in the dictionary has been modified; a session can be explicitly marked as modified by setting its `modified` attribute to `True`; Django can be told to save session data automatically for every single request with `SESSION_SAVE_EVERY_REQUEST` settings variable
* session-related cookies are sent to the user's client only when a session has been created/modified or `SESSION_SAVE_EVERY_REQUEST` setting is `True`
* Django does not provide automatic purging of expired sessions; it's recommended to call `python manage.py clearsessions` to clear expired sessions on a regular basis, for example as a daily cron job

## Authentication & Authorization

* `django.contrib.auth`, if listed in `INSTALLED_APPS` settings variable, is to take care of both authentication and authorization
* in a Django app, regular users, the app's staff, and superusers (admins) are all represented by a single user model class
* Django comes with a default implementation of the user model in `django.contrib.auth.models.User`; the default user model encapsulates username, password, email, first name, last name, etc.; with the default user model, username is the unique textual identifier ("natural key") for users (and not email)
* in Django, a user is typically distinguished by whether they are active (enabled) or not (disabled) according to `is_active` attribute
* the default user model can be replaced with a custom model, which needs to be done before any migrations, or extended in a number of ways
* non-regular users that are known to a Django app are typically distinguished by their superuser and staff status
* in Django, a user can belong to one or more *groups*
* individual users as well as groups can have *permissions* associated with them; all users in a group are automatically granted the group's permissions
* in shell, a superuser can be created with `python manage.py createsuperuser`
* in code, users can be created with `UserModel.objects.create_user(...)` (unlike an ordinary `create`, `create_user` allows for password hashing)
* in shell, a user's password can be changed with `manage.py changepassword username`
* in code, a user's password can be changed with `set_password` method called on the user's model instance and then saving the instance
* by default, changing a user's password will log out all their sessions
* a user's password can be checked with `check_password` method
* to determine whether Django recognizes i.e. authenticates a user as a previously created user by username (or email) and password, `django.contrib.auth.authenticate` function can be called, which returns either the model instance of the user on success or `None` on failure; `authenticate` does not check if the user is active
* to link an authenticated user to the user client from which the request is originating for the duration of a session, i.e. to make the user's client get authenticated automatically with every request (cookie-based), `django.contrib.auth.login` function can be used, e.g. `django.contrib.auth.login(request, authenticated_user)`
* to log out a user, `django.contrib.auth.logout` function can be used, e.g. `django.contrib.auth.logout(request)`; logging a user out clears all session data
* to determine if the current request is coming from an already logged in user, `request.user.is_authenticated()` can be called
* individual function-based view and class-based views in a whole can be restricted to only logged in users with `django.contrib.auth.decorators.login_required` decorator, which by default depends on `LOGIN_URL` setting and can redirect users to the login URL; the decorator, however, does **not** check if the user is active
* with `django.contrib.auth.decorators.user_passes_test` decorator, views can be restricted to only those users who pass a function-defined test, which may include checks for permissions and other conditions; the decorator does not automatically check if the user is logged in
* with `django.contrib.auth.decorators.permission_required` decorator, views can be restricted to only those users who are granted certain permissions (possibly via groups) indicated as one or more arguments to the decorator
* a user can be added to a group through `groups` many-to-many relationship field on the user's model instance, e.g. `user.groups.add(group)`
* permissions are usually identified by strings in the form "subappname.permissioncodename", where "permissioncodename" often are "verb_subject"
* users and groups can be granted permissions through `user_permissions` and `permissions` many-to-many relationship fields respectively, e.g. `user.user_permissions.add(permission)`
* a permission instance can be obtained with `django.contrib.auth.models.Permission.objects.get(codename="permissioncodename")`
* whether a user or group has a permission can be checked with `has_perm` method providing it "subappname.permissioncodename"
* regardless of how a permission is defined, it is visible and can be referenced globally
* permissions can be defined in the context of a model:

```python

class ModelClass(django.db.models.Model):
    ...
    class Meta:
        permissions = (
            ("permissioncodename", "Permission's description"),
            ...
        )

```

* permissions can also be created dynamically, again, in the context of a model:

```python

content_type = django.contrib.contenttypes.models.ContentType.objects.get_for_model(
    ModelClass)
permission = Permission.objects.create(
    codename="permissioncodename",
    name="Permission's description",
    content_type=content_type)

```

* a call to `has_perm` method is cached on the user/group variable so, for any changes to that permission to become visible, the user/group needs to be re-retrieved
* Django can be told to use a custom user model by specifying `AUTH_USER_MODEL` settings variable, e.g. `AUTH_USER_MODEL = "subapp.UserClass"` (the class would be located in `models.py` of the subapp)
* either it's the default user model that is used by Django or a custom one, `django.contrib.auth.get_user_model` function can be used to obtain the class of the user model
* when defining a relationship field on a model, `django.conf.settings.AUTH_USER_MODEL` can be used to reference the user model
* a custom user model inherits from `django.contrib.auth.models.AbstractBaseUser`
* any custom implementation of the user model must define `USERNAME_FIELD` class attribute set to the name of the field that is to be used as the unique identifier for users, `REQUIRED_FIELDS` class attribute set to a list of the field names that will be prompted for when creating a superuser via `createsuperuser` shell command (unrelated to other user types), `is_active` boolean field (with a default), `get_full_name` method returning a user's "long" name, and `get_short_name` method returning a user's "short" name
* a custom user model should also define a custom manager for its instances; a custom manager should extend `django.contrib.auth.models.BaseUserManager` providing two additional methods: `create_user` and `create_superuser`; some of the methods derived from `BaseUserManager` are `normalize_email`, `get_by_natural_key`, `make_random_password`
* for an instance of a custom user model, `get_username` method should be used to obtain the value of the field nominated by `USERNAME_FIELD`
* `django.contrib.auth.models.PermissionsMixin` mixin can be used for a custom user model to add all the methods and database fields necessary to support Django's permission model
* to fit the Django admin site, a custom user model must define some additional attributes and methods: `is_staff`, `has_perm` (via a mixin), `has_module_perms` (via a mixin); if the custom user model extends `AbstractBaseUser`, a custom `ModelAdmin`-like class would need to be defined; it may be possible to subclass the default `django.contrib.auth.admin.UserAdmin` but it will be needed to override any of the definitions that refer to fields on `django.contrib.auth.models.AbstractUser` that aren't on the custom user model class

## Administration

* an admin is created with `python manage.py createsuperuser`
* the password can be changed with `manage.py changepassword username`
* the admin's site is by default located at `/admin/`
* a database table of a subapp is added to the admin's site by registering the table's model class at the top level of the `admin.py` of the subapp:

```python

django.contrib.admin.site.register(ModelClass)

```

* a model can be registered with a class derived from `django.contrib.admin.ModelAdmin` to customize the model's appearance with the admin class' attributes:

```python

class <ModelClass>Admin(django.contrib.admin.ModelAdmin):
    fields = ["field_name_1", "field_name_2"]  # custom field order
django.contrib.admin.site.register(ModelClass, <ModelClass>Admin)

```

```python

class <ModelClass>Admin(django.contrib.admin.ModelAdmin):
    # organize fields into sections
    # first element of a tuple in the list is section name, second is a dictionary
    fieldsets = [
        (None, {"fields": ["field_name_1"]}),
        ("Section 2 Name", {"fields": ["field_name_2"]}),
    ]
django.contrib.admin.site.register(ModelClass, <ModelClass>Admin)

```

```python

class <ModelClass>Admin(django.contrib.admin.ModelAdmin):
    # the second section is initially collapsed
    fieldsets = [
        (None, {"fields": ["field_name_1"]}),
        ("Section 2 Name", {"fields": ["field_name_2"], "classes": ["collapse"]}),
    ]
django.contrib.admin.site.register(ModelClass, <ModelClass>Admin)

```

* the admin page of a model instance can display related model instances inline, with the help of `django.contrib.admin.StackedInline` or `django.contrib.admin.TabularInline`:

```python

class <RelatedManyModelClass>Inline(django.contrib.admin.TabularInline):
    model = RelatedManyModelClass
    extra = <numextrafields>
class <ModelClass>Admin(django.contrib.admin.ModelAdmin):
    fieldsets = ...
    inlines = [<RelatedManyModelClass>Inline]
django.contrib.admin.site.register(ModelClass, <ModelClass>Admin)

```

* instead of listing every model instance just by its `__str__`, the admin page of a model can show the values of selected fields for every listed model instance as well as the outputs of selected model methods (as pseudo-fields):

```python

class <ModelClass>Admin(django.contrib.admin.ModelAdmin):
    ...
    list_display = ("field_name_1", "field_name_2", "method_name")
django.contrib.admin.site.register(ModelClass, <ModelClass>Admin)

```

* for better display of method-based pseudo-fields of model instances on the admin page of a model, the methods can be given attributes in the model's class such as:

```python

method_name.boolean = True  # if the method's output is bool
method_name.short_description = "Column title"
method_name.admin_order_field = "associated_field_name_for_column_sorting"

```

* filtering and search capabilities can be added to a model's page on the admin site:

```python

list_filter = ["field_name_1", ...]
search_fields = ["field_name_1", ...]

```

* admin model page capabilities include changing list pagination, search boxes, filters, date-hierarchies, column-header-ordering, etc.
* admin site templates are `base_site.html`, `index.html`, etc.
* it's possible to deploy more than one admin site for different audiences

## Testing

* by convention, Django tests are located in `tests.py` of each subapp
* tests are run with `python manage.py test` or `python manage.py test subappname`
* when tests are started, Django creates are temporary database just for this purpose so that tests could run as if the database was initially empty
* the tests in a `tests.py` consist of test cases, which are represented by classes derived from `django.test.TestCase`; by a soft convention, a test case class name ends with `Tests`
* a model or view that is being tested should have its separate test class
* in turn, a test case consists of individual tests, which are methods of the class; by a strong convention, a test method name starts with `test_`
* test method names should be maximally descriptive about what each method is testing
* actual checks in test methods are performed with `assert...` methods, e.g. `self.assertEqual(value1, value2)`, `self.assertContains(response, text, count=..., status_code=...)` for responses, `self.assertQuerysetEqual`, etc.
* in test methods, responses from views can be obtained with `self.client.get` function; some attributes of a response are HTTP `status_code`, `content` of the response, `context` dictionary with context variables that were used by the template
