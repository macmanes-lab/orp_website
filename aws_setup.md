# How to install the ORP
---

If you are hoping to attempt a Trinity assembly, requirements for RAM = .5 * X million read pairs. For instance, to assemble 40 million paired-end reads using Trinity, you'll need a minimum of 20Gb of RAM.

These instructions work with a standard Ubuntu 16.04 machine available on AWS. Similar instructions should work for people on their own workstations, especially if you have `sudo` privileges, or even if you don't.


### Update Software and install things from apt-get
This is typically necessary only when starting from a fresh machine.

```
sudo apt-get update && sudo apt-get -y upgrade && sudo apt-get -y install build-essential git
```



### Install the ORP.

Packages are installed mostly via conda - at the end of the make process, you will have an `orp_v2` conda environment, that will contain everything you need for assembly. Make sure to type `source ~/.profile` and the end of the `make` process/

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
sed -i  's_ubuntu_$(whoami)_g' $HOME/Oyster_River_Protocol/software/config.ini
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

You must activate the `orp_v2` conda environment, that `make` made for you.

**You must use the full PATH to the oyster.mk script for it to work**

```
cd $HOME/Oyster_River_Protocol/sampledata

source activate orp_v2

#note use of full PATH (your PATH to oyster.mk might be different)

$HOME/Oyster_River_Protocol/oyster.mk main \
STRAND=RF \
MEM=15 \
CPU=8 \
READ1=test.1.fq.gz \
READ2=test.2.fq.gz \
RUNOUT=test
```
