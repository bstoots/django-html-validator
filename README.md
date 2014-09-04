django-html-validator
=====================

A tool to do validation of your HTML generated from your Django app.

License: [MPL 2](http://www.mozilla.org/MPL/2.0/)


Warning!
--------

If you don't download a local `vnu.har` file, it will use
`http://html5.validator.nu` and send your HTML there over HTTP.
Not over HTTPS.

If you use `htmlvalidator` to validate tests it's unlikely your HTML contains
anything sensitive or personally identifyable but if you use the middleware
option there's a potential risk.

Settings
--------

Independent of how you use `htmlvalidator` you need to switch it on.
It's not on by default. The setting to do that is:

```python
HTMLVALIDATOR_ENABLED = True
```

What this does, is that it prints all validation errors to `stdout`.
But it doesn't stop the execution from running. Even if there are errors.

To make it so that the execution stops as soon as there is any validation
error switch this on in your settings:

```python
HTMLVALIDATOR_FAILFAST = True
```

Now, if there's any validation error going through the client you'll
get a `htmlvalidator.exceptions.ValidationError` exception raised.


Setting the vnu.jar path
------------------------

By default, all validation is done by sending your HTML with HTTP POST to
[html5.validator.nu](http://html5.validator.nu/).

Not only does this put a lot of stress on their server. Especially if you have
a lot of tests. It's also slow because it depends on network latency. A much
better way is to download the `vnu.jar` file from their
[latest release](https://github.com/validator/validator.github.io/releases/latest) on
[GitHub page](https://github.com/validator/).

You set it up simply like this:

```python
HTMLVALIDATOR_VNU_JAR = '~/downloads/vnu.jar'
```

This also **requires java to be installed** because that's how `.jar` files are
executed on the command line.

Be aware that calling this `vnu.jar` file is quite slow. Over 2 seconds is
not unusual.


Validating during running the server
------------------------------------

A way to do HTML validation is to do it during running the
server. E.g. with `./manage.py runserver`.

To do that you need to enable the middleware. In your settings module,
in the `MIDDLEWARE_CLASSES` add
`htmlvalidator.middleware.HTMLValidator` so it looks something like
this:

```python
MIDDLEWARE_CLASSES = (
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'htmlvalidator.middleware.HTMLValidator',  # <-- note!
)
```

Note! See the note above about `HTMLVALIDATOR_ENABLED` to actually
make it do something.
Also, if you enable `HTMLVALIDATOR_FAILFAST`, when running the
`htmlvalidator` middleware it will raise an exception as soon as it
sees some invalid HTML.


Validating HTML in tests
------------------------

Suppose you have a class that does tests. By default it already has a
`self.client` which you use to make requests. All you need to do is to
replace it with the `htmlvalidator.client.ValidatingClient`
class. For example:

```python

from django.test import TestCase
from htmlvalidator.client import ValidatingClient


class MyAppTests(TestCase):

    def setUp(self):
        super(MyAppTests, self).setUp()
        self.client = ValidatingClient()

    def test_homepage(self):
        response = self.client.get('/')
        self.assertEqual(response.status_code, 200)
```
