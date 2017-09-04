# Cracking hashes using AWS GPU instances

## The Concept

Use AWS to run a timeboxed hashcat instance with many GPU's to test the strength of password/key hashes.

The intended architecture is:
* use **slack** to trigger a **Lambda** function (via standard bot-http interface)
* the **Lambda** function would ```instance.start()``` an **ec2** multi GPU instance 
* which on start up would configure **hashcat** using configurations and dictionaries off an **S3** bucket
* let **hashcat** run for a set period
* return the result via a **http webhook** to **slack**.

![architecture](https://github.com/raoul361/AWS-hash-cracking/blob/master/AWS%20GPU%20hashing.png)

## EC2 Instance

I set up an ec2 instance as a proof of concept to run hashcat. AWS provides an instance with 4, 8 or 16 GPU's; I chose the **g3.4xlarge**. For the OS, I'm using the **NVIDIA Linux AMI** which comes with the NVIDIA drivers ready to go. This is important, as otherwise you need to modify kernel drivers which requires rebooting the system. I just want something easy.

### EC2 user-data

I'm using **user-data** as part of the ec2 config to set a couple of parameters, install hashcat and run the benchmarks. In the real version, this would run a script to retrieve hashcat configs and dictionaries from **S3** as well as the user supplied **hash** to be cracked.

```
#!/bin/sh
sudo nvidia-smi -pm 1
sudo nvidia-smi -acp 0
sudo nvidia-smi —auto-boost-default=1
git clone https://github.com/hashcat/hashcat.git
cd hashcat/
git submodule update --init
make
sudo make install
/usr/local/bin/hashcat -b --force >> /var/log/cloud-init-output.log
halt
```

## Results

Disappointing! While this all works in theory, the GPU's afforded through AWS are no where near the power of a current generation gaming GPU. It was much quicker to run this on my desktop than in the cloud - even across 4 GPU's. It should also be noted that the configuration and availability of the high end compute instances varies significantly.

So I left it here. I may pick it up again if/when I find a way to do this on AWS that beats a mid-range GPU.


### Hash rates on AWS

```
OpenCL Platform #1: NVIDIA Corporation
======================================
* Device #1: GRID K520, 1023/4095 MB allocatable, 8MCU
* Device #2: GRID K520, 1023/4095 MB allocatable, 8MCU
* Device #3: GRID K520, 1023/4095 MB allocatable, 8MCU
* Device #4: GRID K520, 1023/4095 MB allocatable, 8MCU

Hashtype: MD5

Speed.Dev.#1.....:  2357.8 MH/s (56.86ms)
Speed.Dev.#2.....:  2359.7 MH/s (56.85ms)
Speed.Dev.#3.....:  2357.9 MH/s (56.89ms)
Speed.Dev.#4.....:  2353.3 MH/s (56.99ms)
Speed.Dev.#*.....:  9428.8 MH/s

Hashtype: SHA1

Speed.Dev.#1.....:   629.8 MH/s (53.24ms)
Speed.Dev.#2.....:   630.0 MH/s (53.22ms)
Speed.Dev.#3.....:   632.0 MH/s (53.06ms)
Speed.Dev.#4.....:   629.2 MH/s (53.29ms)
Speed.Dev.#*.....:  2521.0 MH/s
```

### Hash rates on GTX 1060

```
OpenCL Platform #1: NVIDIA Corporation
======================================
* Device #1: GeForce GTX 1060 6GB, 1536/6144 MB allocatable, 10MCU

Hashtype: MD5

Speed.Dev.#1.....: 11437.2 MH/s (58.60ms)

Hashtype: SHA1

Speed.Dev.#1.....:  4310.1 MH/s (77.76ms)
```
