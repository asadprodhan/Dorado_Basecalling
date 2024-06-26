# **How to re-basecall old Nanopore runs with Dorado?** <br />


### **AUTHOR: Dr Asad Prodhan** https://asadprodhan.github.io/


<br />


<br />


<p align="center">
  <img 
    src="https://github.com/asadprodhan/Dorado_Basecalling/blob/main/2024-06-01_DoradoVSGuppy_v2.png"
 align="center" width=45% height=45% >   
</p>
<p align = center>
Re-basecalling old Nanopore runs with Dorado.
</p>

<br />


<br />


## **Install softwares**



### **Create and activate a conda environment**


```
conda create -n dorado
```


```
conda activate dorado
```


<br />


### **Install Nextflow and Singularity**



- Install Nextflow



```
conda install -c bioconda nextflow
```


- Run the following command to make sure that Nextflow has been installed


```
nextflow -h
```


> If you see the Nextflow options, then the Nextflow has been installed



- Install Singularity


```
conda install -c conda-forge singularity
```



- Run the following command to make sure that Singularity has been installed


```
singularity -h
```


<br />


### **Install pod5**


```
pip install pod5
```


<br />


### **Add the pod5 executable path to PATH variable permanently**


```
export PATH=$PATH:/path/to/the/pod5/executable
```


<br />



### **Install Dorado basecaller**



```
wget https://cdn.oxfordnanoportal.com/software/analysis/dorado-0.6.2-linux-x64.tar.gz
```


> More deatils: https://github.com/nanoporetech/dorado?tab=readme-ov-file 


```
tar -zvxf dorado-0.6.2-linux-x64.tar.gz
```


<br />


### **Add the dorado executable path to PATH variable permanently**



```
nano .bashrc
```


- Add the following line to the .bashrc profile


```
export PATH=$PATH:/xx/xx/bin/dorado-0.6.2-linux-x64/bin
```


<br />


<br />


## **Analyse data**


### **Convert the fast5 files into pod5**



```
pod5 convert fast5 sample01.fast5 --output sample01.pod5
```



Loop function:



```
for FILE in ./fast5/*.fast5; do FILENAME=$(basename "$FILE" .fast5); pod5 convert fast5 "$FILE" --output ./pod5/${FILENAME}.pod5; done
```


** It can create the pod5 directory itself**


<br />


### **Basecall from the pod5 file**


https://www.youtube.com/watch?v=wBDU0X1ZCco


```
dorado basecaller sup --emit-fastq barcode07.pod5 > barcode07.fastq
```


> DNA model and its latest version auto selected when you select the mode-fast,hac,sup



Dorado generates unaligned bam file by default. Many useful information about each read are added in the bam tags, which can be extracted by the Dorado summary command.

 

If you want to generate fastq file instead of bam, the use the "--emit-fastq" flag 



Loop function [DNA model and its latest version auto selected]:



```
for FILE in ./pod5/*.pod5; do FILENAME=$(basename "$FILE" .pod5); dorado basecaller sup --emit-fastq "$FILE" > ./fastq/${FILENAME}.fastq; done
```


<br />



## **If you want to hand-pick the DNA model for your old Nanopore runs, then follow the following steps**



Loop function [DNA model hand-picked]:



**First, download the model of your interest:**


```
dorado download --list
```


```
dorado download --model dna_r9.4.1_e8_sup@v3.6
```



**Then run the loop function:**



``` 
for FILE in ./pod5/*.pod5; do FILENAME=$(basename "$FILE" .pod5); dorado basecaller --recursive dna_r9.4.1_e8_sup@v3.6 --emit-fastq "$FILE" > ./fastq/${FILENAME}.fastq; done
```



> It can't create the fastq directory, need to create one beforehand



<br />


## **If you want to collect fastq read summary**



```
dorado summary barcode07.bam > barcode07_seq_info.tsv
```


Alternatively, you can use the **fastqc** program 



<br />


## **Dorado de-multiplexing**



For de-multiplexing, all files need to be concatenated first. Otherwise, the loop function replaces the previous files



```
dorado demux --kit-name SQK-RBK004 --emit-fastq --output-dir ./barcodes beforeDemuxed.fastq
```


Loop function to check record numbers in each fastq file



```
for FILE in ./*.fastq; do grep -c "@" "$FILE"; done
```


<br />


<br />




**Further reading**


https://github.com/nanoporetech/dorado#duplex


https://nanoporetech.com/platform/accuracy/duplex


