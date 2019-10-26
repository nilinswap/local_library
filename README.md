# Local Library

Tutorial follow.

pyenv virtualenv 3.7 ll
pyenv activate ll
pip install -r requirements.txt

## Steps into setting up a simple django app

##### make website folder (which is contextually root folder) by `django-admin start projectname`

    - __init__.py is to treat this directory as a Python package( an entity that can be imported with dotted names.)
    - wsgi.py is to help communicate with the web server.

##### `python3 manage.py startapp catalog` to create a new app.


##### add app in `INSTALLED_APPS` in settings.py.


    - SECRET_KEY. This is a secret key that is used as part of Django's website security strategy. If you're not protecting this code in development, you'll need to use a different code (perhaps read from an environment variable or file) when putting it into production.
    - DEBUG. This enables debugging logs to be displayed on error, rather than HTTP status code responses. This should be set to False on production as debug information is useful for attackers, but for now we can keep it set to True.

##### populate urls.py of project_folder

    - if linked it with urls.py of an app, populate app's urls too.
    - Attaching a general template
        `
        urlpatterns = [
            url(r'^admin/', admin.site.urls),
            path('catalog/', include('catalog.urls')),
            path('', RedirectView.as_view(url='/catalog/', permanent=True)),
        ] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
        `
      for an app name catalog.

##### Test if all works minimally

- python manage.py makemigrations
- python manage.py migrate
- python manage.py runserver
- see an django-known page in 127.0.0.1:8000/

##### Create Model

- decide a model structure. might have to use uml diagrams
- app's model.py needs editing

##### Register Model with Admin

- open catalog's admin.py and paste
     `from catalog.models import Author, Genre, Book, BookInstance
        admin.site.register(Book)`
- One can customise Admin site to advanced. follow (this)[https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Admin_site]



# for admin login
username: swapnil
password: asd