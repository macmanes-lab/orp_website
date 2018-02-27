# How to set up a clean AWS instance for the ORP
---

If you are hoping to attempt a Trinity assembly, requirements for RAM = .5 * X million read pairs. For instance, to assemble 40 million paired-end reads using Trinity, you'll need a minimum of 20Gb of RAM.

These instructions work with a standard Ubuntu 16.04 machine available on AWS. Similar instructions should work for people on their own workstations, especially if you have `sudo` privileges.


### Update Software and install things from apt-get

```
sudo apt-get update && sudo apt-get -y upgrade && sudo apt-get -y install ruby build-essential mcl python python-pip default-jre
```


### Install LinuxBrew


```
sudo mkdir /home/linuxbrew
sudo chown $USER:$USER /home/linuxbrew
git clone https://github.com/Linuxbrew/brew.git /home/linuxbrew/.linuxbrew
echo 'export PATH="/home/linuxbrew/.linuxbrew/bin:$PATH"' >> ~/.profile
echo 'export MANPATH="/home/linuxbrew/.linuxbrew/share/man:$MANPATH"' >> ~/.profile
echo 'export INFOPATH="/home/linuxbrew/.linuxbrew/share/info:$INFOPATH"' >> ~/.profile
source ~/.profile
brew tap brewsci/science
brew tap brewsci/bio
brew update
brew install gcc python metis parallel
```

### Install Python Modules

```
pip install cvxopt numpy biopython scipy
```

### Install the ORP

```
git clone https://github.com/macmanes-lab/Oyster_River_Protocol.git
cd Oyster_River_Protocol
make -j4


### Make sure to add the items to your profile file, as needed.
Make sure to ```source``` the profile file after, to make sure everything is loaded.  
```

### Set up BUSCO

```
### Download databases

mkdir $HOME/busco_dbs && cd $HOME/busco_dbs

# Eukaryota
wget http://busco.ezlab.org/v2/datasets/eukaryota_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/fungi_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/metazoa_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/nematoda_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/arthropoda_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/insecta_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/vertebrata_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/tetrapoda_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/aves_odb9.tar.gz
wget http://busco.ezlab.org/v2/datasets/mammalia_odb9.tar.gz

tar -zxf eukaryota_odb9.tar.gz

### Move and edit config file
mv $HOME/Oyster_River_Protocol/software/busco/config/config.ini.default $HOME/Oyster_River_Protocol/software/busco/config/config.ini
nano $HOME/Oyster_River_Protocol/software/busco/config/config.ini

### add this line under the `[busco] line`

lineage_path = /home/ubuntu/busco_dbs/eukaryota_odb9

### obviously, if you're using another database, that name will change.

### you'll need to change the PATH entries for, at least,
BLAST and HMMER... (see https://gitlab.com/ezlab/busco/issues/46).
Try ```which hmmscan``` and ```which blastp``` to find locations.
```

### Test the Installation

This is a very small data set that should assemble ~30 transcripts. It will finished in a few minutes or less using desktop-sized computer. The BUSCO numbers you get at the end are bad, for obvious reasons. If this finishes without error, you're good to move on to a 'real' assembly!! Good luck, and ping me on Gitter if issues!

```
cd $HOME/Oyster_River_Protocol/sampledata

../oyster.mk main \
MEM=15 \
CPU=8 \
READ1=test.1.fq.gz \
READ2=test.2.fq.gz \
RUNOUT=test
```
