# CKV File

Clium key value file.

# AIM

Aim of CKV file is to provide a simple, human readable, key value pair storage file that promotes reusability over multiple CKV files and lets the consumer associate meta data with the keys.

# EXAMPLE

A simple example:

```ckv
// Import key-val pairs from other ckv files into this file
import "path/to/other/file.ckv"

// This is a comment

/*
This is a multi-line comment
*/

// This is a variable whose value is always a string
var xyz = abc

THIS_IS_A_KEY =
  After a tab, starts the value
  Value can be spanned across multiple lines.
  Every tabbed line in continuation is part of value of THIS_IS_A_KEY.

#[some_attribute(args_to_attribute)]
ATTRIBUTE_EXAMPLE_KEY =
  ATTRIBUTE_EXAMPLE_KEY has meta data associated to it
  in the form of attributes. Attributes are defined by the parser implementing
  the CKV file.
  For example,
  clium defines an attribute "use" which can be used like #[use(var(xyz))].
```

From a sample clium file:

```ckv
import "general.ckv"

#[!use(std/macros)]

var CC = gcc

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

# KEY-VALUE

```ckv
KEY =
  VALUE
  VALUE CONTINUED
```

- Key and values are the building blocks of ckv file. A letter in the key matches the following character class: `[0-9a-zA-Z_-]`. 
Following the key, there should be an equal-to (=).
- The next line immediately after the key must be started with a tab. Following the tab is the value of the key.
- If the next line also starts with a tab, it will be a part of key's value and this will go on until a line following
- doesn't start with a tab or `----` (4 hypens).
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

# KEY'S METADATA/ATTRIBUTE

Metadata can be associated to keys in the form of attributes. This can be used by according to the context.

```ckv
#[attr(nest_attr(val))]
KEY =
    Value
```

- Here the key has the attribute `attr`. It's value is an attribute `nest_attr` whose value is `val`.
- So, an attribute's value can be another attribute or value.
- A value inside parantheses `()` can contain any string. If the string ends with `()`, then it is an attribute itself.
- If the string needs to contain  `(`, `)`, `[`, `]`, `\` or `,`, it needs to be backslashed.
- The string passed is parsed once to process backslashes in the it. Each backslashed is removed and if the character following the backslash is any of the special chars mentioned in the previous point, then it is evaulated as it as without any special meaning.

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

- It's valid to have only values on the top level.

```ckv
#[attr(nest = "val")]
KEY =
    Value
```

- In the above snippet, `nest` is an attribute which as a value `val`. The syntax is `attr = "val"`.
- The shortcoming of this syntax is that the value of such an attribute cannot be an attribute.
- The value is enclosed in double quotes (`" "`). Any string is allowed inside the quotes.
- To use `"` character inside double quotes, they need to be backslashed.
- The string is initially parsed to remove remove special meaning of a character following backslash. In this case, if you need you string to contain `"` or `\`, they need to be backslashed.

# GLOBAL METADATA/ATTRIBUTE

```ckv
#[!attr()]

KEY1 =
    Value1

KEY2 =
    Value2
```

A global attribute starts with an exclamation mark (`!`). It applies to all the keys in the file. This also applies when the file is being imported. But it doesn't effect the file it is being imported to.
