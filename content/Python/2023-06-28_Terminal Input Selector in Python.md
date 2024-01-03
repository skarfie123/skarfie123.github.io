When writing a cli program that needs the user to make a choice, you might want to present a list of options and get a selection. The simplest way might be to print each option and get the user to type the one or the index of the one they want. It's better if they can simply use arrow keys to select instead.

A handy library for doing this is [simple-term-menu](https://pypi.org/project/simple-term-menu/):

```python
from simple_term_menu import TerminalMenu

choices = ["one", "two", "three"]
chosen = choices[TerminalMenu(choices, title="Choose one of the following:").show()]
print(">", chosen)
```
