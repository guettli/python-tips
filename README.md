# Güttli's opinionated Python Tips

# Testing

Use [pytest](https://docs.pytest.org/en/stable/), and if you use Django, then use [pytest-django](https://pytest-django.readthedocs.io/en/latest/).

Reason: `assert a==b` is far more easy to read and write than `self.assertEqual(a, b)`.

`pytest -k keyword` is very handy. Just a keyword (or some characters) which are part of the filename or test-function, and only the functions containing this string will get called.

Execute test from your IDE. This way you can jump directly from the nice stacktrace to your beautiful code.

# Web Development

Use Django.

# Related

* [Güttli's opinionated Django Tips](https://github.com/guettli/django-tips)
* [Güttli working-out-loud](https://github.com/guettli/wol)





