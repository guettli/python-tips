# Güttli's opinionated Python Tips

# How to start?

My recommendation: Buy a book, switch off your PC. Read.

# Avoid

Native GUI (tkinter, gtk, Qt, ...)

# IDE

I like PyCharm. See [My PyCharm Introduction](//github.com/guettli/why-i-like-pycharm)

# Testing

## PyTest
Use [pytest](https://docs.pytest.org/en/stable/), and if you use Django, then use [pytest-django](https://pytest-django.readthedocs.io/en/latest/).

Reason: `assert a==b` is far more easy to read and write than `self.assertEqual(a, b)`.

`pytest -k keyword` is very handy. Just a keyword (or some characters) which are part of the filename or test-function, and only the functions containing this string will get called.

Execute test from your IDE. This way you can jump directly from the nice stacktrace to your beautiful code.

BTW, [pytest caching](https://docs.pytest.org/en/stable/cache.html) allows you the re-run only the failed tests.

[pytest parametrize](https://docs.pytest.org/en/stable/parametrize.html) is handy. It helps you to write concise tests.

## Directory/File Layout

Concerning tests, I like this layout:
```
setup.py
myapp/utils.py
myapp/utils_test.py
myapp/conftest.py
...
```

[conftest.py](https://docs.pytest.org/en/latest/writing_plugins.html#conftest-py-plugins) is for configuring pytest.

This layout follows the [LoB (Locality of Behaviour)](https://htmx.org/essays/locality-of-behaviour/) Prinicple. 

I even use a small test which ensures that for every python file, there is a correspondig `..._test.py` file. Of course the is a small exclude list, but
nevertheless this test helps and reminds me to write tests.



## pytest-xdist

[pytest-xdist](https://github.com/pytest-dev/pytest-xdist)

> The pytest-xdist plugin extends pytest with some unique test execution modes:
> 
> test run parallelization: if you have multiple CPUs or hosts you can use those for a combined test run. This allows to speed up development or to use special resources of remote machines.
> 
> --looponfail: run your tests repeatedly in a subprocess. After each run pytest waits until a file in your project changes and then re-runs the previously failing tests. This is repeated until all tests pass after which again a full run is performed.
> 
> Multi-Platform coverage: you can specify different Python interpreters or different platforms and run tests in parallel on all of them.

## Coverage

[Coverage](https://coverage.readthedocs.io/) is a handy tool to check if most of your code is tested.

If you have a huge code base, and you only care for a small part, you can do this:

```
# run only tests matching this pattern and collect coverage data:
coverage run -m pytest -k job

# Only create the coverage report for files which match this pattern:
coverage html --include '*job.py'

# Open browser with the created index.html:
run-mailcap htmlcov/index.html 
```

Let tests fail, if coverage is below PERCENT. I use 85 to 95.
```
pytest --cov-fail-under=PERCENT
```

## Coverage with Context

With [Contexts](https://coverage.readthedocs.io/en/latest/contexts.html) coverage can answer you the question "What test ran this line?"

## Output in Tests is cut

The output gets cut by pytest, if it is too long. You want to see the whole data instead of `...`?

You can use a debugger, set a breakpoint and inspect the current state of the local variables.

Or you can help yourself by temporary adding this snippet to your test. For example you want to
see the value of `response.content`. Because the tools like IDE provide so many cool features, it is
easily forgotten to use the basics. This creates a file `/tmp/o.html`.

```Python
    with open('/tmp/o.html', 'wb') as f:
        f.write(response.content)
```


## Freezegun
[Freezegun](https://pypi.org/project/freezegun/)

> FreezeGun is a library that allows your Python tests to travel through time by mocking the datetime module.

```
@freeze_time("2012-01-14")
def test():
    assert datetime.datetime.now() == datetime.datetime(2012, 1, 14)
```

## Raising an Exception in a mock

You want to raise an excepion in a mock. First solution: do `raise Exception()` in a lambda, since you
don't want create a new method. Then you realize that this is not possible.

Solution: [Mock.side_effect()](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.side_effect)

## Type Annotations

For me it feels much more productive to write tests, compared to write type annotations. I don't think type 
annotations are important. They increase the code size, which means my eyes read more and my brain needs to process more data. 
This increases the cognitive load. With other words type annotations sometimes decreases the readability.


# Web Development

Use Django. Related [Django-Tips](//github.com/guettli/django-tips)

# Use Virtualenv

Virtualenv is a great tool to get isolated environments. It is very light-weighted and
I almost always use it.

I avoid to develop in Docker, virtual machines or Vagrant.

If a database is needed, then I usualy set it up on my local machine.

If the application needs a lot of servers (redis, solr, s3, ...) then I create 
containers to provide the service. Nevertheless during development 
my code runs directly on
my local machine, not inside a container or VM.

If the application is a web application (for example with Django), I use **http**
server (like `manage.py runserver`) and access the application this way. I don't
set up a https server for development. Serving the application via https is
only needed for the production environment, not for development.

This way I can easily run and debug my code.

I know that some IDEs have plugins to connect to vagrant/docker/ssh, but I avoid this
for daily development. I want a fast edit/test loop.

See "How do you develop for the cloud?" in [Python Developer Survey](https://www.jetbrains.com/lp/python-developers-survey-2020/): Most
people develop locally with virtualenv.

# pre-commit.com

[reorder_python_imports](https://github.com/asottile/reorder_python_imports) (instead of isort)

# Automatically format your code: Black

[Black](https://black.readthedocs.io/en/stable/):

> By using Black, you agree to cede control over minutiae of hand-formatting. In return, Black gives you speed, determinism, and freedom from pycodestyle nagging about formatting. You will save time and mental energy for more important matters.

> Black makes code review faster by producing the smallest diffs possible. Blackened code looks the same regardless of the project you’re reading. Formatting becomes transparent after a while and you can focus on the content instead.


I use `black -S`. The `-S` options: Don't normalize string quotes or prefixes.

# Unicode Symbols

Often you can avoid fancy SVG/PNG icons. You can use the unicode symbols: For example `\N{Lock}` 🔒

# Detect confusable Unicode Characters.

There a many [confusable Unicode characters](https://util.unicode.org/UnicodeJsps/confusables.jsp)

To detect them you can use this:
```
>>> 'TEST_DАТА_MANAGEMENT_ACCOUNT'.encode()
b'TEST_D\xd0\x90\xd0\xa2\xd0\x90_MANAGEMENT_ACCOUNT'
```
More details:
```
>>> from unicodedata import name
>>> for char in 'TEST_DАТА_MANAGEMENT_ACCOUNT':
...     print(name(char))
... 
LATIN CAPITAL LETTER T
LATIN CAPITAL LETTER E
LATIN CAPITAL LETTER S
LATIN CAPITAL LETTER T
LOW LINE
LATIN CAPITAL LETTER D
CYRILLIC CAPITAL LETTER A
CYRILLIC CAPITAL LETTER TE
CYRILLIC CAPITAL LETTER A
LOW LINE
...
```

# Statistical Profiling

[Statistical Profiling](https://github.com/guettli/programming-guidelines#statistical-profiler)

[PyInstrument](https://github.com/joerick/pyinstrument)

# Tracing

[Eliot](https://github.com/itamarst/eliot) is a loggin/tracing tool which shows you the call tree. It is not an config-free solution. You need to modify your code 
for creating tracing-events.

Or just use the module [trace](https://docs.python.org/3/library/trace.html):

```
 python -m trace --trace --ignore-dir=/usr:$VIRTUAL_ENV/lib/ your-script.py
 ```

[hunter](https://github.com/ionelmc/python-hunter) has a cool and simple domain language to filter the lines you want to log.


[viztracer](https://github.com/gaogaotiantian/viztracer)

# Async http client

I recommend [aiohttp](https://docs.aiohttp.org/en/stable/). Unfortunately there are many old and unmaintained async http solutions. AFAIK aiohttp is the best solution today.

# Avoid to modify sys.path

Don't fiddle with sys.path or PYTHONPATH. It is not needed, if you use the common patterns.

# Docker

If you use `pip` in a Dockerfile, `pip` downloads files from the internet again and again
if you build the container several times. The usual cache method does not work.

Here is a solution how to provide a cache to pip running in a Dockerfile: [Using a pip cache directory in docker builds](https://stackoverflow.com/questions/58018300/using-a-pip-cache-directory-in-docker-builds)

# Related

* [Güttli's opinionated Programming Guidelines](https://github.com/guettli/programming-guidelines)
* [Güttli's opinionated Django Tips](https://github.com/guettli/django-tips)
* [Güttli working-out-loud](https://github.com/guettli/wol)





