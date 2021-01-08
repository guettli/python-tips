# Güttli's opinionated Python Tips

# Testing

Use [pytest](https://docs.pytest.org/en/stable/), and if you use Django, then use [pytest-django](https://pytest-django.readthedocs.io/en/latest/).

Reason: `assert a==b` is far more easy to read and write than `self.assertEqual(a, b)`.

`pytest -k keyword` is very handy. Just a keyword (or some characters) which are part of the filename or test-function, and only the functions containing this string will get called.

Execute test from your IDE. This way you can jump directly from the nice stacktrace to your beautiful code.

# Web Development

Use Django. Related [Django-Tips](//github.com/guettli/django-tips)

# Use Virtualenv

Virtualenv is a great tool to get isolated environments. It is very light-weighted and
I always use it.

I avoid to develop in Docker, virtual machines or Vagrant.

# Automatically format your code: Black

[Black](https://black.readthedocs.io/en/stable/):

> By using Black, you agree to cede control over minutiae of hand-formatting. In return, Black gives you speed, determinism, and freedom from pycodestyle nagging about formatting. You will save time and mental energy for more important matters.

> Black makes code review faster by producing the smallest diffs possible. Blackened code looks the same regardless of the project you’re reading. Formatting becomes transparent after a while and you can focus on the content instead.


# Related

* [Güttli's opinionated Django Tips](https://github.com/guettli/django-tips)
* [Güttli working-out-loud](https://github.com/guettli/wol)





