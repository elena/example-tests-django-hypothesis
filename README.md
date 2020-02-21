
# Hypothesis Testing with Django

Simple primer.

Django web framework: https://github.com/django/django

Hypothesis property-based testing: https://github.com/HypothesisWorks/hypothesis

---

We're assuming:

* You write tests regularly
* You write tests using Django
* You are maybe curious about fuzzy/property testing


We're assuming these things aren't new to you, but if they are, welcome! Thanks for looking in! We're going to be assuming all kinds of knowledge and using plenty of jargon ... so if you want to go and do some more Django Testing background research and maybe trying out some tests so it makes more sense, but by all means stay and have a read if you're curious too! Just in case here are some links to some Django Testing background/research materials:

* https://docs.djangoproject.com/en/3.0/intro/tutorial05/
* https://docs.djangoproject.com/en/3.0/topics/testing/


Cool, hopefully you're comfortable  ...

![I'm ready!](./ready.gif)


### Let's get started


## Django Testing

**Are you sick of writing stupid variable names?**

*sadness*:

        def tests_basic_new(self):
            test_item = Item.add_stock(key="TEST THIS IS A NEW ITEM")


**Do you want to make sure you check *every edge case* in every test?**


*sadness*:

        def tests_basic_new(self):
            test_item = Item.add_stock(key="TEST REALLY LONG TEST CASE WITH FUNNY CHARACTErS ₧ · ₨ · ₩ · ₪ · ₫ · ₭ · ₮ · ₯ · ₹")


#### Example of some Django Tests that Need Help


*example **existings** django test*:

![sadness](./really.gif)

```
from django.test import TestCase

from myproducts.models import Item

class TestItemAddStock(TestCase):

    def tests_basic_add_stock_pass(self):
        test_key = "TEST THIS IS A NEW ITEM"
        test_item = Item.add_stock(key=test_key)

        check_item = Item.objects.get(key=test_key)
        assert check_item.key == test_key


    def tests_basic_add_stock_crazy_name_pass(self):
        test_key = "TEST REALLY LONG TEST CASE WITH FUNNY CHARACTErS ₧ · ₨ · ₩ · ₪ · ₫ · ₭ · ₮ · ₯ · ₹"
        test_item = Item.add_stock(key=test_key)

        check_item = Item.objects.get(key=test_key)
        assert check_item.key == test_key
```

This is just a simplified example and assumes some cool magic in your model method `Item.add_stock()` for which this test makes sense.

So given that: maybe this kind of pattern generally looks familiar. If it does we may know that this is not good. We may know that this must be better.

This is the future and now we can save ourselves some worlds of pain by using Hypothesis to instead generate **strings of craziness** on our behalf.


## Hypothesis


### Install Hypothesis

https://hypothesis.readthedocs.io/en/latest/quickstart.html#installing

    pip install hypothesis[django]




### New Jargon: `strategies`

There is a new bit of hypothesis jargon called: `strategies`.

For your test you pick a `strategy` and then hypothesis will generate a whole bunch of craziness to use based upon this type.

In our case here the `strategy` we're looking for here to replace all that grossness is called `text`. This is at: `hypothesis.strategies.text`

Check out all the available strategies: https://github.com/HypothesisWorks/hypothesis/blob/master/hypothesis-python/src/hypothesis/strategies/__init__.py


#### Then what happens:

Then to quote (https://hypothesis.readthedocs.io/en/latest/)
> It works by generating arbitrary data matching your specification and checking that your guarantee still holds in that case. If it finds an example where it doesn’t, it takes that example and cuts it down to size, simplifying it until it finds a much smaller example that still causes the problem.


### Let's Rewrite this Test!


#### `hypothesis...TestCase`

** :warning: IMPORTANT **

Note that Hypothesis **outright replaces** `django.test.TestCase`.

Instead import:

`from hypothesis.extra.django import TestCase`


#### `strategies`

We also specify our `strategy` like:

`import hypothesis.strategies as st`

`test_key=st.text()`


#### `from_model`


We also have to tell hypothesis explicity which model we want to test using:

`from hypothesis.extra.django.from_model`

`test_item=from_model(Item)`

:warning: That is: don't use `Item.add_stock()` in your test, here use `test_item.add_stock()` instead.


Another gotcha is that our test now needs to have a wrapper and our hypothesis stuff as args:

*before*:

`def tests_basic_add_stock_pass(self):`

*after*:

:warning: `def tests_basic_add_stock_pass(self, test_item, test_key):`

And there's the `@given` wrapper, but hopefully it's obvious what's going on here.


### Go already ...

Our re-written test looks like this:


```
from hypothesis import given
from hypothesis.extra.django import TestCase, from_model
import hypothesis.strategies as st

from myproducts.models import Item

class TestItemAddNew(TestCase):

    @given(test_item=from_model(Item), test_key=st.text())
    def tests_basic_add_stock_pass(self, test_item, test_key):
        test_item = test_item.add_stock(key=test_key)

        check_item = Item.objects.get(key=test_key)
        assert check_item.key == test_key
```

... and profit *(or something)*


```
$ ./manage.py test myproducts.tests

Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 tests in 0.156s

OK
Destroying test database for alias 'default'..
```


![Winning!](./yes.gif)


This is just a basic start, there is way more information about Hypothesis testing.

Official docs: https://hypothesis.readthedocs.io/en/latest/django.html

It can also be useful to refer to the source test app here:

https://github.com/HypothesisWorks/hypothesis/tree/master/hypothesis-python/tests/django/toystore
https://github.com/HypothesisWorks/hypothesis/blob/master/hypothesis-python/tests/django/toystore/models.py
https://github.com/HypothesisWorks/hypothesis/blob/master/hypothesis-python/tests/django/toystore/test_given_models.py

The hypothesis testing for django is very powerful. You can add other models and FKs, there's tremendous support for fixtures.

Django is just python and we actually have all the power of hypothesis testing for all of python, including things like `numpy` and `pandas`.

Many thanks and :sparkling_heart: to contributors everywhere but here especially to all the https://github.com/HypothesisWorks/hypothesis/graphs/contributors
