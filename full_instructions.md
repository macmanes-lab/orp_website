# Step by step instructions
---

0. Archive Reads.
-----------------------------------
It is likely a good idea to compress your raw reads and save them elsewhere - like another computer. Computers fail, drives corrupt. Better to NOT lose your data in the process.


1. Initial Quality Check
-----------------------------------

::

  SolexaQA++ analysis file_1.fastq file_2.fastq

Plot Results using R

::

  R #this opens R on your AWS machine

  qual1 <- read.delim("file_1.fastq.quality")
  qual2 <- read.delim("file_2.fastq.quality")
  jpeg('qualplot.jpg')
  par(mfrow=c(2,1))
  boxplot(t(qual1), col='light blue', ylim=c(0,.4), frame.plot=F, outline=F, xaxt = "n", ylab='Probability of nucleotide error', xlab='Nucleotide Position', main='Read1')
  axis(1, at=c(0,10,20,30,40,50,60,70,80,90,100), labels=c(0,10,20,30,40,50,60,70,80,90,100))
  boxplot(t(qual2), col='light blue', ylim=c(0,.4), frame.plot=F, outline=F, xaxt = "n", ylab='Probability of nucleotide error', xlab='Nucleotide Position', main='Read2')
  axis(1, at=c(0,10,20,30,40,50,60,70,80,90,100), labels=c(0,10,20,30,40,50,60,70,80,90,100))
  dev.off()
  quit()
  n


2. Error Correct
-----------------------------------

Use RCorrector if you have *more* than 20 million paired-end reads.
NOTE: I've basically taken to using RCorrector for everything, given the peculiarities with running BFC.

::

  run_rcorrector.pl -k 31 -t 30 \
  -1 file_1.fastq \
  -2 file_2.fastq

Use bfc if you have *less* than 20 million paired-end reads. If you are using Illumina fastQ format 1.8 or later, :doc:`read this <bfc_pairing>` before attempting BFC correction

::

  seqtk mergepe file_1.fastq file_2.fastq > inter.fq
  bfc -s 50m -k31 -t 16 inter.fq > bfc.corr.fq
  split-paired-reads.py bfc.corr.fq
  mv bfc.corr.fq.1 bfc.corr.1.fq
  mv bfc.corr.fq.2 bfc.corr.2.fq


3. Aggressive adapter & gentle quality trimming.
-----------------------------------
One should aggressively hunt down adapter sequences and get rid of them. In contrast, gently trim low quality nucleotides. Any more will cause a significant decrease on assembly completeness, as per http://journal.frontiersin.org/article/10.3389/fgene.2014.00013/. I typically do both these steps from within Trinity (using Trimmomatic), but one could do trimming as an independent process if desired.
NOTE: Trimmomatic is a little (or maybe a lot) faster, so in general I use
::

  trimmomatic PE -threads 24 -baseout reads.TRIM.fastq \
  reads/SRR1522987_1.fastq \
  reads/SRR1522987_2.fastq \
  LEADING:3 TRAILING:3 \
  ILLUMINACLIP:barcodes.fa:2:30:10 MINLEN:25

4. Assemble
-----------------------------------
Assemble your reads using as many different assemblers as possible. I typically use Trinity, SPAdes and Shannon. I'd love to use BinPacker, but I can't usually get it to install or work. If you have stranded data, make sure to iclude the ``--SS_lib_type RF`` tag, assuming that is the right orientation (If you're using the standard TruSeq kit, it probably is). Also, you may need to adjust the ``--CPU`` and ``--max_memory`` settings. Change the name of the input reads to match your read names.

::

  Trinity --seqType fq --max_memory 100G --CPU 16 --output Rcorr_trinity --full_cleanup \
  --left skewer-trimmed-pair1.fastq \
  --right skewer-trimmed-pair2.fastq \
  --no_normalize_reads

I assemble using SPAdes with two different kmer values. k=55 and k=75.

::

  rnaspades.py --only-assembler \
  -o spades_k75 \
  --threads 24 --memory 120 -k 75 \
  -1 skewer-trimmed-pair1.fastq \
  -2 skewer-trimmed-pair2.fastq


  rnaspades.py --only-assembler \
  -o spades_k55 \
  --threads 24 --memory 120 -k 55 \
  -1 skewer-trimmed-pair1.fastq \
  -2 skewer-trimmed-pair2.fastq

Shannon assembly. To avoid running the Shanon error correction software (Quorum), I convert the fq reads to fa using `seqtk`. I wish there were a flag for this, but alas, there is none.

::

  python shannon.py -p 24 -K 75 \
  -o shannonassemb \
  --left skewer-trimmed-pair1.fa \
  --right skewer-trimmed-pair2.fa

5. OrthoFuse Merge Assemblies
----------------------------------
Each Assembler will reconstruct a slightly different set of _true_ transcript. OrthoFuse will take them both and merge them together. Orthofuse is new software I've recently written, and should be considered in alpha. It works, and we've found that it does as good a job or better than TransFuse (which we find unreliable in it's installation in running).

::

  orthofuser.mk  all \
  FASTADIR=assemblies/ \
  READ1=skewer-trimmed-pair1.fastq \
  READ2=skewer-trimmed-pair2.fastq  \
  RUNOUT=mergedassembly \
  CPU=24 \
  LINEAGE=busco_dbs/eukaryota_odb9


6. Quality Check
-----------------------------------
If you have followed the ORP AWS setup protocol, you will have the BUSCO Metazoa and Vertebrata datasets. If you need something else, you can download from here: http://busco.ezlab.org/. You should check your assembly using BUSCO. For most transcriptomes, something like 60-90% complete BUSCOs should be accepted. This might be less (even though your transcriptome is complete) if you are assembling a marine invert or some other 'weird' organism.

::

  python3 run_BUSCO.py \
  -i mergedassembly.orthomerged.fasta \
  -m transcriptome --cpu 24 -l eukaryota_odb9 -o orthofused

You should evaluate your assembly with Transrate, in addition to BUSCO. A Transrate score > .22 is generally thought to be acceptable, though higher scores are usually achievable. There is a ``good*fasta`` assembly in the output directory which you may want to use as the final assembly, for further filtering [e.g., TPM], or for something else.

::

  transrate -o assemb_name -t 16 \
  -a mergedassembly.orthomerged.fasta \
  --left skewer-trimmed-pair1.fastq \
  --right skewer-trimmed-pair2.fastq

7. Filter
-----------------------------------

Filtering is the process through which you aim to maximize the Transrate score, which assays structural integrity, while preserving the BUSCO score, which assays genic completeness. At some level this is a trade off. Some people may require a structually accurate assembly and not care so much abot completeness. Others, dare I say most, are interested in completeness - reconstructing everything possible - and care less about structure.

In general, for low coverage datasets (less than 20 million reads), filtering based on expression, using TMP=1 as a threshold performs well, with Transrate filtering often being too aggressive. With higher coverage data (more than 60 million reads) Transrate filtering may be worthwhile, as may expression filtering using a threshold of TMP=0.5. Again, these are general recommendations, you're dataset may perform differently.

To do the filtering, run BUSCO on the ``good*fasta`` file which is a product of Transrate. This assembly may be very good (or maybe not). I typically use this one if the number of BUSCOs does not decrease by more than a few percent, relative to the raw assembly output from Trinity. Use the BUSCO code from above, changing the name of the input and output. In addition to Transrate filtering (of as an alternative), it is often good to filter by gene expression. I typically filter out contigs whose expression is less than TMP=1 or TMP=0.5.


Estimate expression with Kallisto

::

  kallisto index -i kallisto.idx transfuse.fasta
  kallisto quant -t 32 -i kallisto.idx -o kallisto_orig skewer-trimmed-pair1.fastq skewer-trimmed-pair2.fastq

Estimate expression with Salmon

::

  salmon index -t transfuse.fasta -i salmon.idx --type quasi -k 31
  salmon quant -p 32 -i salmon.idx --seqBias --gcBias -l a -1 skewer-trimmed-pair1.fastq -2 skewer-trimmed-pair2.fastq -o salmon_orig

Pull down transcripts whose TPM > 1.

::

  awk '1>$5{next}1' kallisto_orig/abundance.tsv | awk '{print $1}' > kallist
  awk '1>$4{next}1' salmon_orig/quant.sf | sed  '1,10d' | awk '{print $1}' > salist
  cat kallist salist | sort -u > uniq_list

  python ~/share/filter.py transfuse.fasta uniq_list > Highexp.fasta

8. Annotate
-----------------------------------
I have taken a liking to using dammit! (http://dammit.readthedocs.org/en/latest/).

::

  mkdir ~/dammit/ && cd ~/dammit
  dammit databases --install --database-dir ~/dammit --full --busco-group metazoa
  dammit annotate Highexp.fasta --busco-group metazoa --n_threads 36 --database-dir ~/dammit/ --full


9. Report
-----------------------------------
Verify the quality of your assembly using content based metrics. Report Transrate score, BUSCO statistics, number of unique transcripts, etc. Do not report meaningless statistics such as N50
