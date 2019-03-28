# How to install the ORP
---

If you are hoping to attempt a Trinity assembly, requirements for RAM = .5 * X million read pairs. For instance, to assemble 40 million paired-end reads using Trinity, you'll need a minimum of 20Gb of RAM.

These instructions work with a standard Ubuntu 16.04 or 18.04 machine available on AWS. Similar instructions should work for people on their own workstations, especially if you have `sudo` privileges, or even if you don't.


### Update Software and install things from apt-get
This is typically necessary only when starting from a fresh machine.

```
sudo apt-get update && sudo apt-get -y upgrade && sudo apt-get -y install build-essential git python python-pip libxml2-dev libz-dev
```



### Install the ORP.

Packages are installed mostly via conda - at the end of the make process, you will have an `orp` conda environment, that will contain everything you need for assembly. Make sure to type `source ~/.profile` and the end of the `make` process/

```
git clone https://github.com/macmanes-lab/Oyster_River_Protocol.git
cd Oyster_River_Protocol
make
source ~/.profile


### Make sure to add the items to your profile file, as needed.
**Make sure to ```source``` the profile file after, to make sure everything is loaded.**
```

### Edit `config.ini`.
You should just have to change the user path info (the stuff before `/Oyster_River_Protocol/...`). A simple way to do this is via sed.

```
sed -i  "s_ubuntu_$(whoami)_g" $HOME/Oyster_River_Protocol/software/config.ini
```

You may want to install additional BUSCO databases - the Euk. database is installed and used by default.

```
###Download databases

cd $HOME/Oyster_River_Protocol/busco_dbs

# Eukaryota
wget http://busco.ezlab.org/v2/datasets/fungi_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/metazoa_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/nematoda_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/arthropoda_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/insecta_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/vertebrata_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/tetrapoda_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/aves_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/mammalia_odb9.tar.gz

tar -zxf ..
cd
```

### Test the Installation

This is a very small data set that should assemble ~30 transcripts. It will finished in a few minutes or less using desktop-sized computer. The BUSCO numbers you get at the end are bad, for obvious reasons. If this finishes without error, you're good to move on to a 'real' assembly!! Good luck, and ping me on Gitter if issues!

You must activate the `orp` conda environment, that `make` made for you.

**You must use the full PATH to the oyster.mk script for it to work**

```
cd $HOME/Oyster_River_Protocol/sampledata

source activate orp

#note use of full PATH (your PATH to oyster.mk might be different)

$HOME/Oyster_River_Protocol/oyster.mk main \
STRAND=RF \
MEM=15 \
CPU=8 \
READ1=test.1.fq.gz \
READ2=test.2.fq.gz \
RUNOUT=test
```

At the end of the successful run, you should see some text that looks like this. Your numbers will be different, but should similar. Assembly is not deterministic.

```
7|  #
 6| ##
 5| ##
 4| ##
 3| ###
 2| ###       #
 1| ###  ##   #
   -----------

------------------------
|       Summary        |
------------------------
|   observations: 20   |
| min value: -1.000000 |
|   mean : -0.987400   |
| max value: -0.935000 |
------------------------


*****  See the following link for interpretation *****
*****  https://oyster-river-protocol.readthedocs.io/en/latest/strandexamine.html *****



*****  QUALITY REPORT FOR: test using the ORP version 2.1.0 ****
*****  THE ASSEMBLY CAN BE FOUND HERE: /root/ORP/sampledata/assemblies/test.ORP.fasta ****

*****  BUSCO SCORE ~~~~~>               C:0.0%[S:0.0%,D:0.0%],F:0.3%,M:99.7%,n:303
*****  TRANSRATE SCORE ~~~~~>           0.37518
*****  TRANSRATE OPTIMAL SCORE ~~~~~>   0.56393
*****  UNIQUE GENES ORP ~~~~~>          39
*****  UNIQUE GENES TRINITY ~~~~~>      31
*****  UNIQUE GENES SPADES55 ~~~~~>     22
*****  UNIQUE GENES SPADES75 ~~~~~>     23
*****  UNIQUE GENES TRANSABYSS ~~~~~>   35
```
