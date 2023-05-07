The go-to library for adding CLI functionality in python is  the built in `argparse` library. This works really well, but there is a relatively new 3rd party library called `typer` that boasts a number of improvements.

Firstly, with typer, rather than configuring a parser with each argument, you can simply let the it auto-detect the configuration based on your function definition. Type hints will be used for input validation. Required arguments will become positional arguments, and optional arguments will become CLI options.

Take this simple argparse example:

```python
import argparse
from typing import Optional

parser = argparse.ArgumentParser()
parser.add_argument("name")
parser.add_argument("--age", type=int)

def main(name: str, age: Optional[int] = None):
    message = f"Hello {name}"
    if age is not None:
        message = f"{message}, age {age}"
    print(message)

if __name__ == "__main__":
    args = parser.parse_args()
    main(args.name, args.age)
```

With typer it will instead look like this:

```python
from typing import Optional
import typer

def main(name: str, age: Optional[int] = None):
    message = f"Hello {name}"
    if age is not None:
        message = f"{message}, age {age}"
    print(message)

if __name__ == "__main__":
    typer.run(main)
```

You can run both of these with `python app.py NAME --age AGE`.

# Help texts

It's not perfect though, because to add help texts, the syntax gets less simple.

With argparse:

```python
parser.add_argument("name", help="what is your name?")
parser.add_argument("--age", type=int, help="how old are you?")
```

With typer:

```python
from typing_extensions import Annotated
def main(
    name: Annotated[str, typer.Argument(help="what is your name?")],
    age: Annotated[Optional[int], typer.Option(help="how old are you?")] = None,
):
```

However, this is still worth it for the extra functionality you can now get.

# Prompting for input

You can tell typer to prompt for input if an argument was not provided. Simply set the prompt message in the `Option`.

```python
def main(
    name: Annotated[str, typer.Argument(help="what is your name?")],
    age: Annotated[int, typer.Option(prompt="how old are you?")],
):
```

Now if the user did not provide their age via `--age`, they will be prompted to enter it.

With argparse, you would have to make it `Optional`, and implement your own logic to prompt for input when `age is None`.

# Environment Variables

With typer you can specify that an argument can default to a value from an environment variable.

For example:

```python
typer.Argument(help="what is your name?", envvar="NAME")
```

Now you don't have to specify the argument if the `NAME` environment variable is set. If both are provided, the CLI argument takes priority.

With argparse, this is not supported by default. For example, you could do this:

```python
import os
parser.add_argument("name", default=os.environ.get("NAME"))
```

However, this would not raise an error if neither were provided. You'd have to add your own logic to handle when `args.name is None`.

# Explicit app

For more configuration options you can create an instance of the `Typer` class.

``` python
cli = typer.Typer()

@cli.command()
def main(name: str, age: Optional[int] = None):
    ...

if __name__ == "__main__":
    cli()
```

## No args is help

Specify this option to tell typer to print the help text if no arguments are provided. Unfortunately this only works for apps with more than one command  ([GitHub issue](https://github.com/tiangolo/typer/issues/450)).

```python
cli = typer.Typer(no_args_is_help=True)
```

## Short Help Option

Note that by default, typer only registers `--help` to trigger the help text.
To add `-h` you can specify it when creating the typer app:

```python
cli = typer.Typer(context_settings={"help_option_names": ["-h", "--help"]})
```

# Multiple Commands

With the explicit app, you can now easily add multiple commands, simply by adding the decorator to more functions.

``` python
@cli.command()
def greet(name: str, age: Optional[int] = None):
    ...

@cli.command()
def math(num: int):
    print(f"{num}*5={num*5}")
```

With only more than one command registered, now the first positional argument specifies which command to run. Note, I renamed the `main` function to a more appropriate `greet`. You can run this with `python app.py greet NAME --age AGE` or `python app.py math NUM`. The command name is by default the function name, but you can change that by specifying in the decorator eg.

```python
@cli.command("foo")
def bar():
```

This is so much more simple than with argparse where you would have to use the sub parser system, and add your own logic to run the correct function.

## Sub commands in argparse

To illustrate, here is the equivalent to the above:

```python
subparsers = parser.add_subparsers(dest="command")

greet_parser = subparsers.add_parser("greet")
greet_parser.add_argument("name", help="what is your name?")
greet_parser.add_argument("--age", type=int, help="how old are you?")

math_parser = subparsers.add_parser("math")
math_parser.add_argument("num", type=int)

if __name__ == "__main__":
    args = parser.parse_args()
    match args.command:
        case "greet":
            greet(args.name, args.age)
        case "math":
            math(args.num)
        case _:
            parser.print_help()
```

In argparse the way to have multiple commands is using subparsers. Specify the `dest` parameter to store the command entered. This is significantly more boilerplate than with typer.

# Other features

- It includes support for shell autocompletion
- It supports full colour output and more using the `rich` library
