# What is `screen` and why do we use it

The `screen` command is essential for managing multiple terminal sessions. It allows you to disconnect from your terminal and have your session/command keep running. In our case, this will be particularly useful for running the head `snpArcher` job that does the organizing and job tracking via snakemake. 

Here's a silly practical example. After logging in on the server, start a screen session with this:

```
screen -S example   # 'example' can be replaced with whatever name for the session you want
```
Now let's run something that will take a while!

```
sleep 300
```
This command will run for 5 minutes. In the meantime, we can exit our screen session and do some other mundane tasks, like check on running jobs with `sacct`, inspect some output, etc. etc. To exit the session, type <mark>Ctrl+A+D</mark>. When you're done and want to check back in on your session, simply type `screen -r example`. We can see that the `sleep` command is still running, but let's just cancel that and exit the screen session again.

To kill the session, type `screen -X -S example kill`.

Ok, that's pretty much all you need to know for now! You can see how having multiple screens can be helpful for running long tasks that for whatever reason aren't submitted to a cluster (for example, on your own computer). 


# Sed: a brief primer
`sed`, short for stream editor, is a helpful command-line tool used for parsing and editing text on-the-fly. It operates on text streams and is *mainly* used, at least in my workflow, for text substitution, although it's uses are [many](https://www.grymoire.com/Unix/Sed.html). 

The general syntax for sed is:
```
sed [OPTIONS] 'COMMAND' file
```
Let's jump in to some examples with our `samples.csv` file we'll be using for snpArcher. 

## Basic substitution
To replace a string, we use the s command. Let's say that we've made en error with our file paths in `samples.csv`, and we need to replace `data` with `fastq`:

```
sed "s/data/fastq/" samples.csv    # This will replace the first occurence of 'data' with 'fastq' for each line
sed "s/data/fastq/g" samples.csv   # Adding 'g' replaces every occurence of data on a line with fastq
```

Simple enough, right?  We can also use regular expressions with sed, and we'll encounter some examples later in the workshop.


## Deleting Lines
To delete specific lines, use the d command. You can specify line numbers or patterns. For example, let's delete all samples that are too far north from our sample file (latitude > 37), without targeting other occurences of '37' in the file

```
sed -r '/,37.[0-9]+,/d' samples.csv

# "-r" tells sed to use regular expressions
# ",37." matches that exact string
# "[0-9]+," matches any length (+) of numbers [0-9] until a comma (,) is reached
```
Sure we could do this in excel or R, but this handy little one-liner saves some time and means not having to read/write files. It's also great if the files are large. 

## Printing Lines
If we want to inspect just lines 5-10 of the file, it's as simple as:
```
sed -n '5,10p' samples.csv
```

## Editing-in place
To edit a file with sed without writing to `stdout`, simply add the `-i` flag.

```
cp samples.csv dummy.csv    # Make a copy to avoid editing our original file in this instance
sed -i -r '/,37.[0-9]+,/d' dummy.csv    # Take our complex command from earlier
cat dummy.csv
```
Looks like it worked! Ok, I think that's enough of a primer for now.

# Awk: a nice complement to sed

