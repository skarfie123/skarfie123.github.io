The C# compiler is pretty smart. For example, let's say you have a function that returns either a string or null.

```cs
string? GetString()
{
    ...
}

var s = GetString()
```

Then if you try to naively use a string function, it will warn you. Eg. `s.Contains(...)` will generate `warning CS8602: Dereference of a possibly null reference.`. You could solve this with with the [null conditional operator](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-) `?`: `s?.Contains(...)`. Alternatively, you could use an `if` statement to check if it is not null before using it.

```cs
if (s is not null)
{
    s.Contains("Hello");
}
```

Now the compiler is smart enough to know that if that check was true, `s` must really be a string, and it will no longer give the warning.

However, what if you had your own validation function? For example:

```cs
bool CheckString(string? s)
{
    return s is not null && s.Contains("World");
}

if (CheckString(s))
{
    s.Contains("Hello");
}
```

Now the compiler again complains with the same error, despite the function checking the string is not null. This is one of the few cases where we are smarter than the compiler!

One way to solve this, is to use the [null forgiving operator](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-forgiving) `!`: `s!.Contains("Hello");`. Here we assert to the compiler that we are sure `s` will not be null in this case.

However, you would have to use this everywhere that the validation function is used. The null forgiving operator should only be used in rare cases to prevent it becoming a bad habit, as it can be misused.

Instead, we can tell the compiler that anytime this function returns true, the input value must be non-null. We can do this with the [`NotNullWhen`](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.codeanalysis.notnullwhenattribute?view=net-8.0) attribute.

```cs
bool CheckString([NotNullWhen(true)] string? s)
{
    return s is not null && s.Contains("World");
}
```

Now the compiler doesn't complain!

A real example of this is the [`String.IsNullOrEmpty`](https://github.com/dotnet/runtime/blob/5535e31a712343a63f5d7d796cd874e563e5ac14/src/libraries/System.Private.CoreLib/src/System/String.cs#L499-L502) method:

```cs
public static bool IsNullOrEmpty([NotNullWhen(false)] string? value)
{
    return value == null || value.Length == 0;
}
```

You can find several similar attributes [here](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/attributes/nullable-analysis).
