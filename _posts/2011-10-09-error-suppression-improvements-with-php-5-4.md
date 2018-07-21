---
id: 118
title: error suppression improvements with php 5.4
date: 2011-10-09T19:30:36+00:00
author: Tyrael
layout: post
guid: http://www.tyrael.hu/?p=118
permalink: /2011/10/09/error-suppression-improvements-with-php-5-4/
categories:
  - Uncategorized
---
As I mentioned in my recent [talk](http://www.slideshare.net/Tyrael/php54), the upcoming php 5.4 version brings a bunch of really nice and long awaited features and a lot bugfixes and improvements.

One of the those is: &#8220;[Improved performance of @ (silence) operator](http://svn.php.net/viewvc/php/php-src/branches/PHP_5_4/NEWS?view=markup#l260)&#8220;, which I&#8217;ve just blogged [recently](http://www.tyrael.hu/2011/06/26/performance-of-error-handling-in-php/).

Let&#8217;s see how things changed with 5.4:

<pre>Executing baseline.php

real    0m0.664s
user    0m0.660s
sys     0m0.004s
Executing baseline+error.php

real    0m4.895s
user    0m4.884s
sys     0m0.000s
Executing baseline+error_handler.php

real    0m0.644s
user    0m0.644s
sys     0m0.000s
Executing baseline+suppression.php

real    0m1.025s
user    0m1.012s
sys     0m0.012s
Executing baseline+error+error_handler.php

real    0m13.003s
user    0m12.973s
sys     0m0.004s
Executing baseline+error_handler+suppression.php

real    0m1.036s
user    0m1.036s
sys     0m0.000s
Executing baseline+error+suppression.php

real    0m5.634s
user    0m5.624s
sys     0m0.000s
Executing baseline+error+error_handler+suppression.php

real    0m13.357s
user    0m13.329s
sys     0m0.000s</pre>

For comparison, here is the same test with my debian(dotdeb) 5.3 results for the same test on the same virtual machine:

<pre>Executing baseline.php

real    0m3.252s
user    0m3.212s
sys     0m0.036s
Executing baseline+error.php

real    0m9.094s
user    0m9.069s
sys     0m0.008s
Executing baseline+error_handler.php

real    0m3.137s
user    0m3.112s
sys     0m0.016s
Executing baseline+suppression.php

real    0m4.116s
user    0m4.104s
sys     0m0.004s
Executing baseline+error+error_handler.php

real    0m27.172s
user    0m27.102s
sys     0m0.012s
Executing baseline+error_handler+suppression.php

real    0m4.057s
user    0m4.048s
sys     0m0.004s
Executing baseline+error+suppression.php

real    0m10.298s
user    0m10.261s
sys     0m0.016s
Executing baseline+error+error_handler+suppression.php

real    0m28.695s
user    0m28.626s
sys     0m0.012s</pre>

As you can see, there is a general performance improvement(5x speedup in the baseline, 2x for baseline+error), but the differences between each case seems to be similar in percentage than it was before, so using the suppression operator still negligible, but having an error generated or your custom error handler called still noticeably slower.