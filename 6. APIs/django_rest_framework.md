# Django Rest Framework

## Installation

```bash
# Create conda env or use pipenv or use venv but I will not list here
pip3 install django
pip3 install djangorestframework

# Add 'rest_framework' in 'INSTALLED_APPS' in settings.py
```

## General
1. Provides a general web page for API
2. Provides generics class and reduce the effort for boilerplate code
3. Allows API to make use of the Object Relation Mapping (ORM) of django
4. Provide useful serialization and deserialization class for django model object


## Useful Class in Framework (Keypoint Recap)
1. `rest_framework.response`
   1. `Response` -> This is different from the `django.http.HttpResponse`, it works with `APIView` decorator to provides a clean API view webpage without any templates
   2. `JsonResponse`
2. `rest_framework.status`
   1. `HTTP_200_OK` -> Status Code 200
   2. `HTTP_201_CREATED`
   3. `HTTP_400_BAD_REQUEST`
   4. `HTTP_401_UNAUTHORIZED`
   5. `HTTP_403_FORBIDDEN`
3. `rest_framework.viewsets`
   1. Available Classes:
      1. `ViewSet` -> Class like django restframework view, contains class method of `create` (POST), `list` (GET), `update` (PUT), `retrieve`(GET), `partial_update`(PATCH), `destroy` (DELETE)
      2. `ModelViewSet` -> Handle CRUD methods for models but must pass `queryset = model` and `serializer_class`
      3. `ReadOnlyModelViewSet` -> No write operation hence only supports GET
   2. Setting up Filtering and Ordering Function in the class-based Views
      1. Declare `ordering_fields` in the `ModelViewSet` (Default: `__all__`) 
      2. Declare `search_fields`  in the `ModelViewSet` 
      3. Add the following code in `settings.py` (Project-level)
        ```python
        REST_FRAMEWORK = {
            'DEFAULT_FILTER_BACKENDS': [
                'rest_framework.filters.OrderingFilter',
                'rest_framework.filters.SearchFilter',
            ]
        }
        ```
      4. Simply pass `ordering=title` to order by `title` and `title=hello` to filter `title` by `hello`
   3. Setting up Pagination in Class-based Views
      1. Add the following code in `REST_FRAMEWORK` in `settings.py` (Project-level)
        ```python
        REST_FRAMEWORK = {
            'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination'
            , 'PAGE_SIZE': 2
        }
        ```
      2. `PAGE_SIZE` indicates number of items per page
4. `rest_framework.decorators`
   1. `APIView` -> Can take in list of args like 'GET', 'POST', 'PUT', 'PATCH', 'DELETE'
   2. `permission_classes` -> Takes in `rest_framework.permissions` for user verification
   3. `throttle_classes` -> Takes in `classes` in `rest_framework.throttling` to perform API Throttling
   4. `action` -> This is a routing decorator that makes a function name as a routing path.
5. `rest_framework.permissions`
   1. `IsAuthenticated` -> Check if user is logged in
   2. `IsAdminUser` -> Check if logged in user is a superuser
6. `rest_framework.authtoken.views`
   1. `obtain_auth_token` -> A view to be added to urlpatterns such as user can obtain `auth_token` by passing `username` and `password` to the url via Form URL Encoded POST method
7. `rest_framework.throttling`
   1. `AnonRateThrottle` -> This can be inherited and set with a `scope` class variable to be set up in `DEFAULT_THROTTLE_RATES` of `REST_FRAMEWORK` inside `settings.py`
   2. `UserRateThrottle` -> This can be inherited and set with a `scope` class variable to be set up in `DEFAULT_THROTTLE_RATES` of `REST_FRAMEWORK` inside `settings.py`
8. `rest_framework.views`
   1. `APIView` -> This is class-based view
9.  `rest_framework.serializers` -> Works well with models to perform serialization, data validation(`is_valid()`) and deserialization (`save()`)
   1. Available Classes:
      1. `Serializer` -> Define the same field as your model class except that for fields, use the class fields from `rest_framework.serializer`
      2. `ModelSerializer` -> To use it, define a inner `Meta` class and set 2 class var `model` and `fields`. 
         1. `model` = `django.db.models.Model` class, `fields` = list of columns fields in model
         2. To rename fields from the models in the serializer, you need to create a new class variable as you do in `Serializer` except that you need add `source` args in the new fields that maps the field name in your model. Lastly, include the new field name as a string in the `fields` class variable of `Meta` inner class
         3. To do calculation logic in serializer, you may create an instance method. 
           Example:
           ```python
           class MyModelSerializer(serializer.ModelSerializer):
               # renamed_field and calculated_value are just arbitrary name here
               renamed_field = serializer.IntegerField(source='one_old_field')
               calculated_value = serializer.SerializerMethodField(method_name = 'calculate_tax')
               class Meta:
                   model = MyModel
                   fields = ['id', 'title', 'price', 'renamed_field', 'calculated_value']
               def calculate_tax(self, product: MyModel):
                   return product.price * 1.1
           ```
         4. To load additional information of related foreign key, you may use `StringRelatedField` to display string representation of foreign object or create another serializer for that class and assign to your current class.
            ```python
            # In views.py (for function based views)
            # This 'select_related' will reduce the number of sql calls when pulling related foreign key items
            mymodels = MyModel.objects.select_related('another_model').all() # This assumes `MyModel` uses `another_model` as a field for foreign key of another model
            serialized_models = MyModelSerializer(mymodels, many=True) # `many` allows serializing list of objects

            # In serializers.py
            class AnotherModelSerializer(serializer.ModelSerializer):
                class Meta:
                    model = AnotherModel
                    fields = ['id', 'another']
            class MyModelSerializer(serializer.ModelSerializer):
                related_model_obj = AnotherModelSerializer()
                class Meta:
                    model = MyModel
                    fields = ['id', 'title', 'price', 'related_model_obj']
            ```
         5. We can perform data validation conditions in `serializer` on top of default fields validation.
            1. In `Meta` inner class, add another new class variable `extra_kwargs`. It is a dictionary with field as key and another dictionary as value. The inner dictionary will declare the check required.
            ```python
            class MyModelSerializer(serializer.ModelSerializer):
                class Meta:
                    model = MyModel
                    fields = ['id', 'title', 'price']
                    extra_kwargs = {'price': {'min_value': 2}}
            ```
            2. On top of it, we can also use `validate_field` method to validate
            ```python
            class MyModelSerializer(serializer.ModelSerializer):
                class Meta:
                    model = MyModel
                    fields = ['id', 'title', 'price']
                def validate_price(self, value):
                    if (value < 2):
                        raise serializers.ValidationError('Price should not be less than 2.0')
            ```
            3. Or we can validate all field together
            ```python
            class MyModelSerializer(serializer.ModelSerializer):
                class Meta:
                    model = MyModel
                    fields = ['id', 'title', 'price']
                def validate_price(self, attrs):
                    if(attrs['price']<2):
                        raise serializers.ValidationError('Price should not be less than 2.0')
                    if attrs['title'].startswith('model'):
                        raise serializers.ValidationError('Title should start with "model"')
                    return super().validate(attrs)
            ```
      3. `HyperlinkedModelSerializer` -> Display related inner objects as their url endpoint link, usually foreign key fields
   2. Available Fields (This is slightly different from `django.db.models`). They also take in `write_only` and `read_only` args to display or not display in the webpage API
      1. `IntegerField`
      2. `SerializerMethodField` -> Use a function to return a calculated value from the function. A **SUPER** useful function. Given you want to store a value into a model but you do not want user to enter the value (We can use view to retrieve some values from our database or fix value from requests). You can use this. If we used `read_only`, the serializer will not add the value into model. If we used `write_only`, the user will need to enter the value.
      3. `StringRelatedField` -> Return the string representation `__str__` of another related object, usually foreign key fields
      4. `HyperlinkedRelatedField` -> Return the url endpoint of the related object, usually foreign key fields
      5. `SlugRelatedField` -> Return string representation of a field `slug_field` of a foreign key object, set `many=True` if the field is a list of foreign key objects
10. `rest_framework.validators` -> Can be passed to `extra_kwargs` in `serializer.Meta` or `serializer` fields or `validators` args in inner `Meta` class of `serializers`
   1. `UniqueValidator` -> Takes in `queryset` as args to ensure a field is unique
   2. `UniqueTogetherValidator` -> Takes in `queryset` and `fields` as args to ensure the fields in `fields` are unique together
11. `rest_framework.generics` -> All requires two class variables `queryset` (`Model.objects.all()`) and `serializer_class`(rest_framework serializers)
   1. Available classes
      1. `ListCreateAPIView` -> (GET, POST) Display resource collection and create a new resource
      2. `RetrieveUpdateAPIView` -> (GET, PUT, PATCH) Display a single resource and replace or partially update it
      3. `RetrieveDestroyAPIView` -> (GET, DELETE) Display a single resource and delete it
      4. `RetrieveUpdateDestroyAPIView` -> (GET, PUT, PATCH, DELETE) Display, replace or update and delete a single resource
      5. `CreateAPIView` -> (POST) Create a new resource
      6. `ListAPIView` -> (GET) Display resource collection
      7. `RetrieveAPIView` -> (GET) Display a single resource
      8. `DestroyAPIView` -> (DELETE) Delete a single resource
      9. `UpdateAPIView` -> (PUT, PATCH) Replace or partially update a single resource
   2. Authentication
      1. To allow only specific user to access the API via certain methods need to update `get_permissions` function
12. `rest_framework.routers`
   1. `SimpleRouter` -> Works like url router manager. You may register your routes in this and assign `urlpatterns = simpleRouterInstance.urls`
   2. `DefaultRouter` -> Different from `SimpleRouter`, it creates an API root endpoint with a trailing slash that displays all your API endpoints in one place
13. `django.shortcuts`
   1. `get_object_or_404` -> Works like dictionary but when it fails return a error message to API web page. It takes in 2 args, model object class and pk.

## Renderers (Allows API to auto return data in JSON/XML/YAML/CSV)
1. Setup
   ```python
   # In settings.py (Project-level)
   REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer', # Built-in (Default)
        'rest_framework.renderers.BrowsableAPIRenderer', # Built-in
        'rest_framework_xml.renderers.XMLRenderer', # pip install djangorestframework-xml
        'rest_framework_csv.renderers.CSVRenderer', # pip install djangorestframework-csv
        'rest_framework_yaml.renderers.YAMLRenderer', # pip install djangorestframework-yaml
        ]
    }
   ```
2. Determined by `Accept` headers/ `format` query parameters to use which renderer to return what type of content
   1. headers
      1. JSON: `Accept: application/json`
      2. BrowsableAPI: `Accept: text/html`
      3. XML: `Accept: application/xml`
      4. CSV: `Accept: text/csv`
      5. YAML: `Accept: application/yml`

## Security/Authorization/Permission Related
1. When receiving inputs from user, we can perform data sanitization
   1. Sanitize data by using external package `bleach`. Calling `bleach.clean(input_str)` will replace `<`,`>` or html tags to prevent them to be run as HTML.
   2. Prevent SQL Injection by not simply replacing sql code with user input variable. Best practice is to avoid running raw SQL. But it is required we can use the built-in variable replacement provided by django. `MenuItem.objects.raw('SELECT * FROM LittleLemonAPI_menuitem LIMIT %s', [limit]) `
2. User Authentication
   1. Add `rest_framework.authtoken` to `INSTALLED_APPS` in `settings.py` (Project-Level)
   2. Add `'DEFAULT_AUTHENTICATION_CLASSES': ('rest_framework.authentication.TokenAuthentication')` to `REST_FRAMEWORK` inside `setting.py` (Project-Level)
   3. Run `python manage.py createsuperuser` to create a root user
   4. Use django admin site to add user
   5. In your `views.py` for function-based view, add `rest_framework.decorators.permission_classes` to the view you want to protect, it also takes in a list of `rest_framework.permissions.IsAuthenticated`
   6. In your `urls.py`, from `rest_framework.authtoken.views` import `obtain_auth_token`
   7. Add `path('api-token-auth', obtain_auth_token)` to `urlpatterns` in `urls.py`
   8. User can simply send a POST request with his/her `username` and `password` in form-url encoded data to obtain the `token` from the response
   9. Remaining request to API endpoints that requires login will send the header `Authorization: Token <Your Token>` with every API calls until you cleared it
3. User Permission Groups
   1. Use django admin site to add user group
   2. In function-based view, we can use the `request` object to get the `user` object. By accessing `request.user.groups.filter(name='someusergroup').exists()`, it will return True if the user belongs to `someusergroup` and we can determine what to return based on the user's permission group.
   3. In class-based view, we can assign a list of `rest_framework.permissions` to a class variable `permission_groups`. Or we can create a `def get_permission(self)` function.
      1. We can actually create customised `rest_framework.permissions` by inheriting from `rest_framework.permissions.BasePermission`
      2. In the class itself, it contains a `message` variable to display when the user do not have the permission.
      3. In the class, we can define two instance function namely `def has_permission(self, request, view)` and `def has_object_permission(self, request, view, obj)`. Note that `has_permission` run before `has_object_permission`. Also `has_object_permission` does not run in `POST` method.
    Example
    ```python
    class CustomPermission(permissions.BasePermission):
        message = "You do not have the access to this API Endpoint"
        def has_permission(self, request, view):
            return request.user.groups.filter(name="custom_group").exists()
        
        def has_obj_permission(self, request, view, obj):
            return True # Always have permisson, else Return False to deny access
    
    class MyView(generics.ListCreateAPIView):
        queryset = MenuItem.objects.all()
        serializer_class = MenuItemSerializer

        # This is one way to set it
        permissions_classes = [IsAuthenticated, CustomPermission]

        # This is another way. NOTE THAT you need to INSTANTIATE when using `get_permissions`
        def get_permissions(self):
            return [IsAuthenticated(), CustomPermission()]
    ```
4. User Groups (Better Management of Permissions)
   1. `django.contrib.auth.models` provides `User` object and `Group` object
   2. However, they do not provide serializer hence you need to create your own serializer for these two classes. At least I could not find it.
   3. To add user into user groups, you can first obtain the `User` object and check if it is already in the group. Then select the group and add it by either using `Group.objects.get(name='group').user_set.add(selected_user)` or `selected_user.groups.add(selected_group)`
    Example
    ```python
    class MyView(generics.ListCreateAPIView):
        user_group_name = 'selected_group_name'
        queryset = User.objects.filter(group__name=user_group_name)
        serializer_class = UserSerializer
        def post(self, request):
            user_data = request.POST
            user_group_name = self.__class__.user_group_name
            selected_user = get_object_or_404(User, pk=user_data['userId'])
            
            if selected_user.groups.filter(name=user_group_name).exists():
                return Response('User is already a {}'.format(user_group_name), status=HTTP_400_BAD_REQUEST)
            
            manager_group = Group.objects.get(name=user_group_name)

            # manager_group.user_set.add(selected_user) # Method 1
            selected_user.groups.add(manager_group) # Method 2
            return Response(UserSerializer(selected_user).data, status=HTTP_200_OK)
    ```
5. API Throttling
   1. Function Based View
      1. Add `rest_framework.decorators.throttle_classes` to the view and it also takes in a list of `rest_framework.throttling`
      2. Add `'DEFAULT_THROTTLE_RATES': {'anon': '2/minute', 'user': '5/minute'}` to `REST_FRAMEWORK` in `settings.py` (Project-Level)
   2. Class Based View
      1. Add `'DEFAULT_THROTTLE_CLASSES' : ['rest_framework.throttling.AnonRateThrottle', 'rest_framework.throttling.UserRateThrottle']` to `REST_FRAMEWORK` in `settings.py` (Project-Level)
      2. Class Based view does not use decorator. Simply create a class variable named `throttle_classes` and set it to a list of the classes in `rest_framework.throttling` such as `AnonRateThrottle`, `UserRateThrottle` or any inherited classes
      3. Add `'DEFAULT_THROTTLE_RATES': {'anon': '2/minute', 'user': '5/minute'}` to `REST_FRAMEWORK` in `settings.py` (Project-Level)
      4. We can perform conditional throttling by overwriting `get_throttles(self)` in view but `throttle_classes` in class will be **REMOVED** here.
        ```python
        class MenuItemsViewSet(viewsets.ModelViewSet):
            queryset = MenuItems.objects.all()
            serializer_class = MenuItemSerializer

            def get_throttles(self):
                if self.action == 'create':
                    throttle_classes = [UserRateThrottle]
                else:
                    throttle_classes = []
                return [throttle() for throttle in throttle_classes]
        ```
   3. Throttle Class can be inherited. The class variable `scope` will be used in the `'DEFAULT_THROTTLE_RATES'` in `REST_FRAMEWORK` inside `settings.py` (Project-Level)
    ```python
    from rest_framework.throttling import UserRateThrottle
    class TenCallsPerMinute(UserRateThrottle):
        scope = 'ten'
    ```


## Useful External HTTP Request Tools
1. curl
2. postman
3. insomnia

## Short Setup Guide on Django Debug Toolbar (Used for debugging the number of sql code execution codes)
1. Run in terminal `pip3 install django-debug-toolbar` to install the python package
2. In `settings.py`, add `debug_toolbar` into the list variable `INSTALLED_APPS`
3. In `settings.py`, add `debug_toolbar.middleware.DebugToolbarMiddleware` into the list variable `MIDDLEWARE`
4. In `urls.py` (project-level), add url mapping to `urlpatterns`. i.e `path('__debug__', include('debug_toolbar.urls'))`
5. In `settings.py`, add a new variable `INTERNAL_IPS = ['127.0.0.1']`

## External Authentication Python Library (Djoser)
1. Run `pip install djoser` in terminal
2. Add `rest_framework.authtoken` to `INSTALLED_APPS` in `settings.py` (Project-Level)
3. In `settings.py`, add `djoser` in `INSTALLED_APPS`. It is **important** to keep `djoser` under `rest_framework` in `INSTALLED_APPS` as `djoser` uses some of the functionality or classes in `rest_framework`.
4. In `settings.py`, add `DJOSER` variable as a dictionary that maps `USER_ID_FIELD` to `username`. With this mapping, you can also choose to use email as the login username when performing user authentication.
5. To have Django admin logim with browsable API view of djoser, add `rest_framework.authentication.SessionAuthentication` in `DEFAULT_AUTHENTICATION_CLASSES` in `REST_FRAMEWORK` in `settings.py`
    ```python
    REST_FRAMEWORK = {
        'DEFAULT_AUTHENTICATION_CLASSES': [
            'rest_framework.authentication.TokenAuthentication' 
            ,'rest_framework.authentication.SessionAuthentication'
        ]
    }
    ```
6. In `urls.py` (Project Level), add the two `djoser` urls to `urlspattern`. `path('auth/', include('djoser.urls'))` and `path('auth/', include('djoser.urls.authtoken'))`
7. These are the endpoints provided by `djoser` -> You need to login in as superuser before able to access these endpoints
   1. /users/ -> Sending GET Request with your token as a superuser will get full list of users, normal user will get their own information. Send POST request to this endpoint with `username`, `password` and `email` will create a new user
   2. /users/me/
   3. /users/confirm/
   4. /users/resend_activation/
   5. /users/set_password/
   6. /users/reset_password/
   7. /users/reset_password_confirm/
   8. /users/set_username/
   9. /users/reset_username/
   10. /users/reset_username_confirm/
   11. /token/login/ -> Returns `auth_token` in response after you send a POST request to this endpoints with your `username` and `password` in form-url encoded
   12. /token/logout/
8.  Send a request with your header `Authorization: Token <auth_token>`  endpoint to access endpoint that requires user logged in
9.  Note that if you are to manipulate the `django.contrib.auth.models.User` class to create directly from database. You need to use `User.set_password` to set the user account password otherwise you will **NOT** be able to authenticate. The reason is that in the `set_password` it will perform a hashing before storing, and during user login, the password submitted by the user will be hashed and compared with *supposed hashed password*

## JSON Web Token Authentication (djangorestframework-simplejwt) (A external plugin of Djoser)
1. Run `pip install djangorestframework-simplejwt`
2. In `settings.py`, add `rest_framework_simplejwt` to `INSTALLED_APPS`
3. In `settings.py`, add `rest_framework_simplejwt.authentication.JWTAuthentication` to `DEFAULT_AUTHENTICATION_CLASSES` in `REST_FRAMEWORK`.
    ```python
    REST_FRAMEWORK = {
        'DEFAULT_AUTHENTICATION_CLASSES': [
            'rest_framework.authentication.Tokenuthentication'
            , 'rest_framework.authentication.SessionAuthentication'
            , 'rest_framework_simplejwt.authentication.JWTAuthentication'
        ]
    }
    ```
4. In `urls.py` (project-level), from `rest_framework_simplejwt.views` import `TokenObtainPairView` and `TokenRefreshView`. Add two views into two endpoints (`urlpatterns`).
   ```python
    urlpatterns = [
        path('admin', admin.sites.url)
        , path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_view')
        , path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh')
    ]
    ```
5. Send a POST request with your `username` and `password` in form-url encoded to `api/token` endpoint and obtain `refresh` token and `access` token.
   1. `access` token expires after 5 minutes. It is sent as a header `Authorization: Bearer <ACCESS TOKEN>`
   2. `refresh` token is used to regenerate the `access` token
      1. How to refresh token?
      2. Send a POST request with your `refresh` as key and `refresh` token as value in form-url encoded to `api/token/refresh` endpoint and obtain **ONLY** the `access` token
   3. It is **important** that we need to blacklist the `refresh` token when we are no longer going to use it.
6. In `settings.py`, create variable `SIMPLE_JWT` as a dictionary and add key value pair of `'ACCESS_TOKEN_LIFETIME'` as key and `timedelta(minutes=5)` as value.
7. JSON Web Token Blacklist Refresh Token
   1. In `settings.py`, add `rest_framework_simplejwt.token_blacklist` to `INSTALLED_APPS`
   2. Run `python manage.py migrate` after Step 1.
   3. In `urls.py` (project-level), from `rest_framework_simplejwt.views` import `TokenBlacklistView`. Add this view to an endpoint.
    ```python
    urlpatterns = [
        path('admin', admin.sites.url)
        , path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_view')
        , path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh')
        , path('api/token/blacklist/', TokenRefreshView.as_view(), name='token_blacklist')
    ]
    ```
8. For subsequent requests, to access endpoints that requires authentication, simply set the headers `Authorization: Bearer < Token>`