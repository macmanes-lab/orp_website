==============================================
Oyster River Protocol For Transcriptome Assembly
==============================================

    The Oyster River Protocol for (eukaryotic) transcriptome assembly is an actively developed, evidenced based method for optimizing transcriptome assembly. The manuscript corresponding to this protocol is here: https://peerj.com/articles/5428/ 
    In brief, the protocol assembles the transcriptome using a multi-kmer multi-assembler approach, then merges those assemblies into 1 final assembly.

--------------------------------------------------
Contact Information
--------------------------------------------------

    - Gitter (preferred) |ImageLink|_
    - Email (good): Matthew.MacManes@unh.edu
    - Twitter (good):  `@MacManes <http://twitter.com/macmanes>`_
    - Phone (discouraged): 603-862-4052
    - Office (I'm hiding under my desk): 189 Rudman Hall

Some method you'd like me to benchmark? File an `issue <https://github.com/macmanes-lab/Oyster_River_Protocol/issues>`_

.. |ImageLink| image:: https://badges.gitter.im/macmanes-lab/Oyster_River_Protocol.svg
.. _ImageLink: https://gitter.im/macmanes-lab/Oyster_River_Protocol



1. Installing the software
-----------------------------------
In general, the ORP can be successfully and easily installed on Linux operating systems. OSX might work,
but I have not tried and it is unsupported. Don't try this on Windows.

Here are the instructions for installation. Getting stuff installed will be the hard part (the included makefile should do must/all of the hard work, though). Once you have things installed, should be smooth sailing!

--------------------------------------------------
 :doc:`aws_setup`
--------------------------------------------------


2. List of dependencies
------------------------
Sorry there are so many. Assembly is complex.. The makefile should take care of this.

- Rcorrector, HMMER, Trimmomatic, Trinity, SPAdes, TransABySS, MCL, Metis, OrthoFuser, BLAST, seqtk, BUSCO (make sure to install databases), TransRate (the ORP version packaged here).
- Python modules numpy, scipy, biopython, cvxopt

3. Usage
---------
After activating the `orp_v2` conda environment. this command will run the entire ORP in one shot! You can add the ```--dry-run``` flag to the end to see the individual commands that it will run, if you are curious.

**You must use the full PATH to the oyster.mk script for it to work**

::

    source activate orp_v2

    /path/to/Oyster_River_Protocol/oyster.mk main \
    MEM=150 \
    CPU=24 \
    READ1=SRR2016923_1.fastq \
    READ2=SRR2016923_2.fastq \
    RUNOUT=SRR2016923

4. Version 2.0 Changelog
---------

- The final assembly is now called `$RUNOUT.ORP.fasta`.
- Shannon has been removed, and TransABySS has been added in it's place. MANY users (and myself) have struggled with the RAM use and runtime of Shannon. TransABySS is much faster, and uses much less RAM.
- Diamond is leveraged for transcript recovery. It had been noted by some users that a few "real" transcripts were getting lost during the OrthoFuser steps.. Diamond, which is run after, recovers those.
- The use of LinuxBrew has been removed, in favor of conda. Dependencies are now managed by conda. You will need to launch the `orp_v2` conda environment before assembling.
- cd-hit-est is now run as default. 
