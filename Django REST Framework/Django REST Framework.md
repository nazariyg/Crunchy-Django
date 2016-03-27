# Django REST Framework (v3.*)

* installed with `pip install djangorestframework`
* helps implement machine- and human-oriented RESTful APIs that are exposed via path-like URLs and operate based on the HTTP protocol's methods for creating, retrieving, updating, and deleting resources that are organized into lists
* most common RESTful actions:
    * **list**: a GET request is sent to the URL of the list to be retrieved
    * **create**: a POST request is sent to the URL of the list in which the resource is to be created
    * **read**: a GET request is sent to the URL of the resource to be retrieved
    * **update**: a PUT request is sent to the URL of the resource to be updated (or list to be created)
    * **delete**: a DELETE request is sent to the URL of the resource (or list) to be deleted
* in how framework's classes are named, the framework refers to "read" as "retrieve" and to "delete" as "destroy"
* the actions can be restricted by means of permissions, including instance-level permissions
* for either of directions, information can be formatted in one of a number of supported formats; the default format is JSON
* the information in the fields of Django model instances is generated from input data and is translated to output data through *serializers*
* serializers inherit from one of the classes in `rest_framework.serializers` and are usually defined in a module named `serializers.py`
* conceptually, a serializer is similar to a Django form; both `is_valid` and `save` methods can be used
* a serializer can explicitly declare fields through which data should flow in and out of the RESTful API, e.g. `rest_framework.serializers.CharField`, and resource creation and updating can be customized with `create` and `update` methods, which are called on saving; `save` method, which calls one of the two methods, can be overridden as well
* a serializer can be created from a model instance for output (`SerializerClass(instance)`), from Python data for input (`SerializerClass(data=data)`), or for updating an instance with input Python data (`SerializerClass(instance, data=data)`); for a serialized model instance, the Python data is located in `data` attribute of the serializer
* multiple model instances can be serialized (for listing) with e.g. `SerializerClass(ModelClass.objects.all(), many=True)`
* Python data with native Python types, such as dictionary and list, is the language of serializers
* Python data is converted into e.g. JSON by *rendering* (`JSONRenderer`) and JSON is converted into Python data by *parsing* (`JSONParser`); before JSON can be parsed with `JSONParser`, it needs to be converted into `django.utils.six.BytesIO` stream; `JSONRenderer` and `JSONParser` are not typically used directly and data formats and content type negotiation are taken care of implicitly by `rest_framework.request.Request` and `rest_framework.response.Response`
* same as with `Form`/`ModelForm` in Django, a serializer class can be defined in association with the fields in a model class by deriving the serializer from `rest_framework.serializers.ModelSerializer` so that no serializer fields need to be declared explicitly and serializer fields are inferred automatically (and so are `create` and `update` methods); the associated model is then indicated in `model` class attribute of `Meta` inner class of the serializer's class and a subset of fields is indicated in `fields` class attribute
* the pretty print of **how** a subclass of `ModelSerializer` has interpreted the fields of the associated model can be seen with `repr(ModelSerializerSubclass())`
* as with Django forms, `is_valid` method can validate input data according to how the serializer's fields are defined
* after validation, `save` method of a serializer (if associated with a model) can save the data as a new model instance or update an existing instance
* `rest_framework.request.Request` class extends the Django's class for requests by being able to parse input data from any supported format and by abstracting parsed data into a single `data` attribute (as opposed to just `POST` attribute) regardless whether the request's method is POST, PUT, or PATCH
* `rest_framework.response.Response` class extends the Django's class for responses by being able to determine automatically what data format is expected by the request's sender and render the serialized data into that format
* the use of `rest_framework.request.Request` and `rest_framework.response.Response` is enabled by means of `rest_framework.decorators.api_view` decorator for function-based views and `rest_framework.views.APIView` base class for class-based views; `api_view` takes as arguments a list of HTTP methods to be supported by the decorated function, e.g. `@api_view(["GET", "POST"])`; at this level of abstraction, however, view functions still need to use conditional branching for how HTTP methods are handled and methods of `APIView`-based views need to be named appropriately to their HTTP methods
* `rest_framework.status` module contains constants for HTTP status codes that can be used to improve readability, e.g. `rest_framework.status.HTTP_400_BAD_REQUEST`
* at a higher level of abstraction compared to `APIView`-based views are class-based views that inherit from `rest_framework.generics.GenericAPIView` and use mixins in `rest_framework.mixins`, e.g. `ListModelMixin`; at this level, view methods are still named according to HTTP methods but inherited methods such as `list`, `create`, `retrieve` etc. are available; what serializer class should be used, how queryset should be formed, and other customizations can be specified by appropriately named class attributes/methods
* at a higher level of abstraction compared to the use of mixins is inheriting from an already mixed-in generic view, e.g. `rest_framework.generics.ListCreateAPIView` or `rest_framework.generics.RetrieveUpdateDestroyAPIView`; at this level, no HTTP-related methods need to be defined explicitly
* the highest level of abstraction is provided by classes in `rest_framework.viewsets`, e.g. `rest_framework.viewsets.ModelViewSet`
* in serializers, reverse relationship fields are not serialized unless explicitly declared
* for serializers, the order of automatic method/logic invocation is roughly as follows:

```

is_valid
  validate_<fieldname>
  validate
  Meta.validators
create/update
save

```

* `is_valid` method can take `raise_exception` argument
* `save` method can take keyword arguments that will be assigned to the same-named model fields
* in addition to a model instance as the first positional argument and data as a keyword argument, a serializer's initializer can take `many` argument to indicate an array, `partial` argument to indicate a partial update, `context` argument to provide additional context
* some of serializer attributes are `request`, `validated_data` (after validation), `instance` (if was provided), `initial_data` (if data was provided)
* the `Serializer` class is itself a type of `Field` and can be used to represent relationships where one object type is nested inside another
* `ModelSerializer` includes simple default implementations for `create` and `update`
* with `ModelSerializer`, `unique_together` validators are generated automatically
