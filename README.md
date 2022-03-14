# Güttli's opinionated Python Tips

# How to start?

If you are new to Python or to programmming, then my recommendation: Buy a book, switch off your PC. Read.

After you learned the basics, this text might help you.

# Avoid

Avoid native GUIs (tkinter, gtk, Qt, ...). Or native mobile apps.

If you need a GUI, then use HTML.

# After the basics

After you learned the basics of Python learn web development:

* http
* html
* css

Javascript is not important. SQL is important.

# IDE

I like PyCharm. See [My PyCharm Introduction](//github.com/guettli/why-i-like-pycharm)

# Testing

## PyTest

Use [pytest](https://docs.pytest.org/en/stable/), and if you use Django, then use [pytest-django](https://pytest-django.readthedocs.io/en/latest/).

Reasons: 

* `assert a == b` is far more easy to read and write than `self.assertEqual(a, b)`.
* If your assertion fails, pytest will show you the values. Example: `assert a == b` fails. Then pytest will show you the value of `a` and `b`. 
* The fixture system is really great. This is much more flexible than `setUp()`. In the old unittest `setUp()` method you tend to create things you finally don't need for all tests. This makes the inner edit-test feedbackloop slower.
* You can avoid TestCase classes. A simple method starting with `def test_...()` is enough.

`pytest -k keyword` is very handy. Just a keyword (or some characters) which are part of the filename or test-function, and only the functions containing this string will get called.

Execute test from your IDE. This way you can jump directly from the nice stacktrace to your beautiful code.

BTW, [pytest caching](https://docs.pytest.org/en/stable/cache.html) allows you the re-run only the failed tests.

[pytest parametrize](https://docs.pytest.org/en/stable/parametrize.html) is handy. It helps you to write concise tests.

## TestCase.setUp() should not be used

This tip applies, if you have not switched to pytest fixtures yet.

The method [TestCase.setUp()](https://docs.python.org/3/library/unittest.html#unittest.TestCase.setUp) gets called for **every** test of this TestCase.

Avoid this method. It is very likely that you just waste time since things get done in this method which is not needed for every test of this TestCase.

If possible, please switch to pytest fixtures.

I have seen code where the setUp() method of a class called `MinimalFooTestCase` was 660 lines long.
This slows down tests, since most test methods of this class don't need all the stuff that gets created during setUp(). Remember that setUp() gets called before every test__method(). So 660 lines get executed before every test_method().

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

## pytest cuts your output

Sometimes pytests cuts your output, and you don't see what you want to see.

One way to work around this: Write your data to a temporary file:

```
    with open('/tmp/x', 'wt') as fd:
        fd.write(json.dumps(data, indent=2))
    assert 0
```

Run your test and then inspect the file `/tmp/x`.

But only add this snippet temporarily, since this is vulnurable to a [symlink race](https://en.wikipedia.org/wiki/Symlink_race)

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

## Mocking works locally, but not in CI?

Mocking in Python exchanges a name. If you patch `foo.utils.my_method`, this might work if you use `from foo.utils import my_method`.
It depends which code was run first. If your call to `mock.path()` was called before `from foo.utils import my_method`, then it works.
But if the import happends before the `patch()`, then it does not work.

This means you test works locally, but in CI the test might fail because in CI the import happened in a previous test.

Imagine you import and use `my_method()` in `caller.py`. The you can patch like this `mock.patch('caller.my_method`, ...)`.

If you test calls `my_method()` several times from different files, then this won't help. The you need to patch the internals of `my_method()` to return the desired result. 

Your options:

* use `utils.my_method()` instead of `my_method()`. Not nice.
* Refactor the implementation of `my_method()` so that you patch something inside it, so that it returns the desired result. Not nice.
* Patch all places which use `my_method()` during your test. Not nice.

Up to now I know no nice way to solve this.

See [Python Docs "Where to patch?"](https://docs.python.org/3/library/unittest.mock.html#where-to-patch)


## Type Annotations

For me it feels much more productive to write tests, compared to write type annotations. I don't think type 
annotations are important. They increase the code size, which means my eyes read more and my brain needs to process more data. 
This increases the cognitive load. With other words type annotations sometimes decreases the readability.


# Web Development

Use Django. Related [Django-Tips](//github.com/guettli/django-tips)

# Avoid "as" imports

Example:

```
import datetime as dt
```

That's possible, but it is confusing. I don't recommend this. If you can type with ten fingers,
then typing "datetime" is fast.


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

# Automatically format your code

[Black](https://black.readthedocs.io/en/stable/):

> By using Black, you agree to cede control over minutiae of hand-formatting. In return, Black gives you speed, determinism, and freedom from pycodestyle nagging about formatting. You will save time and mental energy for more important matters.

> Black makes code review faster by producing the smallest diffs possible. Blackened code looks the same regardless of the project you’re reading. Formatting becomes transparent after a while and you can focus on the content instead.


I use `black -S`. The `-S` options: Don't normalize string quotes or prefixes. Related [Black Docs "Strings"](https://black.readthedocs.io/en/stable/the_black_code_style/current_style.html?highlight=quotes#strings). I prefer single quotes, since they are easier to type.


But if you prefer single quotes to double quotes and a way to configure the process, then [blue](https://blue.readthedocs.io/en/latest/) might your tool.

Related: [darker](https://github.com/akaihola/darker)
> Apply black reformatting to Python files only in regions changed since a given commit. 

Related https://github.com/asottile/pyupgrade

Related https://github.com/asottile/reorder_python_imports


# Iterators are overrated

For me this code is perfectly fine:

```
def my_method(...):
    ret = []
    for foo in ...:
        if ...:
            continue
        ...
        ret.append(...)
    return ret
```

Of course I could return an iterator instead of plain and boring list. But what do I gain?

I think iterators make things more complicated. One reason for this: You can't loop over the iterator
several times.

In general: a list is stateless, an interator is stateful. In most cases the stateless solution
is simpler and more mature.

If you need to long list of items, and handling all data in memory does not work any more, then it
is maybe time to use a [Task Queue](https://www.fullstackpython.com/task-queues.html). This way you 
can split your work into small tasks. This gives you much more power than an iterator.

Finally there are two kind of happy developers: Some are happy because they know fancy methods like [more_itertools.spy()](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.spy) and some developers are happy because they don't need to these fancy methods.

# Avoid map(), filter() and reduce()

Use list- or dict-comprehension instead.

```
# Example List-Comprehension: Remove items from list where are 0:

old_list = [0, 1, 2, 3, 4, 5, 0]
new_list = [item for item in old_list if item != 0]
```

```
# Example: Dict-Comprehension: Remove values which are not True:

old_dict = {'foo': 1, 'bar': 2, 'empty': 0}
new_dict = {k:v for k, v in old_dict.items() if v}
```

# functools.partials()

[functools.partial()](https://docs.python.org/3/library/functools.html#functools.partial) is cool.

You can create new methods which get additional arguments.

In this example we needed to provide an old interface after refactoring. We remove a lot of code
by creating a general method `my_getter()`:

```
def my_getter(foo, bar, my_model):
    ...
    
for foo in ...:
    for bar in ...:
        setattr(MyModel, foo + '_' + bar, property(functools.partial(my_getter, foo, bar)))
```        


# Unicode Symbols

Often you can avoid fancy SVG/PNG icons. You can use the unicode symbols: For example `\N{Lock}` 🔒

# I like classmethod

My rule of thumb: If a method of a class does not need the variable "self", then I use `@classmethod`. I never use `@staticmethod`.

This makes my life easier (reduces cognitive load), since I don't need to think about "a vs b".

Related article: https://medium.com/school-of-code/classmethod-vs-staticmethod-in-python-8fe63efb1797

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

# HTML sanitizing library

[bleach](https://github.com/mozilla/bleach)

# Parsing and Changing HTML

BeautifulSoup, which supports CSS Selectors via [SoupSieve](https://beautiful-soup-4.readthedocs.io/en/latest/index.html#css-selectors)

# Statistical Profiling

[Statistical Profiling](https://github.com/guettli/programming-guidelines#statistical-profiler)

[PyInstrument](https://github.com/joerick/pyinstrument)

# pathlib

[pathlib](https://docs.python.org/3/library/pathlib.html) offers classes representing filesystem paths with semantics appropriate for different operating systems.

I don't use it.

Lately I don't play around with file paths that much.

In the past it was different. But dealing with files is becoming less and less important to me (and in general).

I store data in a database, not in files. 

# Packaging 

Please follow the official and maintained guide: [Packaging Projects](https://packaging.python.org/tutorials/packaging-projects/)

If you use google to find a packaging guide, then you might read outdated and not maintained blog articles.

# Tracing

Standard library module [trace](https://docs.python.org/3/library/trace.html):

```
 python -m trace --trace --ignore-dir=/usr:$VIRTUAL_ENV/lib/ your-script.py
 ```

[hunter](https://github.com/ionelmc/python-hunter) has a cool and simple domain language to filter the lines you want to log.


[viztracer](https://github.com/gaogaotiantian/viztracer). Supports [perfetto](https://perfetto.dev/)

Time Travel Debugging [PyTrace](https://github.com/gleb-sevruk/pycrunch-trace)

[PySnooper](https://github.com/cool-RR/PySnooper) Like `set -x` in the Bash Shell. Or [snoop](https://github.com/alexmojaki/snoop)

[Tracing Python Code with settrace](https://github.com/guettli/tracing-python-code)

# Async http client

I recommend [aiohttp](https://docs.aiohttp.org/en/stable/). Unfortunately there are many old and unmaintained async http solutions. AFAIK aiohttp is the best solution today.

# Avoid to modify sys.path

Don't fiddle with sys.path or PYTHONPATH. It is not needed, if you use the common patterns.

# Docker

If you use `pip` in a Dockerfile, `pip` downloads files from the internet again and again
if you build the container several times. The usual cache method does not work.

Here is a solution how to provide a cache to pip running in a Dockerfile: [Using a pip cache directory in docker builds](https://stackoverflow.com/questions/58018300/using-a-pip-cache-directory-in-docker-builds)

# Plugin System

https://pluggy.readthedocs.io/en/latest/

# Poetry vs Pipenv vs pip-tools

If unsure take pip. If you need more take pip-tools.

# Using Python versions, which are not available for your operating system

If you want to test your software on a Python version which is not available for your
operating system, you can use [pyenv](https://github.com/pyenv/pyenv) to get the right version.

Example: You are running Ubuntu 20.04 which ships with Python 3.8, but you want to test your code
with Python 3.10.

# Related

* [Güttli's opinionated Programming Guidelines](https://github.com/guettli/programming-guidelines)
* [Güttli's opinionated Django Tips](https://github.com/guettli/django-tips)
* [Güttli working-out-loud](https://github.com/guettli/wol)





