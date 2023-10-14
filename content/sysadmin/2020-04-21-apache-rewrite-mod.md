---
date: "2020-04-21T14:05:00Z"
title: Apache Rewrite Mod
---

`mod_rewrite` is a powerful module that Apache can utilize. <!--more-->It is a way of rewriting URLs, modifying the request that Apache recieves. This could be a moved document, or enforcing SSL (rewriting the URL from http to https).

{{< rawhtml >}}
<a data-flickr-embed="true" href="https://www.flickr.com/photos/kabads/49800557653/in/datetaken/" title="Apache_HTTP_server_logo_(2016)"><img src="https://live.staticflickr.com/65535/49800557653_9ca139375d_n.jpg" width="320" height="122" align="left" style="padding:20px" alt="Apache_HTTP_server_logo_(2016)"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script> 
{{< /rawhtml >}}


This is a complex subject, which cannot be covered here, but for further information, refer to the [full documentation](https://httpd.apache.org/docs/current/mod/mod_rewrite.html).
Rewrite rules can exist in a .htaccess file, or the main configuration file, or preferably a `<Directory>` stanza.
`mod_rewrite` uses Regex (compatible with Perl) for its pattern matching engine. This is nearly unlimited in searching across a range of URLs.

### Enable Rewrite Engine

To enable `mod_rewrite` you should include the following in you Apache Documentation:

`RewriteEngine on`

A restart of Apache will be required to load the engine.

### Declaring a Rewrite Rule

To declare a rule you will have something similar to the code below:

`RewriteRule ^/old.html$ new.html [R]`

This will redirect a request that Apache receives for old.html page to new.html page. The `^` indicates that it must be the initial part of the request, and the `$` indicates that it is the end of the request. If `/subdir/old.html` is passed, it will fail this search pattern as `/subdir/` is at the beginning of the pattern. This is in line with Regular Expressions pattern matching.

These rules can be embedded within a particular directory, using the `<Directory` stanza:

    <Directory /var/www/html/subdirectory>
      RewriteEngine on
      RewriteRule "^old.html$" "new.html"
    </Directory>

The above rule only applies to the directory named `subddirectory`.

#### Rewrite Flags

At the end of each `RewriteRule` is a set of flags that determines what should be done - these are enclosed in a set of square brackets. One of the most common is `[R]` which is a redirect, carried out at the browser level (issued by the webserver).

A full list of flags is documented at [https://httpd.apache.org/docs/2.4/rewrite/flags.html](https://httpd.apache.org/docs/2.4/rewrite/flags.html\\P).

#### Regular Expressions and mod_rewrite

| Character | Meaning                                                                                       | Example                                                                |
| --------- | --------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| .         | Match any character                                                                           | c.t matches cat                                                        |
| +         | Repeats the previous match one or more times                                                  | a+ a, aa, aaa                                                          |
| \*        | Repeats the previous match zero or more times                                                 | a\* matches the same as a+ but will also match an empty string         |
| ?         | Makes the match optional                                                                      | colou?r will match color and colour                                    |
| \\        | Escape the next character                                                                     | . will match a . (dot) and not any single character as explained above |
| ^         | Called an anchor, matches the beginning of thes string                                        | ^a will match a string that begins with a                              |
| $         | The other anchor that matches the end of a string                                             | a$ will match a string that ends with a                                |
| ( )       | Groups several characters into a single unit, and captures a match for use in a backreference | (ab)+ matches abababab - the + applies to the group                    |
| [ ]       | A character class - matches one of the characters                                             | c[uoa]t matches cut, cot, cat                                          |
| [^ ]      | Negative character class - matches any character not specified                                | c[^/] matches cat or c=t but not c/t                                   |

### mod_rewrite Log Level for < v2.4

The `mod_rewrite` will write to the usual apache log files.

For previous versions (pre Apache 2.4) the directive `RewriteLogLevel` will set the level of logging written, ranging in values from 0-9 with 0 being no logging and 0 being the most verbose. Logs will appear as `pass through` lines in the file.


### mod_rewrite Log Level for v2.4

With the current version of Apache (2.4), `mod_rewrite`, the older methods of controlling logging have now been replaced by a new per-module logging method:

```
LogLevel alert rewrite:trace3
```
