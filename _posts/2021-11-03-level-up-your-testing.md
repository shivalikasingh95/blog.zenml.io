---
layout: post
author: Alex Strick van Linschoten
title: 10 Ways To Level Up Your Testing with Python
description: "A mix of mental and technical skills you can develop to get better
at testing your Python code."
category: tech-startup
tags: tech-startup python testing tooling evergreen
publish_date: November 4, 2021
date: 2021-11-04T00:02:00Z
thumbnail: /assets/posts/testing-glasses.jpg
image:
  path: /assets/posts/testing-glasses.jpg
  # height: 1910
  # width: 1000
---

**Last updated:** February 16, 2022.

There's nothing like working on testing to get you familiar with a codebase.
I've been [working](https://github.com/zenml-io/zenml/pull/118) on
[adding back](https://github.com/zenml-io/zenml/pull/149) in
[some testing](https://github.com/zenml-io/zenml/pull/130) to
[the ZenML codebase](https://github.com/zenml-io/zenml) this past couple of
weeks and as a relatively new employee here, it has been a really useful way to
dive into how things work under the hood.

This being my first time working seriously with Python, there were a few things
that I had to learn along the way. What follows is an initial set of lessons I
took away from the experience.

## 1. One-size-fits-all won't cut it

Looking at things from a higher level, it's important to realize that there are
lots of different approaches that you could take to testing. It's a truism that
you should 'test intent, not implementation', but I imagine that in some
scenarios like for software being deployed on a space shuttle you'd want to
maybe also test the implementation as well.

Similarly, different companies and projects have different needs for testing. If
you're a huge company, testing is a way of ensuring reliability and preventing
catastrophic failures along the way. If you're a small company, where speed of
creation and the pace of development is frantic, having too rigid a set of tests
may actually end up hurting you by stifling your ability to iterate through
ideas and changes quickly.

I found it helped to take a step back early on in my testing to really think
through what I was doing, why I was doing it, and what larger goal it was there
to support.

## 2. 'Don't be that person': testing to crush the spirits of your team

It's worth reiterating the previous remark about testing intent and not
implementation.

If you test every last conditional statement, checking that the code is built in
exactly that specific way, changing anything in the original codebase is going
to become incredibly tiresome. Moreover, your testing library will start to
resemble a kind of byzantine twin replica of your original code.

For preventing this, it helps if everyone in the team is testing as much as they
are writing new code. This way it is just part of the development process and
not a separate add-on from a QA-like team. At ZenML, we're small enough that the
expectation is that if you work on a new feature, you should also be responsible
for writing the tests that go alongside.

## 3. Pytest, O Pytest!

[Pytest](https://docs.pytest.org/en/latest/) is amazing. It has everything you
need to write your tests, is easy to understand, and has great documentation of
even the slightly more niche features. Can you tell I really enjoyed getting to
know this open-source library?

For now, I'll mention some of the really useful combinations of CLI commands
that I found useful.

```bash
# make the test output verbose
pytest tests/ -v

# stop testing whenever you get to a test that fails
pytest tests/ -x

# run only a single test
pytest tests/test_base.py::test_initialization

# run only tests tagged with a particular word
pytest tests/ -m specialword

# print out all the output of tests to the console
pytest tests/ -s

# run all the tests, but run the last failures first
pytest tests/ --ff

# see which tests will be run with the given options and config
pytest tests/ —collect-only

# show local variables in tracebacks
pytest tests/ —showlocals
```

And there are so many more! The flexibility of the CLI tool allows you to be
really nimble and ensures you don't have to hang around for already-passing
tests to run.

## 4. Temp Files & Temp Directory Choice Paralysis

At a certain point I needed to test that certain functions were having side
effects out in the real world of a filesystem. I didn't want to pollute my hard
drive or that of whatever random CI server was running the tests, so then I
started looking around for options for the creation of temporary files and
directories.

It turns out that between the Python standard library, Pytest and some
library-specific features, we're spoiled for choice when it comes for
convenience helpers to create temporary files and directories. Python has
[`tempfile`](https://docs.python.org/3/library/tempfile.html) which is a
platform-agnostic way of creating temporary files and directories. Pytest has
`tmp_path` which you can insert as an argument into your test function and have
a convenience location which you can use to your heart's content. (There are
also
[several other options](https://docs.pytest.org/en/latest/how-to/tmp_path.html#tmp-path)
with Pytest). Then other libraries you're using may have specific testing
capabilities. We use [`click`](https://click.palletsprojects.com/en/8.0.x/) for
our CLI functionality and there's a
[useful convenience pattern](https://click.palletsprojects.com/en/8.0.x/testing/#file-system-isolation)
for running commands from a temporary directory:

```python
def test_something():
   runner = CliRunner()
   with runner.isolated_filesystem():
      # do something here in your new temporary directory
```

## 5. Decorate your way to clearer test code

Pytest has a bunch of helper functions which enhance the test code you already
have. For instance, if you want to wanted to iterate over a series of values and
pass them in as arguments to a function, you can just use the `parametrize`
functionality:

```python
@pytest.mark.parametrize("test_input,expected", [("3+5", 8), ("2+4", 6), ("6*9", 42)])
def test_eval(test_input, expected):
    assert eval(test_input) == expected
```

Note that this would fail because 6x9 does not equal to 42.

If you have a test that you know is failing right now, but you want to put it to
the side for the moment, you can mark it down as being expected to fail with
`xfail`:

```python
@pytest.mark.xfail()
def test_something() -> None:
	# whatever code you have here doesn't work
```

I find it's more useful in this way to get a full sense of which tests aren't
working rather than just commenting them out.

The `mark` method in general is a great way of creating some custom ways to run
your tests. You could — using a `@pytest.mark.no_async_call_required` decorator
— distinguish between tests that take a bit longer to run and tests that are
more or less instantaneous, for example.

## 6. Use `hypothesis` for random arguments

Hypothesis is a Python library to check that functions work the way you think
they do. It works by setting up certain conditions under which the function
should work.

For example, you can say that this function should be able to accept any
`datetime` value without any problem. Instead of trying to come up with a list
of different possible edge cases, hypothesis instead will run (in parallel) a
whole series of values to check that this is actually the case. As the docs
state:

> "It works by generating arbitrary data matching your specification and
> checking that your guarantee still holds in that case. If it finds an example
> where it doesn’t, it takes that example and cuts it down to size, simplifying
> it until it finds a much smaller example that still causes the problem. It
> then saves that example for later, so that once it has found a problem with
> your code it will not forget it in the future."
> ([source](https://hypothesis.readthedocs.io/en/latest/))

These custom ways of testing certain kinds of inputs are called 'strategies',
and it has
[a whole bunch](https://hypothesis.readthedocs.io/en/latest/data.html#core-strategies)
of these to choose from. The ones I most often use are text, integers, decimals
and `datetime`.

## 7. Use `tox` to test multiple versions of Python

[`tox`](https://tox.wiki/en/latest/) allows you to automate running your test
suite through multiple versions of Python. It's likely that your CI process does
this as well, so in order to test that these are passing locally as well, you
can use `tox`. It creates new virtual environments using the versions you
specify and runs your test suite through each of them.

Note that if you're using `pyenv` as your overall Python version manager, you
may have to use something like the following command to make sure that all the
various Python versions are available to `tox`:

```bash
pyenv local zenml-dev-3.8.6 3.6.9 3.7.11 3.8.11 3.9.6
```

The first argument passed in is my development environment in which I usually
work, but the other Python versions / environments are to make those versions
available to `tox`.

## 8. Debug your failing tests with `pdb`

Pytest has a bunch of handy ways of inspecting exactly what's going on at the
point where a test fails. I showed some of those above, where you can show, for
example, whatever local variables were initialized alongside the stacktrace.

Another really useful feature is the `--pdb` flag which you can pass in along
with your CLI command. This will deposit you inside a `pdb` debugging
environment at exactly the moment your test fails. Super useful that we get all
this convenience functionality out of the box with Pytest.

## 9. Linting: before and beyond testing

At ZenML we use [`pre-commit`](https://pre-commit.com/) hooks that kick into
action whenever you try to commit code. (Check out
[our `pyproject.toml` configuration](https://github.com/zenml-io/zenml/blob/main/pyproject.toml)
and
[our `scripts/` directory](https://github.com/zenml-io/zenml/tree/main/scripts)
to see how we handle this!) It ensures a level of consistency throughout our
codebase, ensuring that all
[our functions have docstrings](https://interrogate.readthedocs.io/en/latest/index.html),
for example, or implementing a standard order for `import` statements.

Some of this — the [`mypy`](https://mypy.readthedocs.io/en/stable/index.html)
hook, for example — starts to verge into what feels like testing territory. By
ensuring that functions all have type annotations you sometimes are doing more
than just enforcing a particular coding style. When you add `mypy` into your
development workflow, you get up close and personal with exactly how different
types are passed around in your codebase.

## 10. …and remember, coverage is just a number!

It's always good to have a number to chase. It gives you something to work
towards and a feeling of progress. Tools like [Codecov](https://codecov.io)
offer fancy visualizations of just which parts of your codebase still need some
attention. Automating all this as part of the CI process can highlight when
you've just added a series of features but no accompanying tests.

Bearing all these positives in mind, you should still always remember that your
tests are there to serve your broader goals. If your goal is to rapidly iterate
and create new features, maybe having a goal of 100% test coverage at all times
is an unrealistic expectation. A 100% test coverage does not necessarily mean
your code is bug-free and robust. It just means that you invoked it during the
testing process.

Similarly, different kinds of codebase will have different kinds of test
weightings. We didn't really talk much about the different types of tests (from
unit to integration to usability), but some systems or types of designs will
require more focus on different pieces of this bigger picture.

_Alex Strick van Linschoten is a Machine Learning Engineer at ZenML._
