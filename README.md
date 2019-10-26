# Local Library

Tutorial follow.

pyenv virtualenv 3.7 ll
pyenv activate ll
pip install -r requirements.txt

# Test if all works minimally
- python manage.py makemigrations
- python manage.py migrate
- python manage.py runserver
- see a django-known page in 127.0.0.1:8000/

# for admin login
username: swapnil
password: asd