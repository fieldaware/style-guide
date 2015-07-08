---
layout: page
title: Python
---

# General Tips

* Try to follow [PEP8](https://www.python.org/dev/peps/pep-0008/), except for line length. Use a linter (like [pylint](http://www.pylint.org/)) to enforce them automatically in your code.
* Use full names for variables where possible. For example, choose `job` over `j`, `something_longer` over `sl`, but not `this_variable_name_is_the_longest_name_evar_celery_man`.
* Functions and classes should have at least minimal docstrings. A good guide to write them can be found in [PEP 0257](https://www.python.org/dev/peps/pep-0257/).
* No mutable default params please! No `def foo(bar={'my': 'default'}):` and no `def foo(bar=MyClass()):` (assuming the `MyClass` instance is mutable)
* For methods exposed to the outside world, especially exposed methods that read lists of data from the database, default limits should always be applied to prevent accidental or intentional DOS attacks. Such an example can be found in `webapp/controllers/locations.py:get_locations(..)`. Before its latest rewrite, it did not apply any bounds on the `count` argument, causing severe load issues on web application servers.

# Syntax

## Rules to ease reviews

* Don't leave trailing white spaces.
* Add a new line char at the end of each file.
* Add trailing commas when enumerating.

``` python
from foo import (
    bar,
    baz,              # <--
)
# Or
d = {
    "key1": "value",
    "key2": 123,      # <--
}
```

## Exception handling

* When raising exceptions, try to include enough information to help debugging: what was expected? What actually happened?
* When writing controllers which handle a lot of Exceptions, try to encapsulate the logic within one `try..except`

``` python
try:
    job = get_my_job(job_ref) # Raises some exception when job_ref isn't correct
    job.change_it_somehow('weee') # Raises some other exception just because
    job.delete('never wanted it anyways') # Raises another exception
except MyJobWasntFoundException as e: # Handle each Exception individually with a descriptive name
    log.info(e.message) # Potentially log as error if serious
except JobCantBeAltered as e:
    log.info(e.message)
except JobDeleteFailedException as e:
    log.info(e.message)
```


# ChangeLog

#### 2015.07.08

- Added mention of applying limits

#### 2015.06.22

- Added tip: No mutable default params please

#### 2015.05.20

- Initial Draft
