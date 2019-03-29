# How to install the ORP using Docker
---

1. Build the image

```
docker build -t orp:2.2.2 -f $HOME/Oyster_River_Protocol/Dockerfile/Dockerfile
```

2. Run the Image

```
docker run -it orp:2.2.2 bash
```

3. Test the Installation

```
cd /home/training/Oyster_River_Protocol/sampledata

source activate orp

/home/training/Oyster_River_Protocol/oyster.mk \
STRAND=RF \
TPM_FILT=1 \
MEM=15 \
CPU=8 \
READ1=test.1.fq.gz \
READ2=test.2.fq.gz \
RUNOUT=test
```
