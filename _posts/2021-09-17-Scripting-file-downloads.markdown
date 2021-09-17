---
layout: post
title:  "Scripting File Downloads"
date:   2021-09-17 20:02:00 +0200
categories: bash cli script download sed grep cat curl wget
---
# Scripting File Downloads
Hi, this post will help you when you have a page with files that you want to download, but the actual URL to the files are hidden on subpages and does not follow any logic to prevent automatic scripts from downloading them.

For instance, say you have the URL `https://example.com/list_of_files`. And on that page, there are a number of other URLs listed, none of which are actual files you want to download. But if you follow the URLs you'll end up on `https://example.com/file/the_file_name` and that page does contain the URL to the file you want to download, `https://example.com/file_hash123asd.extension`.

To illustrate, this is how the page structure could be visualized:
```
https://example.com
       |--------/list_of_files <- has a list of all /file/{name}
       |
       |--------/file
       |          |--/file_a <- has a link to https://example.com/file_hash123asd.extension
       |          |--/file_b <- has a link to https://example.com/file_hash321dsa.extension
       |         ...
       |
       |--------/file_hash123asd.extension
       '--------/file_hash321dsa.extension
```

I sometimes see similar situations on pages that usually let you download the files for free if you manually click all the links on the subpages.
But to have the convinience to download all files with the press of a single button you have to pay a membership fee. Which might be inconvinient because of a number of reasons.
For instance, anonymity.

There are many plugins to many different browsers to download multiple resources from a web page, but that usually require the page to be somewhat predictable in its structure. And more times than what I would like I've run in to pages that doesn't work well with those plugins.

A solution I propose here is to write our own script to "manually" request all the link from the subpages.
It's quite simple, and I'll show you how below.

Performance is not all that important in this use case either since you are only going to run it once for the page, then you have all your files. But even if performance were important, it wouldn't be a problem, because we are only going to process some text.

# Let's get started!
First I would start off by just `curl` the URL, such as:
```
$ curl https://example.com/list_of_files
```
That should give us all the links to the subpages and a lot of noise around the links.
To clean up html page that we get back we will use `grep` and `sed`. Primarily because these are the tools I know. I'm sure there are other tools that do fit the task as well.

To avoid having to download the page every time we test a change in our script, we can save the result to a file by simply doing the following:
```
$ curl https://example.com/list_of_files > /tmp/list_of_files
```
Then we can use `cat` to quickly work with the page:
```
$ cat /tmp/list_of_files
```

Now to quickly filter most of the noise you got in your file, you can use `grep` like this:
```
$ cat /tmp/list_of_files | grep 'href="'
```
And that will filter out all the lines that does not contain `href="`. In html this would mean we only keep the lines containing links.

It is probable that there are other links that we are not interested in on the page we downloaded as well.
To filter those out we will first need to identify some characteristic of the links we want to follow.
In our case we can see that all the URLs contain `/file/` in their path. So we could try to grep on that instead on `'href="'` or we can just add `/file/` to our chain of commands to narrow it down further. It doesn't really matter. The first step was mostly to help us identify what we really wanted to grep.

So the next step would be:
```
$ cat /tmp/list_of_files | grep 'href="' | grep '/file/'
```
This should let you end up with lines that look similar to either of the following:
```
  <a href="https://example.com/file/file_name_a">
```
```
  <td class="clickable-row"><a href="/file/file_name_b">The Best File</a></td>
```
Notice that the first one of the lines contain an absolute URL while the second one contain a relative URL.
The absolute URL is ready to be used, but the relative need some more work before we can use it.
Usually relative and absolute URLs are not mixed on the same page for the same type of resource, so we will just continue this explaination with the relative URL since we will have to do some extra step to turn that one into an absolute URL. Then both cases will be the same anyway.
If you need to filter the lines even further than what we have done up until now, you can just add more grep commands.

The next step will be to parse out the relative link from the tags surrounding it and turn it into a absolute URL so we can use `curl` to get that page as well.
To parse it, we will use `sed` as such:
```
$ cat /tmp/list_of_files | grep 'href="' | grep '/file/' | sed -e 's/^.*href="//'
```
To explain our sed command: `-e` adds and expression, you can add as many expressions that you want by simply adding another `-e` flag.
The string following the flag is the expression and the `s` means we want to substitute something, the `/`s are separating the different parts of the expression, the `^.*href="` is the pattern we want to substitute (`^` matches beginning of line, `.` matches any character, `*` modifies the previous symbol to match 0 or more and `href="` matches literally what it says. So we match everything from the beginning of the line until we find `href="`).
After the next `/` we enter what we want to substitute it with, nothing.
And after the last `/` we can enter any modifiers, for instance `g` to modify every line globally or `i` to ignore case. But in our case we don't need that here.

The next step is to remove any trailing garbage on the links. To do that we just add another expression to our sed command:
```
$ cat /tmp/list_of_files | grep 'href="' | grep '/file/' | sed -e 's/^.*href="//' -e 's/">.*$//'
```
What is new here is the `$` in the expression. It simply means end of line.

Great, now our links should look similar to `/file/file_name_b`.
If they don't, don't fret, just add more sed expressions until you have a nice relative URL.

Next step is to turn the relative URL into a absolute URL.
We do that by simply concatenating the missing part of the absolute URL.
And how do we do that? With another expression! Horray for `sed`!
```
$ cat /tmp/list_of_files | grep 'href="' | grep '/file/' | sed -e 's/^.*href="//' -e 's/">.*$//' -e 's/^/https:\/\/example.com/'
```
What is going on above? We are now substituting the start of the line, represented by `^`, with the start of the URL. Neat.
The `\` are just used to escape the `/` that follows them in the expression due to `/` being the separator.

The links should now look like `https://example.com/file/file_name_b`.
If you have some lines that should be entierly removed you can use `grep -v` to filter out matching lines and `sort` and `uniq` to remove any dublicate lines if some links appear more than once.

Now we are almost done. The only part that is left is to follow the links, find the file we want to download and get it.
This will be done in a loop for each link that we now can follow.
To simplify development, let's grab just one page and work with that one. When we have that one in order, we can just join the two scrips and run it for all the files.

This second part will begin similar to the first part.
Start by getting the one of tha new pages.
```
$ curl https://example.com/file/file_name_b > /tmp/file_page
```

Then we can work locally using `cat` again.
```
$ cat /tmp/file_page | grep 'src="' | sed -e 's/^.*src="//' -e 's/".>.*$//'
```
Should hopefully give you a link to your file similar to `https://example.com/file_hash123asd.extension`.

So now that we have the second piece of our puzzle, we can join the two scripts we wrote together to fetch all the files.
To illustrate how a simple for loop works just for you to understand what will happen when we joing the pieces together:
```
$ for F in *.txt; do echo $F; done
```
`F` will be the substituted with everything that your shell expands `*.txt` to.
Every loop iteration `F` will take on the next value of `*.txt`.
`do` marks the beginning of the loop and in the loop we echo what is stored in `F`.
`done` marks the end of the loop.
So this loop will do more or less the same thing as `ls *.txt`.
We can we can also do `FOO='something'` to assing a variable, in this case `FOO` to `'something`.
We will use assignment below as well, when we join our two scripts together.
And the final piece we need to understand is `wget` that can download a page or resource from a URL that we pass it.

Now have a look below to see the final script when we join everything together in a for loop that passes in all the links to the sub pages. The `` ` `` simply means that the expression inside the backticks should be evaluated as a whole.
```
$ for PAGE in `cat /tmp/list_of_files | grep 'href="' | grep '/file/' | sed -e 's/^.*href="//' -e 's/">.*$//' -e 's/^/https:\/\/example.com/'`; do \
    FILE=`curl $PAGE | grep 'src="' | sed -e 's/^.*src="//' -e 's/".>.*$//'`; \
    wget $FILE;
done
```

And simple as that, our two lines of code are put together and you can now download all the resources you wanted without having to click on them in the browser.
For more info about the commands used in this guide, have a look at their man pages or just search for it using your favourite search engine.
