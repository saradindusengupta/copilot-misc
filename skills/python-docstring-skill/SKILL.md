---
name: python-docstring
description: Write professional, detailed docstrings for Python functions, classes, and modules. Use this skill whenever the user wants to add, generate, or improve docstrings in Python code — including for individual functions, entire classes, or full modules. Trigger on requests like "write docstrings for this code", "document this function", "add docstrings", "generate documentation", or when the user pastes Python code and asks for documentation. Always use this skill for any Python documentation task, even if the user just says "document this" or "add docs".
---

# Python Docstring Skill

Generate professional, consistent Python docstrings for functions, classes, and modules.

---

## Output Format by Docstring Type

### Function Docstring

```python
def function_name(arg1, arg2="default"):
    """
    One-sentence summary of what the function does.

    Longer description if needed — explain non-obvious behavior,
    side effects, or important context.

    Args:
        arg1 (str): Description of arg1.
        arg2 (int, optional): Description of arg2. Defaults to "default".

    Returns:
        tuple: Description of what is returned and its structure.
            - element1 (str): What this element represents.
            - element2 (int): What this element represents.

    Raises:
        FileNotFoundError: If the configuration file does not exist.
        ValueError: If arg1 is empty or invalid.
        Exception: For any other unexpected errors.

    Example:
        >>> result = function_name("hello", arg2=42)
        >>> print(result)
        ("hello", 42)
    """
```

**Rules for function docstrings:**
- First line: single-sentence summary, ends with a period.
- `Args`: list every parameter. Include type in parentheses. Note defaults for optional params with `Defaults to <value>.`
- `Returns`: state the type first, then describe. If returning a tuple/dict/complex type, break down sub-elements.
- `Raises`: include ALL exceptions the function may raise, including re-raised ones.
- `Example`: show a minimal, runnable example using `>>>` doctest format.
- Omit sections that don't apply (e.g., no `Raises` if the function never raises).

---

### Class Docstring

```python
class ClassName:
    """
    One-sentence summary of what the class represents or does.

    Longer description explaining the class's purpose, design intent,
    and any important behavioral notes.

    Attributes:
        attr1 (str): Description of the attribute.
        attr2 (int): Description of the attribute. Defaults to 0.

    Methods:
        method_one: One-line description of what it does.
        method_two: One-line description of what it does.

    Raises:
        ValueError: If initialization arguments are invalid.
        ConnectionError: If the class cannot connect to required resources.

    Example:
        >>> obj = ClassName("value", attr2=10)
        >>> obj.method_one()
        'result'
    """
```

**Rules for class docstrings:**
- Describe the class as a whole, not the `__init__` method.
- `Attributes`: list all instance attributes set in `__init__` (and class-level ones if important).
- `Methods`: one-liner per public method. Skip private/dunder methods unless unusual.
- `Raises`: include exceptions from `__init__` or that propagate across methods at the class level.
- `Example`: show how to instantiate and use the class.
- Add separate docstrings on each method (use function docstring format above).

---

### Module Docstring

```python
"""
One-sentence summary of the module's purpose.

Longer description of what the module does, what problem it solves,
and any important architectural notes. Use numbered list for complex pipelines.

Key Classes:
------------
- ClassName: One-line description.
- AnotherClass: One-line description.

Key Functions:
--------------
- function_one: One-line description.
- function_two: One-line description.

Usage:
    Describe typical usage pattern in plain English, then show example.

    >>> from module_name import function_one
    >>> result = function_one("input")
    >>> print(result)
    'output'
"""
```

**Rules for module docstrings:**
- Place at the very top of the file, before any imports.
- Describe what the module does, not how.
- List all public classes and functions with a one-liner each. Skip private ones (prefixed with `_`).
- `Usage`: show the most common or representative use case.

---

## General Rules

1. **Accuracy first**: only document what the code actually does. Never invent behavior.
2. **Types must be precise**: use Python type names (`str`, `int`, `list`, `dict`, `tuple`, `bool`, `None`, `Path`, `Optional[str]`, etc.). For complex types, be specific (e.g., `list[str]`, `dict[str, int]`).
3. **Defaults must be exact**: copy default values from the function signature verbatim.
4. **No redundancy**: don't repeat the arg name in its description (bad: `name (str): The name parameter`).
5. **Tense**: use present tense ("`Returns` the bucket name", not "returned").
6. **Examples must work**: the `>>>` block should be copy-paste runnable (or clearly illustrative if I/O is involved).
7. **Indentation**: match the indentation of the surrounding code. Use 4-space indentation inside docstrings.

---

## Workflow

When the user provides code to document:

1. **Parse** the code to identify all functions, classes, and whether a module docstring is needed.
2. **Determine** which docstring types are required (function / class / module).
3. **Write** docstrings following the formats above. If the user provides the full file, output the complete file with docstrings inserted.
4. **Verify** that all arguments, return types, and exceptions match what the code actually does.

If code is ambiguous (e.g., a function's return type is unclear), make a reasonable inference and note it with a comment like `# inferred from usage`.

---

## Examples

### Example: Function

Input code:
```python
def read_config(config_file, log_metadata=None):
    if not os.path.exists(config_file):
        raise FileNotFoundError(f"Config file not found: {config_file}")
    with open(config_file) as f:
        config = yaml.safe_load(f)
    return config["bucket"], config["prefix"], config["region"]
```

Output docstring:
```python
def read_config(config_file, log_metadata=None):
    """
    Reads the AWS S3 bucket name, prefix, and region from a YAML file.

    Args:
        config_file (str): Path to the YAML configuration file.
        log_metadata (dict, optional): Additional metadata for logs. Defaults to None.

    Returns:
        tuple: Tuple containing:
            - bucket (str): The S3 bucket name.
            - prefix (str): The S3 key prefix.
            - region (str): The AWS region.

    Raises:
        FileNotFoundError: If the configuration file does not exist.
        KeyError: If required keys are missing from the config file.
        Exception: For any other unexpected errors.

    Example:
        >>> bucket, prefix, region = read_config("config.yaml")
        >>> print(bucket)
        'my-bucket'
    """
```

### Example: Class

Input code:
```python
class S3Client:
    def __init__(self, bucket, region="us-east-1"):
        self.bucket = bucket
        self.region = region
        self._client = boto3.client("s3", region_name=region)

    def upload(self, key, data):
        self._client.put_object(Bucket=self.bucket, Key=key, Body=data)

    def download(self, key):
        response = self._client.get_object(Bucket=self.bucket, Key=key)
        return response["Body"].read()
```

Output docstring:
```python
class S3Client:
    """
    Provides a simplified interface for uploading and downloading objects from AWS S3.

    Wraps the boto3 S3 client to provide a minimal interface scoped to a single
    bucket, reducing repetitive argument passing across operations.

    Attributes:
        bucket (str): The name of the S3 bucket to operate on.
        region (str): The AWS region of the bucket. Defaults to "us-east-1".

    Methods:
        upload: Uploads binary data to a specified S3 key.
        download: Downloads and returns the content of a specified S3 key.

    Raises:
        botocore.exceptions.BotoCoreError: If AWS credentials are missing or invalid.
        botocore.exceptions.ClientError: If the bucket does not exist or access is denied.

    Example:
        >>> client = S3Client("my-bucket", region="eu-west-1")
        >>> client.upload("data/file.txt", b"hello world")
        >>> content = client.download("data/file.txt")
        >>> print(content)
        b'hello world'
    """
```
