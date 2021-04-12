# CKV

Clium key value.

# AIM

Aim of CKV file is to provide a simple, human readable, key value pair storage file such that promotes reusability over multiple CKV files and let's the consumer associate meta data with the keys.

# EXAMPLE

A simple example:

```text
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

```text
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
