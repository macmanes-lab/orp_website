# How to set up a clean AWS instance for the ORP
---

If you are hoping to attempt a Trinity assembly, requirements for RAM = .5 * X million read pairs. For instance, to assemble 40 million paired-end reads using Trinity, you'll need a minimum of 20Gb of RAM.

These instructions work with a standard Ubuntu 16.04 machine available on AWS. Similar instructions should work for people on their own workstations, especially if you have `sudo` privileges.


### Update Software and install things from apt-get

```
sudo apt-get update && sudo apt-get -y upgrade && sudo apt-get -y install ruby build-essential mcl python python-pip

```


### Install LinuxBrew


```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install)"
echo 'export PATH="/home/ubuntu/.linuxbrew/bin:$PATH"' >> ~/.profile
source ~/.profile
brew tap homebrew/science
brew update
brew install gcc python python3 metis parallel
```

### Install Python Modules

```
sudo pip install cvxopt numpy biopython scipy
```

### Install the ORP

```
git clone https://github.com/macmanes-lab/Oyster_River_Protocol.git
cd Oyster_River_Protocol
make


### Make sure to add the items to your profile file, as needed.
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
mv /home/ubuntu/Oyster_River_Protocol/software/busco/config/config.ini.default /home/ubuntu/Oyster_River_Protocol/software/busco/config/config.ini
nano /home/ubuntu/Oyster_River_Protocol/software/busco/config/config.ini

### add this line under the `[busco] line`

lineage_path = $HOME/busco_dbs/eukaryota_odb9

### obviously, if you're using another database, that name will change.
```

### Test the Installation

```
cd sampledata

../oyster.mk main \
MEM=15 \
CPU=8 \
READ1=test.1.fq.gz \
READ2=test.2.fq.gz \
RUNOUT=test
```






cd
curl -LO https://github.com/TransDecoder/TransDecoder/archive/2.0.1.tar.gz
tar -xvzf 2.0.1.tar.gz
cd TransDecoder-2.0.1
make -j6
export PATH=$PATH:$HOME/TransDecoder-2.0.1
```

### Install LAST

```
cd
curl -LO http://last.cbrc.jp/last-658.zip
unzip last-658.zip
cd last-658
make
export PATH=$PATH:$HOME/last-658/src
```

### Install dammit!

```
sudo gem install crb-blast
sudo pip install -U setuptools
sudo pip install numpy --upgrade
sudo pip install matplotlib --upgrade
sudo pip install dammit
```


### Install Perl Module
```
sudo cpan URI::Escape
```

### Install seqtk

```
cd $HOME
git clone https://github.com/lh3/seqtk.git
cd seqtk
make -j4
PATH=$PATH:$(pwd)
```

### Install bwa

```
cd $HOME
git clone https://github.com/lh3/bwa.git
cd bwa
make -j4
PATH=$PATH:$(pwd)
```

### Install khmer

```
sudo easy_install -U setuptools
sudo pip install khmer
```

### Install Kallisto

```
cd
git clone https://github.com/pachterlab/kallisto.git
cd kallisto
mkdir build
cd build
cmake ..
make
sudo make install
```


### Install Salmon

```
cd
curl -LO https://github.com/COMBINE-lab/salmon/archive/v0.5.1.tar.gz
tar -zxf v0.5.1.tar.gz
cd salmon-0.5.1/
mkdir build
cd build
cmake -DCMAKE_C_COMPILER=$(which gcc-4.9) -DCMAKE_CXX_COMPILER=$(which g++-4.9) ..
make -j6
sudo make all install
export LD_LIBRARY_PATH=/home/ubuntu/salmon-0.5.1/lib
```

### Install Transrate

```
cd
curl -LO https://bintray.com/artifact/download/blahah/generic/transrate-1.0.1-linux-x86_64.tar.gz
tar -zxf transrate-1.0.1-linux-x86_64.tar.gz
PATH=$PATH:/home/ubuntu/transrate-1.0.1-linux-x86_64
```

### Install BUSCO

```
cd
curl -LO http://busco.ezlab.org/files/BUSCO_v1.1b1.tar.gz
tar -zxf BUSCO_v1.1b1.tar.gz
cd BUSCO_v1.1b1
chmod +x BUSCO_v1.1b1.py
PATH=$PATH:$(pwd)
curl -LO http://busco.ezlab.org/files/metazoa_buscos.tar.gz
curl -LO http://busco.ezlab.org/files/vertebrata_buscos.tar.gz
tar -zxf vertebrata_buscos.tar.gz
tar -zxf metazoa_buscos.tar.gz
```

### Install BLAST


```
cd
curl -LO ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/ncbi-blast-2.3.0+-x64-linux.tar.gz
tar -zxf ncbi-blast-2.3.0+-x64-linux.tar.gz
PATH=$PATH:/home/ubuntu/ncbi-blast-2.3.0+/bin
```

### Install Trinity

```
cd
git clone https://github.com/trinityrnaseq/trinityrnaseq.git
cd trinityrnaseq
make -j6
PATH=$PATH:$(pwd)
```

### Install bfc

```
cd $HOME
git clone https://github.com/lh3/bfc.git
cd bfc
make
PATH=$PATH:$(pwd)
```

### Install RCorrector

```
cd
git clone https://github.com/mourisl/Rcorrector.git
cd Rcorrector
make
PATH=$PATH:$(pwd)
```

### Add all these things to the permanent path

```
echo PATH=$PATH >> ~/.profile
echo export LD_LIBRARY_PATH=/home/ubuntu/salmon-0.5.1/lib >> ~/.profile
source ~/.profile
```
