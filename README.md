# wpBullet [![Build Status](https://travis-ci.org/LukaSikic/wpbullet.svg?branch=dev)](https://travis-ci.org/LukaSikic/wpbullet)
A static code analysis for WordPress Plugins/Themes (and PHP)


## Installation
wpBullet requires Python 3. It was only tested on Python 3.7
- `$ git clone https://github.com/LukaSikic/wpbullet` 
- `$ python3 wpbullet-master/wpbullet.py`


## Usage
Available options:
```
--path (required) System path to folder in which plugin files are located, ex. --path="/path/to/plugin"
--enabled (optional) Check only for given modules, ex. --enabled="SQLInjection,CrossSiteScripting"
--disabled (optional) Don't check for given modules, ex. --disabled="SQLInjection,CrossSiteScripting"

$ python wpbullet.py --path="/var/www/wp-content/plugins/plugin-name"
```

## Creating modules
Creating a module is flexible and allows for override of the `BaseClass` methods for each module as well as creating their own methods

Each module in `Modules` directory is implementing properties and methods from `core.modules.BaseClass`,
thus each module's required parameter is `BaseClass`

Once created, module needs to be imported in `modules/__init__.py`. Module and class name must be consistent
in order to module to be loaded.

__If you are opening pull request to add new module, please provide unit tests for your module as well.__


### Module template

`Modules/ExampleVulnerability.py`
```python
from core.modules import BaseClass


class ExampleVulnerability(object):

    # Vulnerability name
    name = "Cross-site Scripting"

    # Vulnerability severity
    severity = "Low-Medium"

    # Functions causing vulnerability
    functions = [
        "print"
        "echo"
    ]

    # Functions/regex that prevent exploitation
    blacklist = [
        "htmlspecialchars",
        "esc_attr"
    ]

```

#### Overriding regex match pattern
Regex pattern is being generated in `core.modules.BaseClass.build_pattern` and therefore can be overwritten in 
each module class.

`Modules/ExampleVulnerability.py`
```python
import copy


...
# Build dynamic regex pattern to locate vulnerabilities in given content
def build_pattern(self, content, file):
    user_input = copy.deepcopy(self.user_input)

    variables = self.get_input_variables(self, content)

    if variables:
        user_input.extend(variables)

    if self.blacklist:
        blacklist_pattern = r"(?!(\s?)+(.*(" + '|'.join(self.blacklist) + ")))"
    else:
        blacklist_pattern = ""

    self.functions = [self.functions_prefix + x for x in self.functions]

    pattern = r"((" + '|'.join(self.functions) + ")\s{0,}\(?\s{0,1}" + blacklist_pattern + ".*(" + '|'.join(user_input) + ").*)"
    return pattern
```

## Testing
Running unit tests: `$ python3 -m unittest`