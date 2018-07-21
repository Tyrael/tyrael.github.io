---
id: 94
title: performance of error handling in php
date: 2011-06-26T22:41:15+00:00
author: Tyrael
layout: post
guid: http://tyrael.hu/?p=94
permalink: /2011/06/26/performance-of-error-handling-in-php/
categories:
  - Uncategorized
tags:
  - performance
  - php
---
We had a little bit of a discussion on twitter about the performance implications of errors in php.

I won&#8217;t surprise you as it is (should be) well-known that the error generation and the error suppression is slow. See this blogpost from Derick Rethans: <http://derickrethans.nl/five-reasons-why-the-shutop-operator-should-be-avoided.html>

So basically @$foo = bar; is slow because:

  * an error will be generated and formatted regardless the error\_reporting level or the existence of a custom error handler, because you can access the last error through error\_get\_last and $php\_errormsg
  * the @ operator only wraps the statement between two ini call: setting the error\_reporting to 0, and restoring it to the original value. this is 2 more call to be executed and as you would guess, this means if you have a custom error handler, where you didn&#8217;t set the second $error\_types parameter in your set\_error\_handler call, your custom error handler will be executed on the suppressed errors.
  * as Derick pointed out, the Zend Engine will generate slower code to the suppressed statements.

I&#8217;ve just wrote a little benchmark on the issue, available on [github](https://github.com/Tyrael/php-microbenchmarks)

Basically it executes an assigment in a for loop, either using the string &#8216;foo&#8217; or the non-existent constant foo, which will fallback to &#8216;foo&#8217; and trigger a notice (I know, it is not a perfect test, because the constant lookup also has some overhead, but still better and more close to the real life issues than trigger_error()).

I mixed this with using the suppress operator and a custom error handler, here are the results:

<pre>tyrael@thor:~/checkouts/php-microbenchmarks$ ./src/testError/test.sh
Executing baseline.php

real    0m1.280s
user    0m1.257s
sys     0m0.023s

Executing baseline+error.php

real    0m7.299s
user    0m7.276s
sys     0m0.022s

Executing baseline+error_handler.php

real    0m1.284s
user    0m1.255s
sys     0m0.029s

Executing baseline+suppression.php

real    0m2.035s
user    0m2.011s
sys     0m0.021s

Executing baseline+error+error_handler.php

real    0m20.405s
user    0m20.377s
sys     0m0.021s

Executing baseline+error_handler+suppression.php

real    0m2.000s
user    0m1.973s
sys     0m0.027s

Executing baseline+error+suppression.php

real    0m8.293s
user    0m8.271s
sys     0m0.021s

Executing baseline+error+error_handler+suppression.php

real    0m20.869s
user    0m20.842s
sys     0m0.020s</pre>

  * As you can see the overhead of having a custom error handler is almost negligible if it isn&#8217;t called.
  * The suppression operator adds a small overhead without errors. (~1.6X)
  * Having an error without @ or  custom error handler still slow (~5.7X).
  * if you have an error and a custom error handler which gets executed, that yields for a ~10X performance loss, regardless of using the suppression operator or not.

So my advices:

  * if you see errors in your log, fix them, thats also a performance gain.
  * if you use custom error handler, don&#8217;t forget to set the $error\_types, you can use bitmask like for error\_reporting(in our case E\_ALL &~ E\_NOTICE), or you can pass the error\_reporting() to set your handler to the same level as your error\_reporting.
  * don&#8217;t overuse the @ operator, and if you do, always handle the return values of your suppressed statements, or you will suck big time when you have to debug your application (check out the [scream](http://pecl.php.net/package/scream) pecl extension or [xdebug.scream](http://xdebug.org/docs/all_settings#scream))