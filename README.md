#BCB546X Unix Assignment

##File Inspection
###Requirements for the assignment

Inspect the following files to describe their structure and dimensions (file size, number of columns, number of lines, etc.): 

* `fang_et_al_genotypes.txt`: published SNP dataset with maize, teosinte, Tripsacum individuals represented
* `snp_position.txt`: includes information about the SNPs in `fang_et_al_genoyptes.txt`, notably 1st column = SNP id, 3rd column = chromosome location, 4th column = nucleotide location

###Workflow
Working in the directory: `/home/snodgras/BCB546X-Fall2018/assignments/UNIX_Assignment` on hpc-class 

* Look at file size for both files

```
$ du -h *.txt
11 M	fang_et_al_genotypes.txt
84K		snp_position.txt
``` 

* Make sure all characters are ASCII characters

```
$ file *.txt 
fang_et_al_genotypes.txt:	ASCII text, with very long lines
snp_position.txt:			ASCII text
```

* Get number of lines, words, bytes

```
2783	2744038	11051939	fang_et_al_genotypes.txt
984		13198		82763		snp_position.txt
3767	2757236	11134702	total
```

* Looking at the top of the file, note if there is metadata at the top of the file 

```
$ head -n 2 fang_et_al_genotypes.txt  
#so much data I won't print it here, but the column names go Sample_ID  JG_OTU  Group  ...SNP identifiers... and the following rows have the sample id name then nucleotide/nucleotide or ?/? 
#no metadata outside of column names
$ head -n 2 snp_position.txt 
SNP_ID	cdv_marker_id 	Chromosome	Position    alt_pos   mult_positions   amplicon   cdv_map_feature.name   gene   	candidate/random	   Genaissance_daa_id   Sequenom_daa_id   count_amplicons   count_cmf   count_gene
abph1.20	5976	2	27403404	abph1	AB042260   abph1   candidate   8393   10474   1   1   1
#no metadata outside of column names
```

* Look at the bottom of the file

```
$ tail -n 2 fang_et_al_genotypes.txt
$ tail -n 2 snp_position.txt
#similar to what was seen in the 2nd line of the header command
```

* Get number of columns

```
$ awk -F "\t" '{print NF; exit}' snp_position.txt
15 
$ awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
986
```

##Data Processing
###Requirements for assignment
Format these two data files for downstream analysis. Join the datasets so both genotypes and positions are recorded in a series of input files with the first column = SNP_ID, second column = Chromosome, third column = Position, and subsequent columns as genotype data for _either_ maize or teosinte individuals

####For Maize = Group ZMMIL, ZMMLR, ZMMMR in column 3 of `Fang_et_al_genotypes.txt`

Make 22 files total:

* 10 files (1/chromosome) with SNPs ordered based on increasing position values and with missing data encoded 	by '?'
* 10 files (1/chromosome) with SNPs ordered based on decreasing position values and with missing data encoded by 	'-'
* 1 file with all SNPs with unknown positions in the genome (no particular order necessary)
* 1 file with all SNPs with multiple positions in the genome (no particular order necessary)

####For Teosinte = Group ZMPBA, ZMPIL, ZMPJA in column 3 of `Fang_et_al_genotypes.txt`
Make 22 files total:
	
* 10 files (1/chromosome) with SNPs ordered based on increasing position values and with missing data encoded 	by '?'
* 10 files (1/chromosome) with SNPs ordered based on decreasing position values and with missing data encoded by 	'-'
* 1 file with all SNPs with unknown positions in the genome (no particular order necessary)
* 1 file with all SNPs with multiple positions in the genome (no particular order necessary)
	
###Workflow
* check to see if the file is sorted `sort -c | echo $?`
* `sort -V` allows you to detect numbers in strings
* `grep -v "^#" Mus_musclus.GRCm38.75_chr1.gtf | cut -f3 |sort | uniq -c | sort -n` means grab everything except a header (line that starts with #), cut so only the 3 column remains, sort and then count uniqs, then sort by increasing number/counts
* `join -1 1 -2 1 example_sorted.bed example_lengths.txt > example_with_lengths.txt` look at file 1 column 1, file 2 column 1 and join them based on those values and send to new file.
* After a join inspect the new file to make sure everything was joined `wc -l files`
* `sed` as a find and replace command, though capable of more