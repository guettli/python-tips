# Güttli's opinionated Python Tips

# Testing

Use [pytest](https://docs.pytest.org/en/stable/), and if you use Django, then use [pytest-django](https://pytest-django.readthedocs.io/en/latest/).

Reason: `assert a==b` is far more easy to read and write than `self.assertEqual(a, b)`.

`pytest -k keyword` is very handy. Just a keyword (or some characters) which are part of the filename or test-function, and only the functions containing this string will get called.

Execute test from your IDE. This way you can jump directly from the nice stacktrace to your beautiful code.

BTW, [pytest caching](https://docs.pytest.org/en/stable/cache.html) allows you the re-run only the failed tests.

# Web Development

Use Django. Related [Django-Tips](//github.com/guettli/django-tips)

# Use Virtualenv

Virtualenv is a great tool to get isolated environments. It is very light-weighted and
I almost always use it.

I avoid to develop in Docker, virtual machines or Vagrant.

If a database is needed, then I usualy set it up on my local machine.

If the application needs a lot of servers (redis, solr, s3, ...) then I set up on
or server containers to provide the service. Nevertheless during development 
my code runs directly on
my local machine, not inside a container or VM.

If the application is a web application (for example with Django), I use **http**
server (like `manage.py runserver`) and access the application this way. I don't
set up a https server for development. Serving the application via https is
only needed for the production environment, not for development.

This way I can easily run and debug my code.

I know that some IDEs have plugins to connect to vagrant/docker/ssh, but I avoid this
for daily development. I want a fast edit/test loop.

# Automatically format your code: Black

[Black](https://black.readthedocs.io/en/stable/):

> By using Black, you agree to cede control over minutiae of hand-formatting. In return, Black gives you speed, determinism, and freedom from pycodestyle nagging about formatting. You will save time and mental energy for more important matters.

> Black makes code review faster by producing the smallest diffs possible. Blackened code looks the same regardless of the project you’re reading. Formatting becomes transparent after a while and you can focus on the content instead.

# Statistical Profiling

[Statistical Profiling](https://github.com/guettli/programming-guidelines#statistical-profiler)


# Related

* [Güttli's opinionated Programming Guidelines](https://github.com/guettli/programming-guidelines)
* [Güttli's opinionated Django Tips](https://github.com/guettli/django-tips)
* [Güttli working-out-loud](https://github.com/guettli/wol)





