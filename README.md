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

####Extract the teosinte and maize information from the full genotype dataset. 

Determine the number of maize individuals (1573)

```
$ grep -c "ZMMIL" fang_et_al_genotypes.txt
290
$ grep -c "ZMMLR" fang_et_al_genotypes.txt 
1256
$ grep -c "ZMMMR" fang_et_al_genotypes.txt 
27
```

Determine the number of teosinte individuals (975)

```
$  grep -c "ZMPBA" fang_et_al_genotypes.txt 
900
$  grep -c "ZMPIL" fang_et_al_genotypes.txt 
41
$  grep -c "ZMPJA" fang_et_al_genotypes.txt 
34
```

Save the column names from the parental file `fang_et_al_genotype.txt` and create new .txt files for maize and teosinte genotype data. Use `$ grep` to pull out the lines for the group identifers of maize (ZMMIL, ZMMLR, ZMMMR) and teosinte (ZMPBA, ZMPIL, ZMPJA) and add to respective, new genotype files.

```
[snodgras@hpc-class UNIX_Assignment]$ head -n 1 fang_et_al_genotypes.txt > maize_genotypes.txt
[snodgras@hpc-class UNIX_Assignment]$ head -n 1 fang_et_al_genotypes.txt > teosinte_genotypes.txt
[snodgras@hpc-class UNIX_Assignment]$ grep "ZMMIL" fang_et_al_genotypes.txt >> maize_genotypes.txt
[snodgras@hpc-class UNIX_Assignment]$ grep "ZMMLR" fang_et_al_genotypes.txt >> maize_genotypes.txt
[snodgras@hpc-class UNIX_Assignment]$ grep "ZMMMR" fang_et_al_genotypes.txt >> maize_genotypes.txt
[snodgras@hpc-class UNIX_Assignment]$ grep "ZMPBA" fang_et_al_genotypes.txt >> teosinte_genotypes.txt
[snodgras@hpc-class UNIX_Assignment]$ grep "ZMPIL" fang_et_al_genotypes.txt >> teosinte_genotypes.txt
[snodgras@hpc-class UNIX_Assignment]$ grep "ZMPJA" fang_et_al_genotypes.txt >> teosinte_genotypes.txt
```

####Join the genotype and position data files
Transpose the genotype data and quality check number of lines

```
[snodgras@hpc-class UNIX_Assignment]$ awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt
[snodgras@hpc-class UNIX_Assignment]$ awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt
[snodgras@hpc-class UNIX_Assignment]$ wc transposed_maize_genotypes.txt 
    986 1551964 6250961 transposed_maize_genotypes.txt
[snodgras@hpc-class UNIX_Assignment]$ wc transposed_teosinte_genotypes.txt 
    986  962336 3884185 transposed_teosinte_genotypes.txt
```

sorting files: use `tail` to remove metadata and column name information %and then sort only the data containing lines

```
#Maize
$ head transposed_maize_genotype.txt # see that data doesn't start until line 4
[snodgras@hpc-class UNIX_Assignment]$ tail -n +4 transposed_maize_genotypes.txt > headerless_maize_genotypes.txt
[snodgras@hpc-class UNIX_Assignment]$ echo $?
0
[snodgras@hpc-class UNIX_Assignment]$ tail -n +2 snp_position.txt > headerless_snp_position.txt
[snodgras@hpc-class UNIX_Assignment]$ echo $?
0
#Teosinte
[snodgras@hpc-class UNIX_Assignment]$ cut -f 1-3 transposed_teosinte_genotypes.txt | head # shows data doesn't start until line 4
[snodgras@hpc-class UNIX_Assignment]$ tail -n +4 transposed_teosinte_genotypes.txt > headerless_teosinte.txt
```

joining files

```
#Maize
[snodgras@hpc-class UNIX_Assignment]$ join -t $'\t' -1 1 -2 1 headerless_snp_position.txt headerless_maize_genotypes.txt > maize_genotypes_positions_all.txt 2> join_std_error.txt
[snodgras@hpc-class UNIX_Assignment]$ echo $?
0
#Teosinte
[snodgras@hpc-class UNIX_Assignment]$ join -t $'\t' -1 1 -2 1 headerless_snp_position.txt headerless_teosinte.txt > teosinte_genotypes_positions_all.txt
[snodgras@hpc-class UNIX_Assignment]$ echo $?
0
```

####Make individual chromosome files for maize and teosinte
The command searches the 3rd  field (chromosome) for a number then prints the entire line with an output field seperator tab and writes the standard output to a new file.

```
$ awk '$3 ~ /1/ {OFS="\t";print $0}' maize_genotypes_positions_all.txt > maize_genotypes_positions_Chr1.txt # repeat for each chromosome, changing the 1 to the next chromosome number 
$ awk '$3 ~ /1/ {OFS="\t";print $0}' teosinte_genotypes_positions_all.txt > teosinte_genotypes_positions_Chr1.txt # repeat for each chromosome, changing the 1 to the next chromosome number
```
####Sort the chromosome files increasing and decreasing with appropriate missing value symbols

Sort by position increasing

```
$ sort -k4,4V maize_genotypes_positions_Chr1.txt | cut -f 1-5 | head/tail #for inspecting to make sure it worked
$ for i in maize_genotypes_positions_Chr*; do sort -k4,4V $i > sorted_inc_$i; done
$ for i in teosinte_genotypes_positions_Chr*; do sort -k4,4V $i > sorted_inc_$i; done
```

Sort by position decreasing

```
$ sort -k4,4 -r maize_genotypes_positions_Chr1.txt | cut -f 1-5 | head
$ for i in maize_genotypes_positions_Chr*; do sort -k4,4 -r $i > sorted_dec_$i; done
$ for i in teosinte_genotypes_positions_Chr*; do sort -k4,4 -r $i > sorted_dec_$i; done
```
Change symbols for the missing data of the decreasing files to "-"

```
$ for i in sorted_dec_maize_genotypes_positions_Chr*; do sed 's/?/-/g' $i > new_$i; done 
$ for i in sorted_dec_teosinte_genotypes_positions_Chr*; do sed 's/?/-/g' $i > new_$i; done 
```

Create a file for all unknown positions

```
$ grep 'unknown' maize_genotypes_positions_all.txt > maize_genotypes_positions_unknown.txt 
$ grep 'unknown' teosinte_genotypes_positions_all.txt > teosinte_genotypes_positions_unknown.txt
```

Create a file for all multiple positions

```
$ grep 'multiple' maize_genotypes_positions_all.txt > maize_genotypes_positions_multiple.txt
$ grep 'multiple' teosinte_genotypes_positions_all.txt > teosinte_genotypes_positions_multiple.txt
```

Headers added back in manually by taking `head -n 1` of `snp_position.txt` and adding the `head -n 1` of `transposed_maize_genotypes.txt` for maize files and `transposed_teosinte_genotypes.txt` for teosinte files (subtracting the common column 1). I realize this is a brittle solution. In retrospect, I would have sorted by position in the `*genotypes_positions_all.txt` files first, added the header back in, and then broken them out to individual chromosome files. Did not have time to redo it this way, hence the more brittle solution of manual changes. 