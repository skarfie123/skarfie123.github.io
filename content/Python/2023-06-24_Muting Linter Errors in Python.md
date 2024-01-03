Linters and static type checker are great for finding bugs in your code, but sometimes they misunderstand or you have some other reasons for leaving the code as is. In that case you might want to mute the error(s) to allow the linting or type checking to pass, either for your sanity, or to allow CI to pass.

# Flake8[^1]

## Ignore error on a line

```python
example = lambda: 'example'  # noqa: E731
```

## Ignore a specific file

- with a cli arg: `--exclude path/to/file`
- at the top on the file: `# flake8: noqa`

## Ignore in all files

Specify codes with the cli arg: `--ignore=E1,E23,W503`

# Pylint[^2]

With pylint you can ignore based on either the "numerical ID" (eg. `E0633`) or the "symbolic message" (eg. `unbalanced-tuple-unpacking`). I usually prefer the latter, since it is clear to other readers or your future self what you're allowing.

You can also disable a whole category with eg. `E`, group of checks (eg. `typecheck`) or all with `all`.

Use `pylint --list-groups` to get the groups.

## Ignore error on a line

```python
"".blah()  # pylint: disable=E1101
"".blah()  # pylint: disable=no-member
```

## Ignore error on the next line

This is useful if multiple linters are flagging the same line. In that case you can mute pylint this way, and the other one on the same line as the code.

```python
# pylint: disable-next=no-member
"".blah()  # type: ignore[attr-defined]
```

## Ignore errors in a single scope

```python
def test():
    # Disable all the no-member violations in this function
    # pylint: disable=no-member
    ...
```

You can also use `# pylint: enable=no-member` to enable it again before the end of the scope. You can use these comments as many times as you need to affect the specific ranges of code you want.

## Ignore in all files

Use the cli arg: `--disable=no-member`

# MyPy[^3]

## Ignore error on a line

```python
number: int = ""  # type: ignore[assignment]
```

## Ignore a specific file

Add comment at the top of the file

- all errors: `# mypy: ignore-errors`
- specific errors: `# mypy: disable-error-code="truthy-bool, ignore-without-code"`

# Bandit[^4]

You can use either the test id or the test name:

```python
SECRET = "MYSECRET"  # nosec: B105
SECRET = "MYSECRET"  # nosec: hardcoded_password_string
```

[^1]: <https://flake8.pycqa.org/en/3.1.1/user/ignoring-errors.html>
[^2]: <https://pylint.pycqa.org/en/latest/user_guide/messages/message_control.html>
[^3]: <https://mypy.readthedocs.io/en/stable/common_issues.html#spurious-errors-and-locally-silencing-the-checker>
[^4]: <https://bandit.readthedocs.io/en/latest/config.html#exclusions>
