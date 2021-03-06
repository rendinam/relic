# RELIC

[![Jenkins Build Status](https://ssbjenkins.stsci.edu/job/STScI/job/relic/job/master/badge/icon)](https://ssbjenkins.stsci.edu/job/STScI/job/relic/job/master/)

## What is RELIC?

RELIC stands for "Release I Control" and maintains a git project's version information without the need for hardcoding values into the source code. It was designed to aid developers with rapid release cycles.

Just tag your code and get on with with your life. It's that simple.

## License

BSD 3-Clause

## Developing RELIC

```bash
git clone https://github.com/spacetelescope/relic.git
cd relic
python setup.py develop
```

## Incorporating RELIC into your project

**Configure `setup.py`**

The configuration below detects and handles obtaining and/or initializing RELIC. There are three possible scenarios:

1. As a local clone of the repository
2. As a git submodule
3. As an installed package (e.g. site-packages)

Refrain from making changes to this code unless you know what you are doing.

```python
import os
import pkgutil
import sys
from setuptools import setup
from subprocess import check_call, CalledProcessError


if not pkgutil.find_loader('relic'):
    relic_local = os.path.exists('relic')
    relic_submodule = (relic_local and
                       os.path.exists('.gitmodules') and
                       not os.listdir('relic'))
    try:
        if relic_submodule:
            check_call(['git', 'submodule', 'update', '--init', '--recursive'])
        elif not relic_local:
            check_call(['git', 'clone', 'https://github.com/spacetelescope/relic.git'])

        sys.path.insert(1, 'relic')
    except CalledProcessError as e:
        print(e)
        exit(1)

import relic.release

version = relic.release.get_info()
relic.release.write_template(version, 'sample/')

setup(
    name='sample',
    version=version.pep386,
    #...
)
```

### To add RELIC as a submodule (optional)

```bash
git submodule add https://github.com/spacetelescope/relic.git
```

## Additional Requirements

**Configure MANIFEST.in:**

In order to build using a tarball generated by `python setup.py sdist`, you will need to create or edit `MANIFEST.in` to save the `RELIC-INFO` cache. `RELIC-INFO` stores your project's version data in an easy to read JSON array. `[package_path]/version.py` does not need to be packaged with your source distribution either, so it may be omitted.

```
include RELIC-INFO
exclude sample/version.py

# Not recommended: Uncomment below to bundle RELIC during `sdist`
#
#recursive-include relic *
#prune relic/.git
#prune relic/tests
```

**Configure .gitignore:**

As `RELIC-INFO` and `[package_path]/version.py` are generated automatically they should **never** be committed to your repository. To prevent this from happening, append these paths to `.gitignore`:

```
RELIC-INFO
*/version.py
```

## Retreiving Information

A simple reading information from version.py:

```python
>>> from .version import *
>>> print('Sample {0} (Date: {1}) [{2}]'.format(
    __version__,
    __version_date__, 
    __build_status__))
Sample 0.0.6 (Date: 2016-03-25 17:34:17 -0400) [release]
>>>
```

## Version module

Here are a few examples of what `version.py` might look like during different stages of development.

### Standard release tag

```python
# AUTOMATICALLY GENERATED BY 'RELIC':
# * DO NOT EDIT THIS MODULE MANUALLY.
# * DO NOT COMMIT THIS MODULE TO YOUR GIT REPOSITORY

__all__ = [
    '__version__',
    '__version_short__',
    '__version_long__',
    '__version_post__',
    '__version_commit__',
    '__version_date__',
    '__version_dirty__',
    '__build_date__',
    '__build_status__'
]

__version__ = '0.0.6'
__version_short__ = '0.0.6'
__version_long__ = '0.0.6-0-feaf392a
__version_post__ = '0'
__version_commit__ = 'feaf392a'
__version_date__ = '2016-03-25 17:34:17 -0400'
__version_dirty__ = False
__build_date__ = '2016-03-26'
__build_time__ = '17:13:50.997002'
__build_status__ = 'release' if not int(__version_post__) > 0 \
    and not __version_dirty__ \
    else 'development'
```

### Post-commit on tag

```python
# AUTOMATICALLY GENERATED BY 'RELIC':
# * DO NOT EDIT THIS MODULE MANUALLY.
# * DO NOT COMMIT THIS MODULE TO YOUR GIT REPOSITORY

__all__ = [
    '__version__',
    '__version_short__',
    '__version_long__',
    '__version_post__',
    '__version_commit__',
    '__version_date__',
    '__version_dirty__',
    '__build_date__',
    '__build_status__'
]

__version__ = '0.0.6.dev1+gadef541a'
__version_short__ = '0.0.6'
__version_long__ = '0.0.6-1-adef541a'
__version_post__ = '1'
__version_commit__ = 'adef541a'
__version_date__ = '2016-03-26 00:55:27 -0400'
__version_dirty__ = False
__build_date__ = '2016-03-26'
__build_time__ = '17:19:29.884038'
__build_status__ = 'release' if not int(__version_post__) > 0 \
    and not __version_dirty__ \
    else 'development'
```

### Dirty post-commit on tag

```python
# AUTOMATICALLY GENERATED BY 'RELIC':
# * DO NOT EDIT THIS MODULE MANUALLY.
# * DO NOT COMMIT THIS MODULE TO YOUR GIT REPOSITORY

__all__ = [
    '__version__',
    '__version_short__',
    '__version_long__',
    '__version_post__',
    '__version_commit__',
    '__version_date__',
    '__version_dirty__',
    '__build_date__',
    '__build_status__'
]

__version__ = '0.0.6.dev1+gadef541a'
__version_short__ = '0.0.6'
__version_long__ = '0.0.6-1-adef541a-dirty'
__version_post__ = '1'
__version_commit__ = 'adef541a'
__version_date__ = '2016-03-26 00:55:27 -0400'
__version_dirty__ = True
__build_date__ = '2016-03-26'
__build_time__ = '17:20:54.291836'
__build_status__ = 'release' if not int(__version_post__) > 0 \
    and not __version_dirty__ \
    else 'development'
```

