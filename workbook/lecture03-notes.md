## Post-lecture notes: Class Session 3

### Biological Motivations

- Genome annotation represents our current best understanding of the functional "parts" encoded in genomes

- By analyzing annotation information (often in conjunction with additional data sources) we can understand how genomes are structured and quantify key properties of genes and other functional elements

- Examples of questions you can explore by analyzing genome annotation include:

    - How many protein coding genes are there?
    - Is the average density of genes similar across chromosomes?
    - How many exons, on average, does each protein coding gene have?
    - What is the average length of exon and intronic sequences?
    - Do functionally related genes exhibit spatial clustering?
    - How similar is the arrangement of genes and functional elements between two species in the same genus? Between more distantly related groups?

- If we analyze genome annotation in combination with the associated nucleotide sequence data we can ask questions like:

    - Are there nucleotide composition differences between genic and non-genic regions? Or other types of features?
    - Are there sequence patterns (motifs) associated with particular features (e.g. introns) or feature groupings (e.g. genes involved in particular biological processes)


### Computational Motivations

- "Plain text" data formats are ubiquitous in genomics and other fields. Unix core utilities provided a wealth of commands for filtering, restructuring, and manipulating text. Learning to use these tools will enable you to analyze data efficiently and in novel ways

- Genome annotation, and many other types of data commonly used in genomics, is often provided in the form of plain text tabular data (e.g. tab-delimited or comma-separated formats). Learning to work with and manipulate such data is fundamental to many data analysis tasks, not only in genomics but across the sciences


### Example data

- Yeast Genome Sequence (FASTA) -- https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/146/045/GCF_000146045.2_R64/GCF_000146045.2_R64_genomic.fna.gz
    * Read about the [FASTA format here](https://zhanggroup.org/FASTA/)

- Yeast Genome Annotation (GFF) -- https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/146/045/GCF_000146045.2_R64/GCF_000146045.2_R64_genomic.gff.gz
    * Read about the [GFF annotation format here](./gff-annotation.md)

    - For some details about the yeast genome see: https://www.ncbi.nlm.nih.gov/genome/15 . See the table at the bottom that gives the correspondence between RefSeq IDs and chromosome #'s

- The most convenient way to get these files onto your VM is to use the [`wget` command](#downloading-files-using-wget)

- I also recommend you [setup symbolic links](#setting-up-symbolic-lines-with-ln--s) so you can refer to the files above as `yeast.fna` and `yeast.gff`


### Filtering out GFF comments and meta data

- GFF files usually contain comments and meta data. 




For the analyses we want to do, it will be convient to remove these lines. We can do this with a call to `grep`:

```
grep -v "^#" yeast.gff
```

- Explanation:

    * In grep regular expressions the carrot symbol `^` means "at the beginning of a line". So the pattern `"^#"` means "find lines that start with the hash symbol"

    * The `-v` option means to invert the sense of the matching regular expression (i.e. return all lines that DO NOT match the regular expression)


- Before filtering comments and metadata 

    NOTE: output width truncated to provide compact representation
    ```
    ~$ head yeast.gff 
    ##gff-version 3
    #!gff-spec-version 1.21
    #!processor NCBI annotwriter
    #!genome-build R64
    #!genome-build-accession NCBI_Assembly:G
    #!annotation-source SGD R64-3-1
    ##sequence-region NC_001133.9 1 230218
    ##species https://www.ncbi.nlm.nih.gov/T
    NC_001133.9	RefSeq	region	1	230218	.	+	.
    NC_001133.9	RefSeq	telomere	1	801	.	-	.
    ```

- After filtering comments and metadata

    NOTE: output width truncated to provide compact representation

    ```
    ~$ grep -v "^#" yeast.gff | head 
    NC_001133.9	RefSeq	region	1	230218	.	+	.
    NC_001133.9	RefSeq	telomere	1	801	.	-	.	
    NC_001133.9	RefSeq	origin_of_replication
    NC_001133.9	RefSeq	gene	1807	2169	.	-	.	
    NC_001133.9	RefSeq	mRNA	1807	2169	.	-	.	
    NC_001133.9	RefSeq	exon	1807	2169	.	-	.	
    NC_001133.9	RefSeq	CDS	1807	2169	.	-	0	I
    NC_001133.9	RefSeq	gene	2480	2707	.	+	.	
    NC_001133.9	RefSeq	mRNA	2480	2707	.	+	.	
    NC_001133.9	RefSeq	exon	2480	2707	.	+	.	
    ```




### New commands introduced

For details about each of these commands:
    - Read my overview of the [Unix Core Utilities](https://github.com/bio208fs-class/Bio208_Fall2022/blob/main/workbook/unix-coreutils.md)
    - Then take a look at the `man` pages (e.g. `man echo`) to read about various options 

* `less`
* `head` 
* `tail`
* `echo`
* `cat` and `tac`
* `rev`
* `fold`
* Redirection operators: 
    - `>` = redirect output to a file
        - `echo Hello World > hello.txt`
    - `>>` = append output to a file 
        - `echo Goodbye World >> hello.txt`
    - `<` = redirect input to a command


### Command we didn't have time to discuss in class but I'd like you to explore on your own

* `tr` --  translate (substitute) or delete characters in input. Note that unlike most commands `tr` will not take a file as an argument, so typically you would use `cat` or input redirection to send the contents of a file through `tr`. Example

    - `echo "I am an elite hacker" | tr 'e' '3'` -- substitutes all occurences of "e" with "3"
    - `echo "Can you read sentences without vowels?" | tr -d 'aeiou'` -- delete all vowels
    - `echo AATTAGACCAAC | tr "ATCG" "TAGC"` -- computes the complement of a DNA nucleotide sequences

- `sort` 


### Examples of computations on genome annotation using grep and cut

* Filtering metadata lines out of a GFF file using `grep`

    ```
    grep -E -v "^#" yeast.gff
    ```

* Getting specific columns (seqid = 1, feature type = 3) from the GFF file

    ```
    grep -E -v "^#" yeast.gff | cut -f 1,3
    ```

*  How many features are there on yeast chromosome II (NC_001134.8)?

    ```
    grep -E -v "^#" yeast.gff | cut -f 1 | grep -c "NC_001134.8" 
    ```

*  How many exons are there on yeast chromosome II (NC_001134.8)?

    ```
    grep -E -v "^#" yeast.gff | cut -f 1,3 | grep "NC_001134.8" |  grep -c "exon" 
    ```


-----

## Practical computing tips

### Downloading files using `wget`

- `wget` is a command line tool that can be used to download a file directly to your VM (rather than having to download to your laptop and then re-upload to your VM). The most common usage of wget is of the form:

    ```
    wget URL_OF_FILE
    ```

- For example, to download the FASTA file above to the working directory on your VM you could execute the following command:

    ```
    wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/146/045/GCF_000146045.2_R64/GCF_000146045.2_R64_genomic.fna.gz
    ```

### Decompressing compressed `.gz` files with `gunzip`

- Files that end with `.gz` are typically files that have been compressed with the `gzip` tool to make the data smaller and faster to transmit over the internet. `gzip` is similar to the `zip` utility which is found on most Windows and MacOS computers.

- Before we use such files we need to de-compress them using the `gunzip` command. The general form of this command is:

    ```
    gunzip COMPRESSED_FILE.gz
    ```

- For example, to decompress the FASTA file downloaded above we would execute this command:

    ```
    gunzip GCF_000146045.2_R64_genomic.fna.gz
    ```

    - The resulting decompressed file that is produced is `GCF_000146045.2_R64_genomic.fna`



### Setting up symbolic lines with `ln -s`

- When working with data provided by third parties it is good practice to preserve file naming schemes.  For example, the file name `GCF_000146045.2_R64_genomic.fna` includes important meta information -- "GCF_000146045.2" is the official RefSeq identifier for this version of the yeast genome (see https://www.ncbi.nlm.nih.gov/data-hub/genome/GCF_000146045.2/) and the "R64" tells us this is revision 64. 

- While systematic naming schemes like `GCF_000146045.2_R64_genomic.fna` are important they are often not very "readable" or meaningful to us as users, so we naturally might want to create a more meaningful name for such files

- Symbolic links to files can be used to setup "shortcuts" or "aliases" to files with long names or that are stored in different parts of the filesystem

- The general form of creating a symbolic link is as follows:

    ```
    ln -s ORIGINAL_FILE SHORT_NAME
    ```

- For example, to create an alias with a more easily understood name than `GCF_000146045.2_R64_genomic.fna` we could do the following:

    ```
    ln -s GCF_000146045.2_R64_genomic.fna yeast.fna
    ```

- Once the symbolic link is created `yeast.fna` refers to `GCF_000146045.2_R64_genomic.fna` and we can substitute this symbolic link for the longer name when executing commands. For example, the following is more convenient, but in reality still reads from `GCF_000146045.2_R64_genomic.fna` "behind the scenes":

    ```
    less yeast.fna  
    # equivalent to less GCF_000146045.2_R64_genomic.fna
    ```

- Note that deleting a symbolic link does NOT delete the original file

