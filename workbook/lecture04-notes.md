# Lecture notes: Class Session 04

## Biological motivations

* The wealth of genomic data available in public databases such as GenBank provides novel opportunities to address a wide range of biological questions. But knowing what's there and figuring out how to access and compute efficiently on this data can be challenging
* Today we will use our computing skills to mine information about the collection of genome assemblies available in GenBank
* We will ask questions like: How many genome assemblies are available? What taxa have the greatest number of genomes available? How "complete" are genome assemblies in GenBank?

## Computational motivations

* We will introduce a tool called "Awk". Awk is both a command line tool and a programming language.  Awk excels at carrying out computations on tabular data.

* We will learn about some ways to do "iteration" while working in the Unix command line environment.  "Iteration" refers to the repetition of computation to generate a set of outcomes.  Iteration is a key part of almost every non-trivial computing task.
 

## Example data

The [NCBI Genome Page](https://www.ncbi.nlm.nih.gov/genome) is a good starting place to browse information about genome data avaialable in Genbank.  However, if we want to analyze in a systematic way what's available in Genbank we'll need access to some raw meta-data about genome information.  Luckily for us, NCBI maintains such information in various forms. We'll focus on genome assemblies themselves.


* Use `wget` to download the following file that contains info about all genome assemblies in Genbank from:

  https://ftp.ncbi.nlm.nih.gov/genomes/genbank/assembly_summary_genbank.txt 

* See the following link for an explanation of the columns in this file:

    https://ftp.ncbi.nlm.nih.gov/genomes/README_assembly_summary.txt


* Some of code examples below will be best illustrated with a smaller version of the assembly meta data. Let's generate a file called `assembly_snippet.tsv` by using the `head` command to extract the first 10 lines of the file:

    ```bash
    $ head assembly_summary_genbank.txt > assembly_snippet.tsv
    ```


## Introduction to Awk

### Installing GNU awk 

Awk (`awk`) is installed by default on pretty much every modern Unix-based system.  However, there are different implementations of Aw, some of which offer additional functionality. We're going to use an Awk implementation called [GNU Awk](https://www.gnu.org/software/gawk/manual/) which is sometimes referred to as "gawk". 

It's easy to install gawk we'll use a program called `apt` which is the standard Ubuntu "package manager". You can think of a package manager as equivalent to the App Store on your phone or laptop.

* Install GNU awk  (gawk) with the following command:

    ```bash
    $ sudo apt install gawk
    ```

* When you execute this command you will be prompted to enter your password. On your Duke VM this password is your NetID password (i.e. the same password you use to login to the VM). 
  
* `apt` will give you some information about what it's going to install, and prompt you with the message "Do you want to continue". Accept the defaults (hit return) and the installation process will begin. You'll get a bunch of messages about whats being installed. 

### A mental model for working with Awk

It will be easiest to learn Awk if you have a useful mental model for how Awk works. 

Alfred Aho, one of the inventors Awk (the "A" in AWK) described it as follows:

> AWK reads the input a line at a time. A line is scanned for each pattern in the program, and for each pattern that matches, the associated action is executed. -- Alfred Aho

Elaborating on Aho's description:

* Awk is line oriented -- Awk processes its inputs line-by-line
* Awk programs are made up of `pattern {action}` statements -- When using awk we specify one or more `pattern {action}`  rules. Awk then compares every line to each specified pattern and if the line matches the pattern, the corresponding action is executed. 

A few other Awk features that I consider critical for understanding:

* Lines have fields -- Awk considers each line to be made up by fields; each field is a set of characters separated by delimiters (spaces or tabs by default but can be changed). 

    - Fields can be accessed with the syntax `$1, $2, ..., `. The total number of fields in a line is `$NF`; `$0` is the whole line (i.e. all the fields)

* A pattern can be a string expression, a numeric expression, or a regular expression (or some combination)

* An action is one or more valid Awk expressions

* A `pattern {action}` statement can omit the pattern or omit the action but not both.  
  * If the pattern is omitted than the action is applied to every line.
  * If the action is omitted than any line that matches the pattern is printed
  
* Special `BEGIN` and `END` rules can be specified to execute actions at the start and end of a file 

See also Brian Kernighan's (the "K" in AWK) description of how Awk works here: https://www.cs.princeton.edu/courses/archive/spring19/cos333/awk.help


## Example Awk programs

### Filter lines by based on the content of a field

```awk
# extract_ref_assemblies.awk

$5 == "reference genome" { print $0 }
```
Run this program against the `assembly_snippet.tsv` file generated previously as so:

```
$ awk -F "\t" -f extract_ref_assemblies.awk assembly_snippet.tsv
```

Explanation:
* `-F "\t"` tells awk that fields are tab delimited
* There is one `pattern {action}` statement
* We focus on Column 5 of the assembly meta data file which specifies whether the assembly is considered a reference genome in RefSeq
* The `pattern` can be read as: "If the 5th field is equal to ``reference genome''"
* the `action` can be read as: "Print the entire line" (`print` is a built-in Awk function)

This awk program is simple enough that we could have simply written it directly in the terminal as so:

```
$ awk '$5 == "reference genome" {print $0}' assembly_snippet.tsv
```

We can call the `print` function without any arguments; in this case it assumes you want to print the whole line. So we could simplify the above as:

```
$ awk '$5 == "reference genome" { print }' assembly_snippet.tsv
```

As noted above, if an action is omitted than every line that matches the pattern is printed so we could simplify this even further as:

```
$ awk '$5 == "reference genome"' assembly_snippet.tsv
```

### Identify comment lines using regular expressions

* A pattern can be a regular expression
* Regular expressions are written with the syntax: `/<regex>/'
* Here's a Awk program to print all lines that begin with the character "#"

```awk
# print_comments.awk

/^#/ { print $0 } # action could be ommitted here
```

### Match fields with regular expressions

* In the prior example, the regular expression was applied to the line as a whole
* We can also match on specific fields with regular expressions
* The following has two `pattern {action}` statements

```awk

BEGIN {
    FS = "\t"
    OFS = ","
}

$8 ~ /Drosophila/ { print $1, $8, $5 }
```

Execute as:

```
$ awk -f extract_drosophila.awk assembly_snippet.tsv
```

Explanation:
* BEGIN rule applies at the start of input processing
* `FS` is a special variable that refers to the input Field Separator (same as specifing `-F` at the command line)
* `OFS` is a variable specifying the "Output Field Separator" -- the delimiter used for fields when printing from Awk. The default is spaces. Here we replace that with commas.
* Do regular expression matching on the 8th field
* Print the 1st, 8th, and 5th fields



### Print the last field in every line

This program contains only an action. In the absence of a pattern it is applied to every line.

```awk
# printlast.awk
{ print $NF }
```

Run this program as so:

```
$ awk -F "\t" -f printall.awk assembly_snippet.tsv
```

Explanation:
* `$NF` -- `NF` gives the number of fields in each line, so `$NF` extract the text in the last field.

The same program directly in the command line:

```
$ awk -F "\t" '{ print $NF}' assembly_snippet.tsv
```

## Combining Awk into pipelines

Awk programs can be easily integrated with other Unix tools into standard pipelines.

### Filtering with awk, counting with sort and uniq

Here's an awk program that returns all assemblies for which column 8 has the term "Drosophila" but **does not** include the string "melanogaster".

```awk
# non_melanogaster.awk

BEGIN {
    FS = "\t"
    OFS = "\t"
}

# go immediately to the next line when a comment line is encountered
/^#/ { next  }  

# Combining two regular expression with an "AND" operator (&&)
# !~ /<regex>/ means "doesn't match the regular expresssion
$8 ~ /^Drosophila/ && $8 !~ /melanogaster/ { print $8 }
```

Explanation of the Awk program:
* `next` is statement that causes awk to stop evaluating any further rules for the current line and go to the next line
* We combined two regular expressions using the `&&` (AND) operator
  * The OR operator is '||'
* `!~ /<regex>/` means doesn't match the regular expresssion


We can combine this awk program with `sort` and `uniq` to get a count of each non-melanogaster Drosophila species

```
$ awk -f non_melanogaster.awk assembly_summary_genbank.txt | sort | uniq -c
```



## More on awk

For additional introductory notes on Awk see

* My [Bio 724 notes on Awk](https://github.com/Bio724/Bio724-Lecture-Notes/blob/main/lecture-awk/awk.md)
* [learnbyexample: Gnu Awk](https://learnbyexample.github.io/learn_gnuawk/)
 

