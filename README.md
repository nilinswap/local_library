# Local Library

Tutorial follow.

```bash
pyenv virtualenv 3.7 ll
pyenv activate ll
pip install -r requirements.txt
```

## Steps into setting up a simple django app

##### Make website folder (which is contextually root folder) using `django-admin start projectname`
- __init__.py is to treat this directory as a Python package( an entity that can be imported with dotted names.)
- wsgi.py is to help communicate with the web server.

##### `python manage.py startapp catalog` to create a new app.


##### Add app in `INSTALLED_APPS` in settings.py.
- SECRET_KEY: This is a secret key that is used as part of Django's website security strategy. If you're not protecting this code in development, you'll need to use a different code (perhaps read from an environment variable or file) when putting it into production.
- DEBUG: This enables debugging logs to be displayed on error, rather than HTTP status code responses. This should be set to False on production as debug information is useful for attackers, but for now we can keep it set to True.

##### Populate urls.py of project_folder
- If linked it with urls.py of an app, populate app's urls too.
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
```bash
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```
Open `127.0.0.1:8000` to see the familiar django page. 

##### Create Models

- Decide a model structure. Might have to use uml diagrams.
- App's model.py needs editing. Add this
```python
from django.db import models
from django.urls import reverse

# Used to generateURLs by reversingtheURLpatterns
import uuid

# Required for unique book instances


class Genre(models.Model):
    """Model representing a book genre."""

    name = models.CharField(
        max_length=200, help_text="Enter a book genre (e.g. Science Fiction)"
    )

    def __str__(self):
        """String for representing the Model object."""
        return self.name


class Book(models.Model):
    """Model representing a book (but not a specific copy of a book)."""

    title = models.CharField(max_length=200)

    # Foreign Key used because book can only have one author, but authors can
    # have multiple books
    # Author as a string rather than object because it hasn't been declared yet
    # in the file
    author = models.ForeignKey("Author", on_delete=models.SET_NULL, null=True)

    summary = models.TextField(
        max_length=1000, help_text="Enter a brief description of the book"
    )
    isbn = models.CharField(
        "ISBN",
        max_length=13,
        help_text="""
        13 Character
        <a href="https://www.isbn-international.org/content/what-isbn">
        ISBN number</a>""",
    )

    # ManyToManyField used because genre can contain many books.
    # Books can cover many genres.
    # Genre class has already been defined so we can specify the object above.
    genre = models.ManyToManyField(Genre, help_text="Select a genre for this book")
    language = models.ForeignKey("Language", on_delete=models.SET_NULL, null=True)

    def __str__(self):
        """String for representing the Model object."""
        return self.title

    def get_absolute_url(self):
        """Returns the url to access a detail record for this book."""
        return reverse("book-detail", args=[str(self.id)])


class BookInstance(models.Model):
    """
    Model representing a specific copy of a book
    (i.e. that can be borrowed from the library)."""

    id = models.UUIDField(
        primary_key=True,
        default=uuid.uuid4,
        help_text="Unique ID for this particular book across whole library",
    )
    book = models.ForeignKey(Book, on_delete=models.SET_NULL, null=True)
    imprint = models.CharField(max_length=200)
    due_back = models.DateField(null=True, blank=True)

    LOAN_STATUS = (
        ("m", "Maintenance"),
        ("o", "On loan"),
        ("a", "Available"),
        ("r", "Reserved"),
    )

    status = models.CharField(
        max_length=1,
        choices=LOAN_STATUS,
        blank=True,
        default="m",
        help_text="Book availability",
    )

    class Meta:
        ordering = ["due_back"]

    def __str__(self):
        """String for representing the Model object."""
        return f"{self.id} ({self.book.title})"

```

This model.py is incomplete as it has not yet added Author and Language.
Since Author is not yet defined therefore we can not use Author object and use
string "Author" instead unlike for book in BookInstance.

##### Register Model with Admin

- Open catalog's admin.py and paste
```python
from catalog.models import Author, Genre, Book, BookInstance

admin.site.register(Book)
```
- One can customise Admin site to advanced. Follow [this.](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Admin_site)

##### So much but no page displayed yet..
- So far catalog's urls is empty. Although we have called this from project's urls so add to catalog/urls.py
```python
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

- In views.py, add index function that above url mapped to
```python
from catalog.models import Book, Author, BookInstance, Genre

def index(request):
    """View function for home page of site."""

    # Generate counts of some of the main objects
    num_books = Book.objects.all().count()
    num_instances = BookInstance.objects.all().count()
    
    # Available books (status = 'a')
    num_instances_available = BookInstance.objects.filter(status__exact='a').count()
    
    # The 'all()' is implied by default.    
    num_authors = Author.objects.count()
    
    context = {
        'num_books': num_books,
        'num_instances': num_instances,
        'num_instances_available': num_instances_available,
        'num_authors': num_authors,
    }

    # Render the HTML template index.html with the data in the context variable
    return render(request, 'index.html', context=context)
```
- render() takes template as index.html and insert context into it to return an HTML response.

#### Create a template
    
Create template/ inside catalog(our app) and inside it create base_generic.html with following code
```html
<!DOCTYPE html>
<html lang="en">
<head>
  {% block title %}<title>Local Library</title>{% endblock %}
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
  <!-- Add additional CSS in static file -->
  {% load static %}
  <link rel="stylesheet" href="{% static 'css/styles.css' %}">
</head>
<body>
  <div class="container-fluid">
    <div class="row">
      <div class="col-sm-2">
      {% block sidebar %}
        <ul class="sidebar-nav">
          <li><a href="{% url 'index' %}">Home</a></li>
          <li><a href="">All books</a></li>
          <li><a href="">All authors</a></li>
        </ul>
     {% endblock %}
      </div>
      <div class="col-sm-10 ">{% block content %}{% endblock %}</div>
    </div>
  </div>
</body>
</html>
```
This will be extended for every page hence the name.

- See `load static` block this loads style.css and expect it inside css/ and this path is relative to
STATIC_URL defined in settings.py ( pointed out by `static` block inside latter)

- See `block` and `endblock` block which creates holes for data to be inserted

- See `url` block
    The tag here accepts the name of a path() function called in your urls.py 
and the values for any arguments that the associated view will receive from that
function, and returns a URL that you can use to link to the resource.

#### Books and Authors page

- Using generic class-based list views. 
    - Add following to url_patterns in catalog/urls.py
        ```
        path('books/', views.BookListView.as_view(), name='books'),
        ``` 
    - Add following lines in catalog.views.py
```python
from django.views import generic

class BookListView(generic.ListView):
    model = Book
```
That's it! The generic view will query the database to get all records for the specified model (Book) then render a template located at /locallibrary/catalog/templates/catalog/book_list.html (which we will create below). Within the template you can access the list of books with the template variable named object_list OR book_list (i.e. generically "the_model_name_list").


This awkward path for the template location isn't a misprint — the generic views look for templates in /application_name/the_model_name_list.html (catalog/book_list.html in this case) inside the application's /application_name/templates/ directory (/catalog/templates/).

However this awkwardness can be customised to your comfort. e.g. an alternate way is given below
```python
class BookListView(generic.ListView):
    model = Book
    context_object_name = 'my_book_list'   # your own name for the list as a template variable
    queryset = Book.objects.filter(title__icontains='war')[:5] # Get 5 books containing the title war
    template_name = 'books/my_arbitrary_template_name_list.html'  # Specify your own template name/location
```

This is still incomplete until you describe a detail page(as we are using book.get_absolute_url in above template). so let's do that first.

#### Book detail page

- Add in catalog.urls 
```
re_path(r'^book/(?P<pk>\d+)$', views.BookDetailView.as_view(), name='book-detail'),
```

- Add in views
```
class BookDetailView(generic.DetailView):
    model = Book
```

Also, so if there is no record for the given pk then http404 response is sent by django just alright. 
- Create a file book_detail.html inside templates/catalog/ 
```html
{% extends "base_generic.html" %}

{% block content %}
  <h1>Title: {{ book.title }}</h1>

  <p><strong>Author:</strong> <a href="">{{ book.author }}</a></p> <!-- author detail link not yet defined -->
  <p><strong>Summary:</strong> {{ book.summary }}</p>
  <p><strong>ISBN:</strong> {{ book.isbn }}</p> 
  <p><strong>Language:</strong> {{ book.language }}</p>  
  <p><strong>Genre:</strong> {{ book.genre.all|join:", " }}</p>  

  <div style="margin-left:20px;margin-top:20px">
    <h4>Copies</h4>

    {% for copy in book.bookinstance_set.all %}
      <hr>
      <p class="{% if copy.status == 'a' %}text-success{% elif copy.status == 'm' %}text-danger{% else %}text-warning{% endif %}">{{ copy.get_status_display }}</p>
      {% if copy.status != 'a' %}
        <p><strong>Due to be returned:</strong> {{copy.due_back}}</p>
      {% endif %}
      <p><strong>Imprint:</strong> {{copy.imprint}}</p>
      <p class="text-muted"><strong>Id:</strong> {{copy.id}}</p>
    {% endfor %}
  </div>
{% endblock %}

```
The view will pass it the database information for the specific Book record extracted by the URL mapper. Within the template you can access the list of books with the template variable named object OR book (i.e. generically "the_model_name"). as pointed before, you can always customise.

The one interesting thing we haven't seen before is the function book.bookinstance_set.all(). This method is "automagically" constructed by Django in order to return the set of BookInstance records associated with a particular Book.
And although there is an all(), there is no filter() LOL.

#### Include Pagination

- Add `paginate_by = 10` as an attribute to listview class in views. but this only 
paginates the data. in the sense The different pages are accessed using GET parameters — to access page 2 you would use the URL: /catalog/books/?page=2.

- To include this in html,
add following in base_generic.html

```html
{% block content %}{% endblock %}
  
  {% block pagination %}
    {% if is_paginated %}
        <div class="pagination">
            <span class="page-links">
                {% if page_obj.has_previous %}
                    <a href="{{ request.path }}?page={{ page_obj.previous_page_number }}">previous</a>
                {% endif %}
                <span class="page-current">
                    Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}.
                </span>
                {% if page_obj.has_next %}
                    <a href="{{ request.path }}?page={{ page_obj.next_page_number }}">next</a>
                {% endif %}
            </span>
        </div>
    {% endif %}
  {% endblock %}
```
### For admin login

username: swapnil
password: asd
