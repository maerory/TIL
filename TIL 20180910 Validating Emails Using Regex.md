## TIL 20180910

#### Using regex for checking for valid emails

- Usually, emails are composed of three sections divided by @ and . respectively

- The first part, username, only contains letters, digits, dashes and underscores

- The second part, website name, only contains letters and digits

- The third part, the extension, can only be up to 3 characters

- A solution using regex can be:

  ```python
  import re
  def fun(s):
      return bool(re.match(r'^([A-Za-z0-9-_])+@([A-Za-z0-9])+\.\w{1,3}$', s))
  ```

  - putting the bool function automatically converts the output as boolean type
  - '^' indicates start of string, and '$' indicates the end
  - using the parenthesis we can specify the section of the parts, the plus site indictates a connection between the strings
  - \ + . is escaping to set a special condition, \ + w is setting length limit