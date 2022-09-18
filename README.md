# j2gpp - Jinja2-based General Purpose Preprocessor

`j2gpp` is a command-line tool for rendering templates using the Jinja2 syntax. It's intended purpose is to serve as a preprocessor for any programming or markup language with a unified syntax and flow accross languages.

## Installation

Simply download the latest executable for your platform from the [release page](https://github.com/Louis-DR/j2gpp/releases), unzip it to the desired installation directory and add to your PATH.

## Basic usage

`j2gpp` requires at least one source be provided. The source paths can be relative or absolute, and can use UNIX-style patterns such as wildcards. Template file names must end with the `.j2` extension which will be stripped at render.

For more information about the Jinja2 syntax, see the documentation at [jinja.palletsprojects.com](https://jinja.palletsprojects.com/).

For instance, suppose we have a templatized source file `foo.c.j2` :

``` c
#include <stdio.h>

{% set message = "Hello, world!" %}

int main() {
  printf({{message}});
  return 0;
}
```

We can render the template using `j2gpp` :

``` shell
j2gpp ./foo.c.j2
```

The output is written to `foo.c` next to the source template :

``` c
#include <stdio.h>

int main() {
  printf(Hello, world!);
  return 0;
}
```

The following arguments can be added to the command for additional features. The details of each command is explained in the sections below.

| Argument       | Description                                                |
| -------------- | ---------------------------------------------------------- |
| `-O/--outdir`  | Output directory for all rendered templates                |
| `-o/--output`  | Output file for single template                            |
| `-I/--incdir`  | Include search directory for include and import statements |
| `-D/--define`  | Inline global variables for all templates                  |
| `-V/--varfile` | Global variables files for all templates                   |
| `--perf`       | Measure the execusion time for performance testing         |
| `--version`    | Print J2GPP version and quits                              |
| `--license`    | Print J2GPP license and quits                              |

## Command line arguments

### Specify output directory

By default the rendered files are saved next to the source templates. You can provide an output directory with the `-O/--outdir` argument. The output directory path can be relative or absolute. If the directory doesn't exist, it will be created.

For instance the following command will write the rendered file to `./bar/foo.c`.

``` shell
j2gpp ./foo.c.j2 --outdir ./bar/
```

### Specifying output file

By default the rendered files are saved next to the source templates. If a single source template is provided, you can specify the output file directory and name with the `-o/--output` argument. The output file path can be relative or absolute. If the directory doesn't exist, it will be created.

For instance the following command will write the rendered file to `./bar.c`.

``` shell
j2gpp ./foo.c.j2 --output ./bar.c
```

### Include search directory

The `include` and `import` Jinja2 statements require specifying the directory in which the renderer will search. That is provided using the `-I/--incidr` argument.

For instance, with the following command, the files in the directory `./includes/` will be available to `include` and `import` statements when rendering the template `foo.c.j2`.

``` shell
j2gpp ./foo.c.j2 --incdir ./includes/
```

### Passing global variables in command line

You can pass global variables to all templates rendered using the `-D/--define` argument with a list of variables in the format `name=value`. Values are parsed to cast to the correct Python type as explained [later](#command-line-define). Dictionary attributes to any depth can be assigned using dots "`.`" to separate the keys. Global variables defined in the command line overwrite the global variables set by loading files.

For instance, with the following command, the variable `bar` will have the value `42` when rendering the template `foo.c.j2`.

``` shell
j2gpp ./foo.c.j2 --define bar=42
```

### Loading global variables from files

You can load global variables from files using the `-V/--varfile` argument with a list of files. The file paths can be relative or absolute, and can use UNIX-style patterns such as wildcards. Variables file types supported right now are YAML, JSON, XML, TOML, INI/CFG, ENV, CSV and TSV. Global variables loaded from files are overwritten by variables defined in the command line.

For instance, with the following command, the variable `bar` will have the value `42` when rendering the template `foo.c.j2`.

``` shell
j2gpp ./foo.c.j2 --varfile ./qux.yml
```

With the variables file `qux.yml` :

``` yml
bar: 42
```

## Supported formats for variables

Jinja2 supports variables types from python. The main types are None, Boolean, Integer, Float, String, Tuple, List and Dictionary. J2GPP provides many ways to set variables and not all types are supported by each format.

### Command line define

Defines passed by the command line are interpreted by the Python [ast.literal_eval()](https://docs.python.org/3/library/ast.html#ast.literal_eval) function which supports Python syntax and some additional types such as `set()`.

``` shell
j2gpp ./foo.c.j2 --define test_none=None             \
                          test_bool=True             \
                          test_int=42                \
                          test_float=3.141592        \
                          test_string1=lorem         \
                          test_string2="lorem ipsum" \
                          test_tuple="(1,2,3)"       \
                          test_list="[1,2,3]"        \
                          test_dict="{'key1': value1, 'key2': value2}" \
                          test_dict.key3=value3
```

### YAML

``` yml
test_none1:
test_none2: null

test_bool1: true
test_bool2: false

test_int: 42
test_float: 3.141592

test_string1: lorem ipsum
test_string2:
  single
  line
  text
test_string3: |
  multi
  line
  text

test_list1: [1,2,3]
test_list2:
  - 1
  - 2
  - 3

test_dict:
  key1: value1
  key2: value2
  key3: value3
```

### JSON

``` json
{
  "test_none": null,

  "test_bool1": true,
  "test_bool2": false,

  "test_int": 42,
  "test_float": 3.141592,

  "test_string": "lorem ipsum",

  "test_list": [1,2,3],

  "test_dict": {
    "key1": "value1",
    "key2": "value2",
    "key3": "value3"
  }
}
```

### XML

Note that XML expects a single root element. To avoid having to specify the root elemet when using the variables in a template, J2GPP automatically removes the root element level if it is named "`_`".

``` xml
<_>
  <test_none></test_none>

  <test_bool1>true</test_bool1>
  <test_bool2>false</test_bool2>

  <test_int>42</test_int>
  <test_float>3.141592</test_float>

  <test_string>lorem ipsum</test_string>

  <test_list>1</test_list>
  <test_list>2</test_list>
  <test_list>3</test_list>

  <test_dict>
    <key1>value1</key1>
    <key2>value2</key2>
    <key3>value3</key3>
  </test_dict>
</_>
```

### TOML

``` toml
test_bool1 = true
test_bool2 = false

test_int = 42
test_float = 3.141592

test_string1 = "lorem ipsum"
test_string2 = """
multi
line
text"""

test_list = [1,2,3]

[test_dict]
key1 = "value1"
key2 = "value2"
key3 = "value3"
```
