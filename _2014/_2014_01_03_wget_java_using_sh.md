通常需要下载jdk时，直接用wget命令是不行的。那么，如何解决呢？
只需要在wget的时候加上一个特殊的cookie就可以搞定，下载最新版jdk-7u21的完整命令:

>wget --no-cookie --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F" http://download.oracle.com/otn-pub/java/jdk/7u21-b11/jdk-7u21-linux-i586.rpm
而如果出现Unable to establish SSL connection的话，在wget后面加上–no-check-certificate即可，命令如下：

>wget --no-cookie --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F" http://download.oracle.com/otn-pub/java/jdk/7u21-b11/jdk-7u21-linux-i586.rpm
