---
title: 'Daily Update'
layout: post
---

Life is simple in the command line. 

curl is an example of such simplicity.

```
 if curl https://google.com -I -o headers -s | cat headers | head -n 1 | cut '-d ' '-f2' > result; cat result | grep '200'; then echo 1; else echo 0; fi;
```
