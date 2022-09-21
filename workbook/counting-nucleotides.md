
# Counting nucleotides


## Exemplar data

E coli genome -- https://www.ncbi.nlm.nih.gov/genome/?term=e+coli

https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_genomic.fna.gz

## Strip FASTA headers

```
grep -v "^>" ecoli.fna > ecoli_noheaders.fna
```


## search and count using grep

```
cat ecoli_noheaders.fna | grep -o -i "A" | wc -l
```

## fold and count using fold and grep

```
cat ecoli_noheaders.fna | fold -w 1  | grep -c -i "A"
```

## reduce to nucleotide of interest and count

```
cat ecoli_noheaders.fna | tr -d -c "Aa" | wc -c
```