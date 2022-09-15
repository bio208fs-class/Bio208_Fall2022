## GFF annotation format

* GFF annotation files are tab-delimited files with nine columns as described in the [GFF specification](https://github.com/The-Sequence-Ontology/Specifications/blob/master/gff3.md)

    - The most important columns for our purposes are:
        
        * Column 1, seqid -- the identifier for the landmark sequence that the feature "sits on". In the case of GFF files from RefSeq or Genbank the seqid is the accession number of the corresponding landmark sequence from the genome of interest. Usually these landmark sequences are whole chromosomes, but they are sometimes  contigs if the assembly is not complete

        * Column 3, type -- the type of feature documented. Examples include terms like "gene", "exon", "centromere", etc.

        * Columns 4 and 5 -- the start and end coordinates for the feature (1-indexed) with respect to the coordinate system of the landmark sequence the feature sits on.

        * Column 8, strand -- the strand of the feature,  "+" indicates the positive strand, "-" indicates the minus strand. Strandedness of DNA is an abritrary convention but is important for understanding arrangement of features in the coordinate system of the landmark sequences.

        * Column 9, attributes -- this is a freeform column where additional information is described in a `tag=value` format with multiple attributes being separated by semi-colons (e.g. `tag1=value1;tag2=value2`). Typically this column includes things like feature IDs, common names (e.g. gene names used in the literature), notes about gene function, references to other databases, etc.

* In addition to the nine columns described above, GFF files can include comments and meta-data lines

    - Comment lines start with a single hash symbol,`#` 
    - Meta-data lines start with a double hash symbol, `##`

