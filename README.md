# CKV File

Clium key value file.

# Aim

Aim of CKV file is to provide a simple, human readable, key value pair storage file that promotes reusability over multiple CKV files and lets the consumer associate meta data with the keys.

# Example

A simple example:

```ckv
// Import key-val pairs from other ckv files into this file
import "path/to/other/file.ckv"

// This is a comment

/*
This is a multi-line comment
*/

THIS_IS_A_KEY =
  After a tab, starts the value
  Value can be spanned across multiple lines.
  Every tabbed line in continuation is part of value of THIS_IS_A_KEY.


#[some_attribute(nested_attribute)]
ATTRIBUTE_EXAMPLE_KEY =
  ATTRIBUTE_EXAMPLE_KEY has meta data associated to it
  in the form of attributes. Attributes are defined by the parser implementing
  the CKV file.
  For example,
  clium defines an attribute shell which specifies the shell that will execute the value. (`#[shell(zsh)]`)


// This is an inline key-value
XYZ = abc
```

# Example from a clium CKV file

```ckv
import "general.ckv"::*

#[!use(std/macros)]

#[protected]
CC = gcc

OPEN =
  nvim [FILE_TO_OPEN]

// Value of boilerplate doesn't use any macro
#[unuse(std/macros)]
BOILERPLATE =
  general.c

#[use(var(CC))]
COMPILE =
  !CC [INSTANCE_PATH] -o [OUTPUT_DIR]/[INSTANCE].out

EXECUTE =
  [OUTPUT_DIR]/[INSTANCE].out
```

# Key-Value

```ckv
KEY =
  VALUE
  VALUE CONTINUED
```

- Key and values are the building blocks of ckv file. A letter in the key matches the following character class: `[0-9a-zA-Z_-]`. 
Following the key, there should be an equal-to (=).
- The next line immediately after the key must be started with a tab. Following the tab is the value of the key.
- If the next line also starts with a tab, it will be a part of key's value and this will go on until a following line doesn't start with a tab or `----` (4 hypens).
- 4 hypens (`----`) in the beginning of the line state that the current line is a continuation of the previous line and a newline character should   not be added to the end of the previous line.

```ckv
KEY =
    An apple a day,
----keeps the doctor away.
    So, I eat apples every day
```

Value for `KEY` should be:
```
An apple a day,keeps the doctor away.
So, I eat apples every day
```

# Inline key-value pair

```ckv
KEY1 = Value1

KEY2 =
    Value2
```

Above, `KEY1` has the value `Value1`. It is declared as an inline key-value pair. These cannot be extended to multiple lines.

# Multiple keys with same name

```
KEY1 = 
    Value1

KEY2 =
    Value2

KEY1 =
    Value3
```

It depends on the implementation how it will decide the value of `KEY1`. This should generally be done by attributes. Some possibilities are to append, overwrite, choose the first one, raise an error, etc. The implementation must not leave this as an undefined behaviour.

# Importing keys from other CKV files

```ckv
import "path/to/other/file.ckv"::{KEY1, KEY2, KEY3}

KEY4 =
    Value1
    
KEY2 =
    Value2
```

- Above, we imported `KEY1`, `KEY2` and `KEY3` from `file.ckv`. Upon importing, these should act as if they were defined in the same file on the exact position where the `import` statement was called and in the exact order in which they are specified.
- The keys should retain the attributes they got from the file they are imported from. This should be the default behaviour which is allowed to be changed by attributes.
- If the keys after import `statement` have the same name, like `KEY2` in above example, it depends on the implementation whether it chooses to append, overwrite, raise an error or something else. The implementation must not leave this as an undefined behaviour.

# Wildcards in import statements

```
import "<path_to_file>"::{KEY??, ABC*, DEF+}
import "<path_to_file2>"::{*};
```

The following wildcards are available for use:

- Asterisk (`*`): This matches zero or more occurences of any characters. For example, `ABC*` matches `ABC`, `ABC1`, `ABC_1234`, etc.
- Plus (`+`): This matches one or more occurences of any characters. For example, `DEF+` matches `DEF1`, `DEF_1234`, etc.
- Question mark (`?`): This matches exactly one occurence of a character. For example, `KEY??` matches `KEYQQ`, `KEY12`, etc.

The second line, `"<path_to_file2>"::{*};` asks to import all keys in the file.


# Metadata/Attributes

Metadata can be associated to keys or `import` statements in the form of attributes. This can be used according to the context.

```ckv
#[attr(nest_attr(val))]
KEY =
    Value
```

- Everything in `#[attr(nest_attr(val))]` (`attr`, `nest_attr`, `val`) is an attribute.
- Here the key has the attribute `attr`. Its value is an attribute `nest_attr` whose value is attribute `val`.
- Attributes can be nested inside parantheses.
- Attrubutes may or may not be followed by parantheses.
- A value inside parantheses `()` can contain any string which itself is called an attribute.
- If the string needs to contain  `(`, `)`, `[`, `]`, `\` or `,`, they need to be backslashed.
- The string passed is parsed once to process backslashes in it. Each backslash is removed and if the character following the backslash is any of the special characters mentioned in the previous point, then it is evaulated as it is without any special meaning.

```ckv
#[attr(nest_attr(val), nest_attr2(val2)), attr2()]
KEY =
    Value
```

- Multiple attributes can be separated by commas.

```ckv
#[val1, val2]
KEY =
    Value
```

- It's valid to have attributes without parantheses.

```ckv
#[attr(nest = "val")]
KEY =
    Value
```

- In the above snippet, `nest` is an attribute which as a value `val`. The syntax is `attr = "val"`.
- The shortcoming of this syntax is that the value of such an attribute cannot contain more nested attributes.
- The value is enclosed in double quotes (`" "`). Any string is allowed inside the quotes.
- To use `"` character inside double quotes, they need to be backslashed.
- The string is initially parsed to remove special meaning of a character following backslash. In this case, if you  need string to contain `"` or `\`, they need to be backslashed.

# Global metadata/attribute

```ckv
#[!attr()]

KEY1 =
    Value1

KEY2 =
    Value2
```

- A global attribute starts with an exclamation mark (`!`). It is a syntactic sugar for writing the same attribute over all the keys in the file.
- By default, global attributes only effect the file in which they are declared.
