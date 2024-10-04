# CHAP_Analyzer
Prerequisites
1. Rstudio
2. dplyr package installed in Rstudio
3. GoogleDrive sinked to your mac book so you can run this script locally

Rscript used to manipulate the CHAP output so that manual data curation is reduced. This can help users select individuals that have editing profiles of interest. 
The script reads in CHAP and CHAP.Stats files from a specific SRU run. These files are merged in order to associate PED-IDs, Aliases, and Pedigrees housed in Chap.stats with the Deletion information housed in CHAP. 
The merged data undergoes a series of modifications to be able to perform analysis on PED-IDs with the Boosted gene targets. 
The final output provides an indepth view of editing results for each plant material submitted for sequencing. 
In particular the deletion size is available as a stand alone column as are the reads associated with that deletion, and the percent read representation for that deletion relative to all reads associated with the Plant material + Gene target combination (e.g. E-PED060-100+Spo11). 
There are also columns dedicated to providing a threshold for reliable data. These are used to calculate whether target genes have been fully knocked out in an individual.

It is important to note that there is a current filter geared toward tetraploid crops. The filter removes reads that are deemed uninformative for analysis. For a given plant material + Gene target combination (e.g. E-PED060-100+Spo11) the two largest read percentages are reported if the read_percent sum of the top two is >= 75
For example if the top read is 92.9 percent of the reads reported and the second most reported read represents 2.4 percent of the reads for this target the remainnig rows with read representation >2.4 percent are discarded. Since the sum of these two results in a value >= 75 it is considered "diploid' and the remaining reads are deleted. 
If the read_percent representation was closer to that of a tetraploid we would expect 4 reads with ~25 percent representation. so If the top two reads (by read representation) are 40% and 25% (less than 75 when summed) the next two deletions with the highest read count are retained. 
This is based on the assumption that a diploid would have an expected read representation of 50% for one haplotype and 50% for the other. Tetraploids would provide 4 haplotypes at 25% read representation each. 
The reason for this >75% threshold is as follows: If the individual were tetraploid we expect 4 haploltypes at 25% read representation each. Assuming there are 100 reads for a target in total (low end of what we would consider informative) The expected variance is nxpx(1-p) = 100 x .25 x .75 = 18.75 
The standard deviation is var^.5 = 18.75^.5 = ~4.33
This result is 4 standard deviations greater than the expected 50 reads for two haplotypes summed together. 
The z-score is (75-50)/(18.75+18.75)^.5 = 25/18.5^.5 = 25/6.12 = ~4.08
A z-score of 4.08 corresponds to ~.002% chance of seeing that result

Column Outline:
library_ID: The lib ID for the plant material run through iseq
plant_material: The Plant Material ID
PED_Target: The Plant Material ID + Gene Target which is used for data manipulation within the script
alias: The well location alias associated with the Plant Material ID
SRU: SRU number
Plate_Name: The name of the plate that the Plant Material ID belongs to
plate_ID: SRU plate ID
pedigree: The pedigree of the plant being sequenced
DNA: Gene target
deletion: The deletion size for a reported read
reads: The read count for a single row associated with a plant material + Gene target combination (e.g. E-PED060-100+Spo11). *Example* 253 reads for an 11 base pair deletion for Spo11 for E-PED060-100
read_percent: The percent representation of the specific read based on the total number of reads for a lant material + Gene target combination (e.g. E-PED060-100+Spo11). 
Reads_per_target: The total number of reads for a given plant material + Gene target combination (e.g. E-PED060-100+Spo11). Total Spo11 reads for E-PED060-100
prs: The Benchling PRS code for the targeted gene sequence
D_Count: Counts the number of deletions that have occured for a given read
PRS_RES_DEL: Currently a column under construction. Will likely be changed for Insertion or SNP tracking
rmrna: Provides the Benchling RMRNA code for the targeted gene sequence
cigar_number: Provides deletion size of deleted sequence and location of this deletion
cigar_sequence: Provides ATGC string of deleted sequence and location of this deletion
Legit_Read: Provides a yes/no call based on two criteria, If the row has less than 8 reads or 8% read representation it is considered unreliable and labeled no. 
FS_Edit: Provides a yes/no call based on whether the deletion column value is divisible by 3. If it is it gets a no. If it is not it gets a yes.
Rec8_Edited: Provides a yes/no call based on whether a row with PRS078 (Rec8) has a deletion that is not a 3 base pair deletion (bpd) or has a 3bpd or is WT. If it has a 3bpd or is      WT it is not considered edited and recieves a no. All other deletion sizes are considered Yes.
Spo11_Edited: Provides a yes/no call based on whether a row the PRS257 (Spo11) has any deletion other than a 6bpd that occurs at location 6D2017 in the cigar string. If there is a       deletion and it is not a 6bpd that occurs at location 6D2017 then it is considered knocked out (recieves a yes) otherwise it recieves a no.
Full_FS: Provides a yes/no call based on whether the specific Gene target has frameshift edits for all reads associated with that specific plant material and gene target. If the two TAMg9 deletions are not divisible by 3 (two yes's in the FS_Edit column) then it the two Full_FS columns will get Yes's
Responsible_PRS: Provides the PRS that was fully knocked out. So if all reads are yes's for a target in the Full_FS column then print the PRS#. If TAMg9 has 2 FS knockouts then "PRS658" is printed
Keeper: Provides a codename for the editing profile achieved. For example "TableSpark" is an individual edited for all three boosted targets TAM,Rec, and Spo.
