
# Iteration at the command line

Computers are awesome at helping us accomplish repetitive tasks.  Running the same computation repeatedly for different sets of inputs is fundamenetal to all types of data analysis. Let's look at two ways to accomplish this when working at the Unix command line -- 1) usng a "for-loop" in the bash shell; or 2) using a program called `parallel`.

## Example data

* Hopefully you've already downloaded the yeast reference genome from Genbank and created a symlink with the name `yeast.fna`. If not see the [[lecture03-notes]].
 
* Download two more genome sequences:
  
  The bacterium E coli:
 
   ```
  $ wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_genomic.fna.gz
  $ gunzip GCF_000005845.2_ASM584v2_genomic.fna.gz
  $ ln -s GCF_000005845.2_ASM584v2_genomic.fna.gz ecoli.fna
  ```

  And the SARS-COV-2 reference genome:

  ```
  $ wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/009/858/895/GCF_009858895.2_ASM985889v3/GCF_009858895.2_ASM985889v3_genomic.fna.gz
  $ gunzip GCF_009858895.2_ASM985889v3_genomic.fna.gz
  $ ln -s GCF_009858895.2_ASM985889v3_genomic.fna sarscov2.fna
  ```


## Iteration using for-loops in bash

* Below is an example of a for-loop in bash:

    ```bash
    # forloop01.sh
    # Run this as:  bash forloop01.sh 
    
    for item in how now brown cow
    do
        echo $item
    done
    ```

    * The keyword `for` indicates we're declaring the loop
    * The variable  `item` specifies the name we'll use to refer to the things we're computing on
    * The list of items we want to compute on appears to the right of the keyword `in`
    * The computation we're doing appears between the keywords `do` and `done`

* Here's a second for loop with a slightly more involved computation:

    ```bash
    # forloop02.sh
    # Run this as:  bash forloop02.sh 
    for genome in yeast.fna ecoli.fna sarscov2.fna
    do
        grep -v '^>' $genome | wc -c
    done
    ```

    * We're free to pick a variable name that is meaningful in the context of the computation we're carrying out -- `genome` rather than `item` in this case

* In general the syntax of a for loop in bash looks like the following:

    ```bash
    for var in [inputs]
    do
        [compute with $var]
    done
    ```


### Iteration using GNU parallel

Bash for-loops are relatively straightforward to use, but can become cumbersome when the list of things we want to compute from is long, or when the inputs of interest are stored in a file.  A tool called `parallel` can help us for cases like this.  

* Install `parallel` using `apt`:

    ```
    sudo apt install parallel
    ```


#### Iterating with `parallel`

`parallel` can be used to mimic a for loop. For example, the command below, executed at the terminal is equivalent to `forloop02.sh` above:

    ```
    parallel 'grep -v "^>" {} | wc -c' ::: yeast.fna ecoli.fna sarscov2.fna
    ```

* The list we want to compute on appears to the right of the three colons `:::`
* Instead of specifying a variable name, parallel subsitutes the items in our list to the place holder  `{}`
* One difference between the bash for-loop and `parallel` is that `parallel` doesn't necessarily process your inputs in the order they were provided. This has to do to do with `parallels` ability to run computations simultaneously on different CPU cores on your computer (i.e. "parallel computing" or multi-processing). To force `parallel` to process the input in the order given use the `-k` option

#### Using parallel with inputs taken from a file

`parallel` can also work with a list of inputs from a file instead of specified explicitly on the command line:

* Create the file `genomes.txt` with each input you want to process on it's own line. 

    ```
    yeast.fna
    ecoli.fna
    sarscov2.fna
    ```

* Run parallel as follows, noting the four colons `::::` and then the name of the file with the inputs

    ```
    parallel 'grep -v "^>" {} | wc -c' :::: genomes.txt
    ```

* Since parallel can process the inputs in any order if we want to make sure we know which output corresponds to which input we might modify the previous call as follows to create a small table

    ```
    parallel 'echo -e {}"\t"$(grep -v "^>" {} | wc -c)' :::: genomes.txt
    ```

    Here we're using a bash feature called command substitution. The syntax `$(command)` means execute `command` and then treat the output as a string. 

    In this particular case we're passing the output of the computation as inputs to `echo`, along with the name of the file we're processing (the `{}`) and the tab character `"\t"` to generate a tab-delimited table (read about the `-e` option in the man pages for `echo`)


