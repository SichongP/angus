# Short read quality and trimming

Learning objectives:
- Install software (fastqc, multiqc) via conda
- download data
- visualize read quality
- quality filter and trim reads


Start up a Jetstream m1.medium or larger
[as per Jetstream startup instructions](jetstream/boot.html).

---

You should now be logged into your Jetstream computer!  You should see
something like this

```
titus@js-17-71:~$ 
```

## Getting started

Change to your home directory:

```
cd ~/
```

Let's make sure our conda channels are loaded, just in case.

```
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```

and install FastQC, MultiQC, and trimmomatic:

```
conda install -y -c bioconda fastqc multiqc trimmomatic
```


## Data source

Make a "data" directory:

```
cd ~/
mkdir -p data
cd data
```

and download some data from the
[Schurch et al, 2016](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4878611/)
yeast RNAseq study. We will download three wildtype yeast datasets, and
three mutant datasets (*SNF2* knockout; *SNF2* is a global transcription
regulator). 

```
curl -L https://osf.io/5daup/download -o ERR458493.fastq.gz
curl -L https://osf.io/8rvh5/download -o ERR458494.fastq.gz
curl -L https://osf.io/2wvn3/download -o ERR458495.fastq.gz
curl -L https://osf.io/xju4a/download -o ERR458500.fastq.gz
curl -L https://osf.io/nmqe6/download -o ERR458501.fastq.gz
curl -L https://osf.io/qfsze/download -o ERR458502.fastq.gz
```

Let's make sure we downloaded all of our data using [md5sum](https://en.wikipedia.org/wiki/Md5sum). 
An md5sum hash is a fingerprint that lets you compare with the original file. 
Some sequencing facilities will provide a file containing md5sum hashes for the 
files being delivered to you. By comparing your md5sum with the original md5sum 
generated by the sequencing facility, you can make sure you have downloaded your 
files completely. If there was a disruption that occurred during the download 
process, not all the bytes of the file will be transferred properly and you will 
likely be missing some of your sequences.

```
md5sum *.fastq.gz
```

You should see this:

```
2b8c708cce1fd88e7ddecd51e5ae2154  ERR458493.fastq.gz
36072a519edad4fdc0aeaa67e9afc73b  ERR458494.fastq.gz
7a06e938a99d527f95bafee77c498549  ERR458495.fastq.gz
107aad97e33ef1370cb03e2b4bab9a52  ERR458500.fastq.gz
fe39ff194822b023c488884dbf99a236  ERR458501.fastq.gz
db614de9ed03a035d3d82e5fe2c9c5dc  ERR458502.fastq.gz
```

To check whether your md5sum hashes match with a file containing md5sum hashes:

```
md5sum *.fastq.gz > err_md5sum.txt
md5sum -c err_md5sum.txt
```
(First we're creating a file containing md5sum of the files, then checking it. 
In reality, you wouldn't be making this file to check. The facility would be 
creating it for you.)

You should see this:
```
ERR458493.fastq.gz: OK
ERR458494.fastq.gz: OK
ERR458495.fastq.gz: OK
ERR458500.fastq.gz: OK
ERR458501.fastq.gz: OK
ERR458502.fastq.gz: OK
```

Now if you type:

```
ls -l
```

you should see something like:

```
-rw-rw-r-- 1 titus titus  59532325 Jun 29 09:22 ERR458493.fastq.gz
-rw-rw-r-- 1 titus titus  58566854 Jun 29 09:22 ERR458494.fastq.gz
-rw-rw-r-- 1 titus titus  58114810 Jun 29 09:22 ERR458495.fastq.gz
-rw-rw-r-- 1 titus titus 102201086 Jun 29 09:22 ERR458500.fastq.gz
-rw-rw-r-- 1 titus titus 101222099 Jun 29 09:22 ERR458501.fastq.gz
-rw-rw-r-- 1 titus titus 100585843 Jun 29 09:22 ERR458502.fastq.gz
```

These are six data files from the yeast study.

One problem with these files is that they are writeable - by default,
UNIX makes things writeable by the file owner.  This poses an issue
with creating typos or errors in raw data.  Let's fix that before we
go on any further:

```
chmod a-w *
```

Take a look at their permissions now -- 
```
ls -l
```

and you should see that the 'w' in the original permission string
(`-rw-rw-r--`) has been removed from each file and now it should look like 
`-r--r--r--`.

We'll talk about what these files are below.

### 1. Linking data to our working location

First, make a new working directory:
```
mkdir -p ~/quality
cd ~/quality
```

Now, we're going to make a "link" to our quality-trimmed data in our current working directory:

```
ln -fs ~/data/* .
```

and you will see that they are now linked in the current directory when you do an
`ls`. These links save us from having to specify the full path (address) to their location on the computer, without us needing to actually move or copy the files. But note that changing these files here still changes the original files!

These are FASTQ files -- let's take a look at them:

```
gunzip -cd ERR458493.fastq.gz | head
```

FASTQ files are sequencing files that contain reads. These come off of the 
sequencer and contain information about the quality of the reads. A score 
of 0 is the lowest quality score, and a score of 40 is the highest. Instead
of typing the scores as numbers, they are each encoded as a character.   

```
Quality encoding: !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHI
                  |         |         |         |         |
Quality score:    0........10........20........30........40  
``` 

A score of 10 indicates a 1 in 10 chance that the base is accurate. A score
of 20 is a 1 in 100 chance that the base is accurate. 

Links:

* [FASTQ Format](http://en.wikipedia.org/wiki/FASTQ_format)

### 2. FastQC


We're going to use 
[FastQC](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
summarize the data. We already installed 'fastqc' above, with the
conda command.

Now, run FastQC on two files:

```
fastqc ERR458493.fastq.gz
fastqc ERR458500.fastq.gz
```

Now type 'ls':

```
ls -d *fastqc.zip*
```

to list the files, and you should see:

```
ERR458493_fastqc.zip
ERR458500_fastqc.zip
```

Inside each of the fastqc directories you will find reports from the fastqc 
program. 

We will transfer the html files to our local computers so we can open and view
them in our browsers. We do this because most remote computers do not have the
ability to view images and htmls.  We will use the command `scp` to do this. 

`scp` stands for secure copy. We can use this command to transfer files between
two computers. We need to run this command on our *local* computers (i.e. from
your laptop).

The `scp` command looks like this:

```
scp <file I want to move> <where I want to move it>
```

First we will make a new directory on our computer to store the HTML files we’re
transfering. Let’s put it on our desktop for now. Make sure the terminal program
you used this morning to `ssh` onto your instance is open. If you're in it now,
you can open a new tab in (you can use the pull down menu at the top of your 
screen or the Cmd+t keyboard shortcut). Type:

```
mkdir -p ~/Desktop/fastqc_html 
```

Now we can transfer our HTML files to our local computer using `scp`.

```
scp username@ipadress:~/quality/*.html ~/Desktop/fastqc_html
```

The first part of the command  is the address for your remote 
computer. Make sure you replace everything after username@ with your instance 
number (the one you used to log in).

The second part starts with a : and then gives the absolute path of the files 
you want to transfer from your remote computer. Don’t forget the `:`. We used a 
wildcard (\*.html) to indicate that we want all of the HTML files.

The third part of the command gives the absolute path of the location you want 
to put the files. This is on your local computer and is the directory we just 
created `~/Desktop/fastqc_html`.

You will be prompted for your password. Enter the password we set for your 
instance  during the morning session.

You should see a status output like this:

```
ERR458493_fastqc.html                      100%  642KB 370.1KB/s   00:01    
ERR458500_fastqc.html                      100%  641KB 592.2KB/s   00:01  
```

Now we can go to our new directory and open the HTML files.

You can also download and look at these copies of them:

* [ERR458493_fastqc.html](http://htmlpreview.github.com/?https://github.com/ngs-docs/angus/blob/2018/_static/ERR458493_fastqc.html)
* [ERR458500_fastqc.html](http://htmlpreview.github.com/?https://github.com/ngs-docs/angus/blob/2018/_static/ERR458500_fastqc.html)

These files contain a lot of information! Depending on your application, some
information is more helpful than other. For example, for an RNA-seq example, 
failing duplication levels isn't necessarily a problem -- we expect to have
duplicated sequences given that some transcripts are very abundant.

Questions:

* What should you pay attention to in the FastQC report?
* Is one sample better than another?

Links:

* [FastQC](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
* [FastQC tutorial video](http://www.youtube.com/watch?v=bz93ReOv87Y)

There are several caveats about FastQC - the main one is that it only
calculates certain statistics (like duplicated sequences) for subsets
of the data (e.g. duplicate sequences are only analyzed for the first
100,000 sequences in each file


### 3. Trimmomatic

Now we're going to do some trimming!  Let's switch back to our instance
terminal, as we will be running these commands on our remote instance.
We'll be using
[Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic), which
(as with fastqc) we've already installed via conda.

The first thing we'll need are the adapters to trim off. This command is long,
but it's because the trimmomatic distribution comes with adapter files. 

```
cp /opt/miniconda/pkgs/trimmomatic-*/share/trimmomatic-*/adapters/TruSeq2-PE.fa .
```

(you can look at the contents of this file with `cat TruSeq2-PE.fa`)

Now, to run Trimmomatic on both of them:

```
trimmomatic SE ERR458493.fastq.gz \
        ERR458493.qc.fq.gz \
        ILLUMINACLIP:TruSeq2-PE.fa:2:40:15 \
        LEADING:2 TRAILING:2 \
        SLIDINGWINDOW:4:2 \
        MINLEN:25
        
trimmomatic SE ERR458500.fastq.gz \
        ERR458500.qc.fq.gz \
        ILLUMINACLIP:TruSeq2-PE.fa:2:40:15 \
        LEADING:2 TRAILING:2 \
        SLIDINGWINDOW:4:2 \
        MINLEN:25
        
```

You should see output that looks like this:

```
...
Input Reads: 1093957 Surviving: 1092715 (99.89%) Dropped: 1242 (0.11%)
TrimmomaticSE: Completed successfully
```

We can also run the same process for all 6 samples more efficiently using a `for` loop, as follows:

```
for filename in *.fastq.gz
do
        # first, make the base by removing fastq.gz
        base=$(basename $filename .fastq.gz)
        echo $base

        trimmomatic SE ${base}.fastq.gz \
                ${base}.qc.fq.gz \
                ILLUMINACLIP:TruSeq2-PE.fa:2:40:15 \
                LEADING:2 TRAILING:2 \
                SLIDINGWINDOW:4:2 \
                MINLEN:25
done
```

This script will go through each for the filenames that end with `fastq.gz` and 
run Trimmomatic for it.

Questions:

* How do you figure out what the parameters mean?
* How do you figure out what parameters to use?
* What adapters do you use?
* What version of Trimmomatic are we using here? (And FastQC?)
* Do you think parameters are different for RNAseq and genomic data sets?
* What's with these annoyingly long and complicated filenames?

For a discussion of optimal trimming strategies, see 
[MacManes, 2014](http://journal.frontiersin.org/Journal/10.3389/fgene.2014.00013/abstract) -- it's about RNAseq but similar arguments should apply to metagenome
assembly.

Links:

* [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic)

### 4. FastQC again

Let's take a look at the output files:

```
gunzip -cd ERR458493.qc.fq.gz | head                                                        
```

It's hard to get a feel for how different these files are by just looking at 
them, so let's run FastQC again on the trimmed files:

```
fastqc ERR458493.qc.fq.gz
fastqc ERR458500.qc.fq.gz
```

And now view my copies of these files: 

* [ERR458493.qc_fastqc.html](http://htmlpreview.github.com/?https://github.com/ngs-docs/angus/blob/2018/_static/ERR458493.qc_fastqc.html)
* [ERR458500.qc_fastqc.html](http://htmlpreview.github.com/?https://github.com/ngs-docs/angus/blob/2018/_static/ERR458500.qc_fastqc.html)

Alternatively, if there's time, you can use `scp` to view the files from your 
local computer. 

### 5. MultiQc
[MultiQC](http://multiqc.info/) aggregates results across many samples into a single report for easy comparison.

Run Mulitqc on both the untrimmed and trimmed files

```
multiqc .
```

And now you should see output that looks like this:

```
[INFO   ]         multiqc : This is MultiQC v1.0
[INFO   ]         multiqc : Template    : default
[INFO   ]         multiqc : Searching '.'
Searching 15 files..  [####################################]  100%
[INFO   ]          fastqc : Found 4 reports
[INFO   ]         multiqc : Compressing plot data
[INFO   ]         multiqc : Report      : multiqc_report.html
[INFO   ]         multiqc : Data        : multiqc_data
[INFO   ]         multiqc : MultiQC complete
```

You can view the output html file
[multiqc_report.html](http://htmlpreview.github.com/?https://github.com/ngs-docs/angus/blob/2018/_static/multiqc_report.html) by going to RStudio, selecting the file, and saying "view in Web browser."

Questions:

* is the quality trimmed data "better" than before?
* Does it matter that you still have adapters!?
