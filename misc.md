# Table of contents

1. [SLURM](#slurm)
2. [What is `screen` and why do we use it](#what-is-screen-and-why-do-we-use-it)
3. [Sed: a brief primer](#sed-a-brief-primer)
    1. [Basic substitutions](#basic-substitutions)
    2. [Deleting lines](#deleting-lines)
    3. [Printing lines](#printing-lines)
    4. [Editing-in place](#editing-in-place)
4. [Awk: a nice complement of sed](#awk-a-nice-complement-to-sed)
    1. [Awk structure](#awk-structure)
    2. [How to run awk programs](#how-to-run-awk-programs)
    3. [Awk extra features](#extra-features)

   


# `SLURM`

You may already be familiar with `SLURM` basics, but for everyone's sake, I've included a brief primer with some of the daily `SLURM` commands I reference. First off, **what's `SLURM`?**

- `SLURM` is a popular cluster management and job scheduling system. It has three essential functions:
    - allocate exclusive and/or non-exclusive access to resources to users for some time so they can perform work.
    - provides a framework for starting, executing, and monitoring work on the allocated resources.
    - arbitrates contention for resources by managing a queue of pending work.
  
In general, we could say it allows you to structure and run processes in a contained way that minimizes screwing up the server. 

The basic format of a `SLURM` script is:

```
#!/bin/bash
#SBATCH --job-name=my_job          # Job name
#SBATCH -o logs/output.o           # Standard output log
#SBATCH -e logs/output.e           # Error log. You can specifu the same file for both output and error if you want
#SBATCH -N 1                       # Number of nodes - it is unlikely you will every mess with that
#SBATCH -n 1                       # Number of tasks (processes)
#SBATCH -c 1                       # Number of threads(cpus) - this is what you will change when you want to run something multithreaded
#SBATCH --time=00:10:05            # Time limit hrs:min:sec
#SBATCH --mem=1000                 # Memory limit in MB
#SBATCH -p big_partition           # The name of whatever partition you're submitting the job too. Many clusters will have specialized partitions for long or big mem jobs

## Code here
sleep 600

```
And you can submit a script formatted that way by simply running `sbatch script.sh`!

To check on all jobs that are running, you can use:

```
squeue -u tywooldr    # Replace with your username. This shows pending and running jobs.
sacct                 # This shows all your jobs, including completed/failed ones and the resources they used. This can be useful for refining workflows in the future and providing the right amount of resources
```

If you want to submit a job on the fly and not write a script, you can also do something like this:

```
sbatch --mem=1000 --time=00:10:05 --wrap="sleep 600"
```

Which will run the command inside `wrap` with default parameters for anything not specified (as usual). 

Sometimes, we want to interactively play with large amounts of data and run processes that require a lot of memory. For this, we can set up an **interactive** job. The following command gives you a pseudo-terminal (`--pty`) with 5000MB RAM and 8 CPUs for 2 hours.

```
srun --pty --mem=5000 --time=02:00:00 -c 8 bash
```

This job will end as soon as you close the terminal. If you want to be able to close your computer and return to the interactive job later, then `screen` is your tool!

# What is `screen` and why do we use it

The `screen` command is essential for managing multiple terminal sessions. It allows you to disconnect from your terminal and keep your session/command running. This will be particularly useful for running the head `snpArcher` job, which organizes and tracks jobs via `Snakemake.` 

Here's a silly practical example. After logging in on the server, start a screen session with this:

```
screen -S example   # 'example' can be replaced with whatever name for the session you want
```


Now, let's run something that will take a while!

```
sleep 300
```

This command will run for 5 minutes. In the meantime, we can exit our screen session and do some other mundane tasks, like check on running jobs with `sacct`, inspect some output, etc. To exit the session, type <kbd>Ctrl</kbd>+<kbd>A</kbd><kbd>D</kbd>. When you're done and want to check back in your session, simply type `screen -dr example`. We can see that the `sleep` command is still running, but let's just cancel that and exit the screen session again.

To kill the session, type `screen -X -S example kill`.

Okay, that's all you need to know for now! You can see how having multiple screens can be helpful for running long tasks that aren't submitted to a cluster (for example, on your own computer). 

> [!TIP]
> There's also an alternative to `screen` called [`tmux`](https://github.com/tmux/tmux/wiki).

# Sed: a brief primer

`sed`, short for stream editor, is a helpful command-line tool for parsing and editing text on the fly. It operates on text streams and is *mainly* used, at least in my workflow, for text substitution, although its uses are [many](https://www.grymoire.com/Unix/Sed.html). 

The general syntax for sed is:

```
sed [OPTIONS] 'COMMAND' file
```

Let's start with some examples from our `samples.csv` file, which we'll be using for `snpArcher`. 

## Basic substitutions

To replace a string, we use the `s` command. Let's say that we've made an error with our file paths in `samples.csv`, and we need to replace `data` with `fastq`:

```
sed "s/data/fastq/" samples.csv    # This will replace the first occurrence of 'data' with 'fastq' for each line
sed "s/data/fastq/g" samples.csv   # Adding 'g' replaces every occurrence of `data` on a line with `fastq`
```

Simple enough, right?  We can also use regular expressions with `sed`, and we'll encounter some examples later in the workshop.


## Deleting Lines

To delete specific lines, use the `d` command. You can specify line numbers or patterns. For example, let's delete all samples that are too far north from our sample file (latitude > 37) without targeting other occurrences of '37' in the file.

```
sed -r '/,37.[0-9]+,/d' samples.csv
```

- `-r` tells sed to use regular expressions
- `,37.` matches that exact string
- `[0-9]+,` matches any length (`+`) of numbers `[0-9]` until a comma (`,`) is reached

Sure, we could do this in Excel or R, but this handy little one-liner saves some time and means we don't have to read/write files. It's also great if the files are large. 

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


Looks like it worked! Ok, that's enough of a primer for now.


# Awk: a nice complement to sed

Awk is a program that you can use to select particular records in a file and perform operations upon them.

How does *AWK* work? 

1. Awk breaks each line of input passed to it into fields. By default, a field is a string of consecutive characters delimited by whitespace, but the delimiter can be modified.
2. Awk parses and operates on each separate field.

This is a deal for handling structured text files (especially tables) and organizing data into consistent chunks, such as rows and columns.

## Awk structure 

From [the GNU Awk User Guide](https://www.gnu.org/software/gawk/manual/gawk.html#Getting-Started):

> When you run awk, you specify an awk program that tells awk what to do. The program consists of a series of rules. Each rule specifies one pattern to search for and one action to perform upon finding the pattern.
> Syntactically, a rule consists of a pattern followed by an action. The action is enclosed in braces to separate it from the pattern. Newlines usually separate rules. Therefore, an awk program looks like this:

```
pattern { action }
pattern { action }
```

The program is surrounded by single quotes (strong quoting), and curly brackets enclose the actions of awk code within a shell script.


## How to run awk programs?

You would run:

```
awk 'program' input.file # When using a file

or

input stream | awk 'program'
```

## Extra features 

While each block in `curly brackets` will be executed every time a new line is read, we can execute commands right after `awk` processes the input and after it has processed all input with commands `BEGIN` and `END`. It also supports some pre-defined and automatic variables to help you write your programs. Among them, you will often encounter:

- `NR`: The current input record number. Assuming we are delimiting our "records" with newlines (`\n`), this will match the line number being processed.
- `NF`: Number of fields of the line being processed.
- `FS`/`OFS`: Input and output field separators.

For example, we have our `samples.csv` and want to count per record (line) how many fields we have and for how many we have the minimum required (5).
```
awk 'BEGIN{FS=","; counter=0}NF==5{counter=counter+1}END{print "Number of records with minimum number of fields: ", counter}' samples.csv
```


Let's try to do something more applicable to our data. We want to get information from the `samples.csv` file, as we did with `sed`. Imagine we want to know from the samples file how many are too far north (latitude > 37) and which is the farthest North.

```
 # Column 1 is BioSample
 # Column 10 is latitude
 awk 'BEGIN{
      maxlatitude=0;
     counter=0;
   maxsample=""}
   $10>37{
     counter=counter+1;
     if (maxlatitude<$10) {
       maxlatitude=$10;
        maxsample=$1
      }
    }
    END{
      print "Number of samples up north: ",counter,"\nFarthest North is sample: ",maxsample, " at ", maxlatitude, "latitude". 
    }' samples.csv
```
