# GeneCoexpress
A set of command line steps to create a gene co-expression network for a specific tissue of interest. Use the final output to create a Cytoscape map.

## File Information
First, you will need to obtain a gene regulatory network from the following link.
```
https://www.cell.com/cms/10.1016/j.celrep.2017.10.001/attachment/e7309c03-e579-4119-a95e-376ab2066cbb/mmc2.csv
```
Then, separate your tissue of interest from the others using the following command line. Be sure to replace "tissue" with your tissue of interest.
```
(head -n 1 mmc2.csv; grep 'tissue' mmc2.csv) > grn.csv
```
Obtain a gene co-expression network for your tissue of interest using a scientific paper or database. I used the following link.
```
https://gsajournals.figshare.com/ndownloader/files/31186971
```
Next, go to Ensemble's BioMart and select Human Genes.
Change the attributes to include the following:
Gene stable ID
Gene stable ID version
Transcript stable ID
Transcript stable ID version
Gene name

Save and download to your computer as a .csv file and upload to your working directory.

Make sure your files are tab-delimited using the following command line.
```
cat grn.csv | sed 's/\"//g' | sed 's/,/\t/4;s/,/\t/3;s/,/\t/1;s/,/\t/1' > grn.tab #Convert to tab-delimited format
```
Ensure the sqlite database is in your $PATH before continuing.

After, use sqlite to prepare the files. Be sure to replace tissue with your tissue of interest.
```
module load sqlite
sqlite3 mapping
.separator "\t"
.import grn.tab grn
.separator "\t"
.import gcn.tab gcn
.mode csv
.import mart_export.txt names
.separator "\t"
.headers on
.output mapped.tsv
SELECT TF, names."Gene Name", Tissues, TargetGene
FROM grn
INNER JOIN names on names."Gene stable ID"=grn.TargetGene
WHERE Tissues LIKE '%tissue%'; 
.quit
```

Merge the GRN and GCN files.
```
cat mapped.tsv| awk '{print $1,$2}' > temp
awk '{print "GRN " $0}' temp   > temp2
sed '1s/GRN/NetworkType/g' temp2  > temp3
sed '1s/"Gene/GeneTarget/g' temp3  > GRN_edges.tab
rm temp temp2 temp3
```
```
cat gcn.tab | awk '{print $1,$2}' > temp
awk '{print "GCN " $0}' temp   > temp2
sed '1s/GCN/NetworkType/g' temp2  > GCN_edges.tab
rm temp temp2
```
Merge the files and drop duplicates.
```
(tail -n +2 GRN_edges.tab; tail -n +2 GRN_edges.tab; tail -n +2 GCN_edges.tab) > merged.gcn.grn.tab
cat merged.gcn.grn.tab | uniq | sed 's/\s/\t/g' > unique.merged.gcn.grn.tab
```
