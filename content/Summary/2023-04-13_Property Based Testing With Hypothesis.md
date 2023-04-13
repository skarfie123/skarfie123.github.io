This is a summary based on [this article](https://semaphoreci.com/blog/property-based-testing-python-hypothesis-pytest).

Rather than specifying test cases, define properties that your functions must fulfil, and let `hypothesis` generate dozens of test cases automatically.

```python
from hypothesis import given, strategies as st

@given(st.integers(), st.integers())
def test_func(a, b):
    # get result
    c = func(a, b)

    # test properties based on the result
    assert c > a
    ...
```

Here `@given(st.integers(), st.integers())` indicates the function must work for all integers as `a` and `b`. `hypothesis` will generate many test cases based on this.

- We can filter out values: `st.integers().filter(lambda n: n != 0)`
- Or specify a range: `st.integers(min_value=1, max_value=100)`
- Or both!

Other usecases:

- fuzzing
    ie. simply make sure your function runs without error for all possible values
```python
def test_func(x)
    func(x)
```
- round tripping
```python
def test_func(x)
    assert x == anti_func(func(x))
```
- gold standard comparison
    eg. to validate a new optimisation
```python
def test_func(x)
    assert old_func(x) == func(x)
```

However, the tests may pass as false positive if you don't fully define the properties.

If in doubt you can also include regular tests.
