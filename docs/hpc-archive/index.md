# HPC Archiving

This document covers the usage of the Archiving Storage attached to LIMS-HPC.

## Purpose

The [Australian Code for the Responsible Conduct of Research](https://www.nhmrc.gov.au/guidelines-publications/r39) 
requires researchers to make use the archive storage at a research organisation (i.e. La Trobe University) and keep
records about such data.  La Trobe University's policies specifically reference the Code.

**Relevant Documents**:

* [Australian Code for the Responsible Conduct of Research](https://www.nhmrc.gov.au/guidelines-publications/r39)
* [La Trobe University Policies](https://policies.latrobe.edu.au/)
* [La Trobe University Research Data Management Policy](https://policies.latrobe.edu.au/document/view.php?id=106)

## Glossary

Key terms used within this document

* **Project**: a directory of related files.  These may include multiple collections of data (i.e. multiple 
    sequencing runs) that are related.
* **Active Project**: a project that is being worked on within the last 30 days.
* **Inactive Project**: a project that has ceased to be active.  This might be because of the project coming 
    to an end or being shelved for a period of time.
* **Meta-data**: information about your data.
* **Meta-data file**: In this context, a text file contained within the top level directory for each project.  The
    purpose of this file is to store a description of the data contained within the project.
* **Computational storage**: Storage that is used within jobs (either reading/writing).


### LIMS-HPC key directories

* **/home/USERNAME**: Home directory for the user USERNAME.  No data will be stored here; only config files
* **/home/group/LABGROUP**: the directory for storing *active* projects of research group LABGROUP
* **/home/archive/LABGROUP**: the directory for storing *inactive* projects of research group LABGROUP
* **/data/genomics-archive**: each sequencing run is stored here.  Any files within here do NOT need to be archived.  If 
    you have sequencing runs from outside providers then you can have them stored here by contacting the 
    [Genomics Platform (genomics@latrobe.edu.au)](mailto:genomics@latrobe.edu.au)  

## Archive types

The Archive storage on LIMS-HPC is used for storing data that is no longer being actively processed.  There are two
groups of archived projects: 

### Published

Data analysis that has been published (or is under review).  This data must be cleaned, compressed and have a COMPLETE
meta-data file.  Once the project is ready it will be *locked* so that no users can add, change or delete the files within 
it.  A locked project will only be readable for researchers within the labgroup and not changeable.  A *supplementary* 
sub-directory for each project is used for adding new files/information after the project is locked.  Files within here will
also be locked when ready.

### Shelved

For projects that have been put on hold for the time being.  They can be brought back to the HPC storage (i.e. 
/home/group/*) when work resumes.

## Archive directory layout

Each labgroup's archive directory contains two sub-directories for the Published and Shelved data.

* **/home/archive/LABGROUP/pub**: Research projects that result in published data are stored inside the *pub* directory
* **/home/archive/LABGROUP/shelf**: Research projects that are temporarily shelved are stored inside the *shelf* directory

## Meta-data

Items that must be included as a minimum in archived data (metadata.txt)

* **Contacts**: Phone, email addresses, department of:
	* *Primary*: the researchers who created the results/performed research.
	* *Supervisor/lab group*:
	* *Collaborators*: Internal and external
* **Approvals**:
	* *Incoming*: A list of approvals that were received by you for use of external data if any. (attach email text if 
	  coming via email, best for forward the email then copy all text so it includes the from/to fields)
	* *Outgoing*: A list of approvals that you have given out for other copies of this research data including contact 
	  details and names of persons and for what purpose they were given.
* **Confidentiality/Ethics restrictions**:
	* Are there any restrictions on the confidentiality of all or parts of this data.  Which parts if any can be given 
	  out to others if a request comes from publication?
* **Expiry**:  NOTE: data should only be deleted if it has no useful purpose.  If the expiry data comes around and it's 
  still useful then a new expiry should be set
	* What date should this data be deleted (See section 2.1.1 of r39 of the "AUSTRALIAN CODE FOR THE RESPONSIBLE CONDUCT 
	  OF RESEARCH" for minimum time frames required).
	* Who is responsible for the disposal (or extension)
	* Note any challenges or allegations of research misconduct that are raised against this research.  While unresolved
	  cases remain the results MUST NOT be deleted.
* **References**:
	* Lab books and other documentation that describes the data processing and/or wet lab work associated with this data.
	* *Publications*: Citation and doi/url for any publications that are a result of this data

**[metadata.txt](metadata.txt)**: a template file to use for your projects metadata.txt


## What to keep

<img width="100%" src="../hpc-archive/process.png" />
*Data-wise*, the code says you need to keep at least the raw input files, the output results and anything that is not possible to 
reproduce in your data analysis.  You might also want to keep any intermediate results that take a significant time to reproduce.
Any dataset that you download from the web to use in your analysis should be archived with your results as it might be hard to
recreate the dataset in the future (if the web resource disappears or changes).

*Meta-data* wise, you need to keep all meta-data which includes all your job scripts for each step of your analysis.

Don't keep SAM files particularly if you have BAM files of the same contents.  When developing your processing pipeline consider
a method which produces BAM files directly if possible.

## Copying data

### Preparation

Prior to transferring you need to cleanup the project directory.  See the *Best Practices* section below for the tasks that should
be completed. 

All FASTQ/A files should be compressed as do any other large text files.  When you compress a text file that wasn't compressed when 
you ran pipeline make sure you document that it was done in a README file so that you remember to decompress it if you need to 
perform processing again.


### Transferring

You should transfer the prepared project by 'copying' it rather then 'moving' it; the archive storage is on a separate filesystem
so there is no speed advantage by 'moving' it.  Worse, if the transfer fails part way through the 'move', it is much harder to 
continue afterwards.

Given that both the archive and computational storage are 'mounted' to the LIMS-HPC filesystem (i.e. you can access it by a regular 
unix path), you could use the standard unix *cp* command however this is discouraged.  The most reliable (and best tool to recover)
is to use the *rsync* command.  e.g.

```sh
#!/bin/bash
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1024
#SBATCH --partition=compute
#SBATCH --time=2-0

echo "Starting: $(date)"

# perform copy
rsync -avP PROJECT_DIR_NAME /home/archive/LABGROUP/pub/

echo "Finished: $(date)"
```

The manpage description of the *-avP* flags are:

```sh
        -a, --archive               archive mode; equals -rlptgoD (no -H,-A,-X)
        -r, --recursive             recurse into directories
        -l, --links                 copy symlinks as symlinks
        -p, --perms                 preserve permissions
        -t, --times                 preserve modification times
        -g, --group                 preserve group
        -o, --owner                 preserve owner (super-user only)
        -D                          same as --devices --specials
            --devices               preserve device files (super-user only)
            --specials              preserve special files
        -v, --verbose               increase verbosity
        -P                          same as --partial --progress
            --partial               keep partially transferred files
            --progress              show progress during transfer
```

Which can roughly be translated to *'make an exact copy of our project, resume if we have started before and tell us in 
detail what was done'*

## Data disposal

Data must only be disposed of in accordance with the La Trobe [Research Data Retention and Disposal 
Policy](http://www.latrobe.edu.au/policy/documents/research-data-retention-and-disposal-policy.pdf) and any other relevant 
policies.

Additionally, when a dataset is disposed of, the project directory and metadata file must remain.  A note should be added to 
the metadata file indicating the date, whom and reason it was deleted.  A full directory listing should be stored prior to
deletion:

```sh
ls -lR >> listing.txt
```

## Project best practice

To help produce repeatable science and make your life easier when it comes time to archive here are a number of suggestions
to use while working on a project.

### Each step in its own directory

Within each project there are commonly multiple processing steps used.  It's best practice to create a sub-directory for each
step in your analysis.  This helps reduce the confusion about files and makes it easier to cleanup and archive.  If you add a 
sequence number to the beginning they order correctly with *ls* 

### Clean as you go

Once you have successfully completed a step in your analysis you should remove any files resulting from earlier failed
analysis.

Better yet, move all files from this step out of the way (i.e. to a subdirectory called 'old') and repeat the steps you used to
achieve a successful result.  This makes sure you are able to repeat the analysis.  When finished you can remove the old files.

### Maintain metadata.txt

Create the metadata.txt file when you start a project and complete it as you go along.  As a minimum you should put your contact
details in there and document the source (and any approvals for) data.

### Don't link between projects

You should resist the urge to create links from one project to another as this will result in dead links if one project is
archived before another.  If two projects share the same data then you can apply to have the data stored in the genomics platform
archive and link to it there.

#### Background

When using data within your project it's best practice to use *symbolic links* to grab the source data from the genomics platform
archive instead of making a copy to your project.  There are two types of *symbolic links*: (1) relative and (2) absolute and are 
differentiated by whether you use a relative path or absolute path when creating the link.

**When should you use each**?

* **Absolute**: When linking to files or directories outside of the current project directory.  e.g. linking your nextgen sequence
  data to your projects source data. 
  * <pre>mkdir 00-raw; 
cd 00-raw; 
ln -s /data/genomics-archive/RUNID/Unaligned/sample1.fq.gz .</pre>
* **Relative**: When linking to files or directories inside your current project.  e.g. you might want to create a subset of your
  samples for testing.  This would allow you to use a file regex to select the input files to your command "*testset/\*.fq.gz*"
  * <pre>cd 00-raw; 
mkdir testset; 
cd testset; 
ln -s ../sample2.fq.gz .</pre>

### Read-only

It's a good idea to make your files read-only once they are successful to help prevent data-loss when you do a recursive delete
in the wrong place.  NOTE: this will NOT protect you from the rm -f.

```sh
# individual file(s)
chmod a-w FILENAME

# whole directory
chmod a-w -R DIRECTORYNAME
```

Another helpful hint is to create a file named '0' in directories that contain valuable files and make this read-only.  This will
cause *rm -r ...* to prompt before deleting it so if you accidentally try to delete the directory then this can help save you (by
pressing *CTRL + C* to terminate the command). 

```sh
touch 0; chmod a-w 0;
```



