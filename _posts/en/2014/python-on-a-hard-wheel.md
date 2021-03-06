# Python on A Hard Wheel

- date: 2014-08-19 16:00
- tags: python

How to build and distribute binary wheels on your Mac for every Mac.

----

Wheel is a distribution file format for Python, which was introduced a few
years ago with [PEP427](http://legacy.python.org/dev/peps/pep-0427/). In
case you have no knowledge about wheel, you should read the PEP. If you are
a fan of Armin Ronacher, you might like to read [Python on Wheels](http://lucumr.pocoo.org/2014/1/27/python-on-wheels/).

The wheel format is designed as a binary package format. I had never tried
`bdist_rpm`, because I don't use Red Hat based systems. I had never tried
eggs, which I believe belongs to the old world. Actually, I had never tried
to upload any binary package to PyPI. When I publish a libary, I publish it
as a source package.

I had [a try on wheel](https://github.com/lepture/mistune/issues/3) recently.
It wasn't a pleasant experience. It takes me too much time to create the
powerful format for a binary library:

```
macosx_10_6_intel.macosx_10_9_intel.macosx_10_9_x86_64.whl
```

If you take a look at [Cython PyPI](https://pypi.python.org/pypi/Cython),
you would find the wheels end with this format. But you can't simply build
the wheel yourself.

---

PyPI currently only allows uploading platform-specific wheels for Windows
and Mac OS X. Linux is not included. But it is still useful to create
wheels for these platforms, better than nothing.

For pure Python, a wheel would be something like:

```bash
# python setup.py bdist_wheel --universal
mistune-0.4-py2.py3-none-any.whl
```

Building wheels for pure Python is easy. Building binary wheels for all Mac
OSX is not. At the very first, I created wheels for [mistune](https://github.com/lepture/mistune):

```
mistune-0.4-cp27-none-macosx_10_4_x86_64.whl
```

It is good. But I've seen the wheel for Cython, Pandas and Numpy. They all
end with the complex filename. WTF. Did I miss something? The PEP describes
the file name format as:

```
{distribution}-{version}(-{build tag})?-{python tag}-{abi tag}-{platform tag}.whl
```

The different ending is platform tag:

1. `mistune` is the distribution name
2. `0.4` is package version
3. `cp27` is python tag
4. `none` is ABI tag
5. `macosx_10_4_x86_64` is platform tag

I've read the source code of `bdist_wheel.py`, it turned out that the platform tag
was generated by `distutils.util.get_platform()`. Why is my platform tag `macosx_10_4_x86_64`?
Why can't I build a `macosx_10_6_intel.macosx_10_9_intel.macosx_10_9_x86_64.whl`?
I've googled a lot. The result was not good enough. After all, I've found what I need,
[Spinning wheels](https://github.com/MacPython/wiki/wiki/Spinning-wheels). In this
very wiki, I've learnt the popular Pythons and their platform tags.


| Python source | Python version | OSX version | ``get_platform()`` |
| --------------|----------------|-------------|--------------------|
| Python.org    | 2.7            | 10.9        | macosx-10.6-intel  |
| System Python | 2.7            | 10.9        | macosx-10.9-intel  |
| Macports      | 2.7            | 10.9        | macosx-10.9-x86\_64 |
| Homebrew      | 2.7            | 10.9        | macosx-10.9-x86\_64 |
| Python.org    | 3.4            | 10.9        | macosx-10.6-intel  |
| Python.org    | 2.7            | 10.7        | macosx-10.6-intel  |
| System Python | 2.7            | 10.7        | macosx-10.7-intel  |

My platform tag is not in the table, because I was using the Python created by
[pyenv](https://github.com/yyuu/pyenv). When I tried the System Python, the
wheel turned out:

```
mistune-0.4-cp27-none-macosx_10_9_intel.whl
```

In the wiki of [Spinning wheels](https://github.com/MacPython/wiki/wiki/Spinning-wheels),
I've learnt the very important idea, a `macosx_10_6_intel` would be compatible
with `macosx_10_9_intel` and `macosx_10_9_x86_64`. In this case, you can simply
rename the filename from `macosx_10_6_intel` to:

```
macosx_10_6_intel.macosx_10_9_intel.macosx_10_9_x86_64
```

> Because having a fat binary includes having x86_64, so is compatible with
> x86_64-only builds. Stuff compiled with the 10.6 SDK should also be
> compatible with stuff built against later SDK versions
> (up to and including 10.9). 

In the wiki [MacPython OSX wheel building](https://github.com/MacPython/wiki/wiki/Wheel-building),
a Travis CI approach is teached to you. You can build the idea wheel with
Travis CI, which is exactly the way [pandas](https://github.com/MacPython/pandas-wheels)
and [numpy](https://github.com/MacPython/numpy-wheels) are using.

---

But what if I want to build the wheels on my own machine? All I need is a
Python with platform `macosx_10_6_intel`. But why did pyenv create the python
with platform tag `macosx_10_4_x86_64`?

The source code of pythonz tells me that `macosx_10_4` is defined by environ
variable `MACOSX_DEPLOYMENT_TARGET`, and `intel` can be created by configure
options `--enable-universalsdk=/ --with-universal-archs=intel` when building
Python. Since I've switched to pyenv, it would be done with the shell profile:

```bash
# bash and zsh
export MACOSX_DEPLOYMENT_TARGET="10.6"
export PYTHON_CONFIGURE_OPTS="--enable-universalsdk=/ --with-universal-archs=intel"
```

Installing python with pyenv:

```
$ pyenv install 2.7.8
```

The compiled python would be `macosx_10_6_intel` now. Check the platform tag:

```python
>>> import distutils.util
>>> print(distutils.util.get_platform())
macosx-10.6-intel
```

You can install as many pythons as you like, such as 3.3.5 and 3.4.1, so that
you can create wheels for Python 3.3 and Python 3.4.

---

The final patch for `setup.py` would make it easy to create powerful Mac
wheels:

```python
try:
    from wheel.bdist_wheel import bdist_wheel

    class _bdist_wheel(bdist_wheel):
        def get_tag(self):
            tag = bdist_wheel.get_tag(self)
            repl = 'macosx_10_6_intel.macosx_10_9_intel.macosx_10_9_x86_64'
            if tag[2] == 'macosx_10_6_intel':
                tag = (tag[0], tag[1], repl)
            return tag

    cmdclass = {'bdist_wheel': _bdist_wheel}
except ImportError:
    cmdclass = {}

setup(
    # ...
    cmdclass=cmdclass,
    # ...
)
```

I would suggest that you use this patch for binary wheel. A simple renaming
is not as good as this one patch. This patch would change the platform tag,
and write the information to the wheel meta:

```
# file: mistune-0.4.dist-info/WHEEL
Wheel-Version: 1.0
Generator: bdist_wheel (0.24.0)
Root-Is-Purelib: false
Tag: cp27-none-macosx_10_6_intel
Tag: cp27-none-macosx_10_9_intel
Tag: cp27-none-macosx_10_9_x86_64
```

But a simple renaming would not add those tags to wheel meta. If you dare
have a look at pandas wheel meta:

```
# file: pandas-0.14.1.dist-info/WHEEL
Wheel-Version: 1.0
Generator: bdist_wheel (0.24.0)
Root-Is-Purelib: false
Tag: cp27-none-macosx_10_6_intel
```

It has no tag for `macosx_10_9_intel` and `macosx_10_9_x86_64`.

This is how I created [wheels for mistune](https://pypi.python.org/pypi/mistune).
I'd try windows later (or never).

```bash
$ python setup.py bdist_wheel upload
```

---

Update: I am using Travis CI to build wheels for mistune now. It will upload
the wheels to GitHub releases.

Checkout <https://github.com/lepture/python-wheels>.
