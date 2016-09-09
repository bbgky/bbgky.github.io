---
title: "Webpage tags part 1"
date:   2016-08-15 10:39:34 -0700
tags: Machine-Learning
---
I was working on a project about classifying webpages. I gained quite some knowledge on cleaning data,  ETL (extract,  transform, load), natural language processing, unsupervised machine-learning and supervised machine-learning.

I was given  more than 40 thousand html files which are 3GB in total from an electronic retailer website. All the codes was written in **mac environment**.   

#### Use `bash` to clean a html page.

The bash code is:
{% highlight python linenos %}
cat $FILE | lynx  --stdin --dump | sed -n '/^References$/q;p'| tr '[:space:]' ' ' | tr -c '[:alpha:]' ' ' | tr -s ' ' | tr '[:upper:]' '[:lower:]' > ./tmp/${FILE:0:6}_.txt
{% endhighlight %}
* `cat $FILE` is used to display the html file in the local directory. `lynx` is text browser, and `--stdin --dump` is the method to get text out of the webpage.

* `sed -n '/^References$/q;p'` is used to find the matching string `^References$` and do quit `q` and print `p`. By using this command, all the References in the webpage will be removed. `^References$` is a regular expression which matches a line start and end is `References`.

* `tr` is a command to *translate characters*. It is a powerful tool to do further cleaning on the text. `tr '[:space:]' ' '` is to replace all the *white space* with a normal space; `tr -c '[:alpha:]' ' '` is to replace all the none character string with space; `tr -s ' '` is to squeeze multiple spaces; `tr '[:upper:]' '[:lower:]'` is to convert all the characters to lower class.

Finally, the cleaned text is saved to a *txt* file for further analysis.


#### Use `bash` to Extract header info.

header info is a good source to get descriptions and labels for the webpage.
The bash code is:
{% highlight Python linenos %}
cat $FILE |grep meta|grep -iw 'og:type\|salestype\|CategoryPath\|Country\|language'|grep -o '".*"'|sed -n '/http/q;p'|sed 's/content=//'|sed 's/ .*all-products\// /'|sed 's/\/.*"//'|sed 's/"//g'|sed 's/-n-workstations//'|sed -n '/description/q;p'>>./tmp/${i:0:6}_header.txt
{% endhighlight %}

* `grep` is a command to find lines with pattern. `grep meta` will return lines contains string `meta`. Those lines are normally in the header of the webpage. `grep -iw` is to match full strings and case insensitive. `\|` is used to match multiple strings.  `grep -o '".*"'` is to keep the strings in the `""`.

* `sed 's/content=//'` is to remove specific string `content=`. `sed 's/ .*all-products\// /'` is to replace `all-products/` with space. `\/` is used for matching string `/`. `sed 's/"//g'` is to remove all the `"`, without `g`, then only the first `"` will be removed.

* Finally, the cleaned and useful header info looks like in below:

{% highlight Python %}
og:type product
CategoryPath desktops
SalesType franchise
Country us
Language en
{% endhighlight %}
Those lines are saved into *txt* file for further use.

#### Use `bash` to process and select multiple files.

* The code above are for dealing with one file, to extract and transform multiple files, we can simply use `for` loop in `bash`. An example is shown as below:
{% highlight Python %}

for FILE in *.html
do
	# strips html
	cat $FILE | lynx --stdin --dump | sed -n '/^References$/q;p'| tr '[:space:]' ' ' | tr -c '[:alpha:]' ' ' | tr -s ' ' | tr '[:upper:]' '[:lower:]' > ./tmp/${FILE:0:6}_.txt
done
{% endhighlight %}

* To select files in a directory, usually `ls '*.html'` can do it. However if the number of files are too large, it will return exceptions: `Argument list too long` . To deal with larger amount of files, I use the combination command of `find` and `xargs`. The reference is from [stackoverflow][1].

[1]:http://stackoverflow.com/questions/11289551/argument-list-too-long-error-for-rm-cp-mv-commands

`find . -name "*.html" -print0| xargs -0 ls| wc -l`

The code above will return the number of files by calling `wc -l`.
We can also use `find` to select file by size:

`find . -size -30k -delete `

The above command will delete files with size smaller than 30k.

To randomly select certain amount of files, we can use:

`find . -name "*.html" -print0| xargs -0 ls| gsort -R| tail -n 5000| while read file; do cp $file test;done`

Using the `while` loop to copy `cp` the selected files to another directory.  `file` is a temp variable. `tail` is to display the last part of a file with flag `-n 5000` showing the last 5000 lines. `gsort -R` is to randomly sort the selected files. However, the default `sort` command doesn't have a random sort flag, the `sort` command in Linux has the flag `-R`. To solve this problem, i installed a package called `coreutils`, which contains all the Linux command. `sort` is installed as `gsort`. The reference is from [Superuser][2]

[2]: http://superuser.com/questions/334450/equivalent-of-gnu-sort-r-on-osx

#### Brief summary
It is my first time to use `bash`, and i found it is very powerful to do the ETL for webpages. I mark down all the new knowledges during the learning process, those command like `sed, tr, grep, find` are very important.
