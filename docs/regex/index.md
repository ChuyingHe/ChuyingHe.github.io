## in Python:
```py
import re
import sys

regex_pattern = r"_________"	# Do not delete 'r'.

test_string = input()

match = re.match(regex_pattern, test_string) is not None

print(str(match).lower())
```
|regex expression|Description|
|:--|:--|
|`.`|The dot (.) matches anything (except for a newline).|
|`$`|Ending of the match|
|``||
|``||
|``||
|``||


...\....\....\....

r"(.{3}.){3}.{3}$"