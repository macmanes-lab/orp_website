# How to install the ORP using Docker
---

1. Pull the image from DockerHub


```
docker pull macmaneslab/orp:2.3.3
```

\2. Alternatively, but probably not preferred, to build the image from scratch.

```
docker build -t orp:2.3.3 -f $HOME/Oyster_River_Protocol/Dockerfile/Dockerfile .
```

\3. Run the Image

```
docker run -it macmaneslab/orp:2.3.3 bash

#or with adding a drive, changing `/home/ubuntu/` to your mountpoint

docker run -it \
 --mount type=bind,source=/home/ubuntu/,target=/home/orp/docker \
macmaneslab/orp:2.3.3 bash

```

\4. Test the Installation

```
cd $HOME/Oyster_River_Protocol/sampledata

conda activate orp

$HOME/Oyster_River_Protocol/oyster.mk \
STRAND=RF \
TPM_FILT=1 \
MEM=5 \
CPU=4 \
READ1=test.1.fq.gz \
READ2=test.2.fq.gz \
RUNOUT=test
```
