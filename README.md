![Image of IBM Systems University 2019](https://github.com/guikarai/PE-LinuxONE/blob/master/images/stgu.png)


# Pervasive Encryption on Linux on IBM Z or LinuxONE Applied - Hands-on LABs
Coming soon in IBM Systems Technical University
* Monday 15:30-16:30 Diamant-First Floor

# Purpose of this Hands-on LAB
Pervasive Encryption has consumable capabilities available on Linux on IBM Z in order to protect both data-at-rest and data-in-motion. 
In this hands-on lab, you will be guided in the following activities: 
* environment preparation
* crypto stack configuration
* data at rest protection with a with dm-crypt volume encryption implementation
* data in motion protection with OpenSSL tuning

At the end of the hands-on lab, attendees will be able to quick start with pervasive encryption on Linux on IBM Z.

**Many thanks to:** Sylvain Carta, Guillaume Lasmayous, Eric Phan

# About Pervasive Encryption on LinuxONE
Pervasive encryption is a data-centric approach to information security that entails protecting data entering and exiting the z14 platform. It involves encrypting data in-flight and at-rest to meet complex compliance mandates and reduce the risks and financial losses of a data breach. It is a paradigm shift from selective encryption (where only the data that is required to achieve compliance is encrypted) to pervasive encryption. Pervasive encryption with z14 is enabled through tight platform integration that includes Linux on IBM Z or LinuxONE following features:
* Integrated cryptographic hardware: Central Processor Assist for Cryptographic Function (CPACF) is a co-processor on every processor unit that accelerates encryption. Crypto Express features can be used as hardware security modules (HSMs).
* Data set and file encryption: You can protect Linux file systems that is transparent to applications and databases.
* Network encryption: You can protect network data traffic by using standards-based encryption from endpoint to endpoint.

## LinuxONE Crypto Stack
Pervasive Encryption benefits of the full Power of Linux Ecosystem plus z14 Capabilities
* LUKS dm-crypt – Transparent file & volume encryption using industry unique CPACF protected-keys
* Network Security – Enterprise scale encryption and handshakes using z14 CPACF and SIMD (openSSL, IPSec...)

The IBM Z and LinuxONE systems provide cryptographic functions that, from an application program perspective, can be grouped as follows:
* Synchronous cryptographic functions, provided by the CP Assist for Cryptographic Function (CPACF) or the Crypto Express features when defined as an accelerator.
* Asynchronous cryptographic functions, provided by the Crypto Express features.

The IBM Z and LinuxONE systems provide also rich cryptographic functions available via a complete crypto stack made of a set of key crypto APIs.
![Image of the Crypto Stack](https://github.com/guikarai/PE-LinuxONE/blob/master/images/crypto-stack.png)

**Note:** Locate openSSL and dm-crypt. For the following, we will work on how set-up a Linux environment in order to benefit of Pervasive Encryption benefits.
  
## Enabling Linux to use the Hardware
### 1. CPACF Enablement verification
A Linux on IBM Z user can easily check whether the Crypto Enablement feature is installed and which algorithms are supported in hardware. Hardware-acceleration for DES, TDES, AES, and GHASH requires CPACF. Issue the command shown below to discover whether the CPACF feature is enabled
on your hardware.
```
root@crypt04:~# cat /proc/cpuinfo
vendor_id       : IBM/S390
# processors    : 2
bogomips per cpu: 21881.00
max thread id   : 0
features	: esan3 zarch stfle msa ldisp eimm dfp edat etf3eh highgprs te vx vxd vxe gs sie 
facilities      : 0 1 2 3 4 6 7 8 9 10 12 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 30 31 32 33 34 35 36 37 38 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 57 58 59 60 73 74 75 76 77 80 81 82 128 129 130 131 133 134 135 146 147 168
cache0          : level=1 type=Data scope=Private size=128K line_size=256 associativity=8
cache1          : level=1 type=Instruction scope=Private size=128K line_size=256 associativity=8
cache2          : level=2 type=Data scope=Private size=4096K line_size=256 associativity=8
cache3          : level=2 type=Instruction scope=Private size=2048K line_size=256 associativity=8
cache4          : level=3 type=Unified scope=Shared size=131072K line_size=256 associativity=32
cache5          : level=4 type=Unified scope=Shared size=688128K line_size=256 associativity=42
processor 0: version = FF,  identification = 233EF7,  machine = 3906
processor 1: version = FF,  identification = 233EF7,  machine = 3906

cpu number      : 0
cpu MHz dynamic : 5208
cpu MHz static  : 5208

cpu number      : 1
cpu MHz dynamic : 5208
cpu MHz static  : 5208
```

**Note**: msa on line 5, indicates that the CPACF instruction is properly supported and detected.

**Note 2**: vx on line 5, indicates that SIMD and vector instructions are properly supported and detected.

### 2. Installing libica
To make use of the libica hardware support for cryptographic functions, you must install the libica package. Obtain the current libica version from your distribution provider for automated installation by issuing the following command:
```
root@crypt04:~# sudo apt-get install libica-utils
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  libica3
The following NEW packages will be installed
  libica-utils libica3
0 to upgrade, 2 to newly install, 0 to remove and 0 not to upgrade.
Need to get 76.1 kB of archives.
After this operation, 250 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://fr.ports.ubuntu.com/ubuntu-ports bionic/universe s390x libica3 s390x 3.2.1-0ubuntu1 [60.5 kB]
Get:2 http://fr.ports.ubuntu.com/ubuntu-ports bionic/universe s390x libica-utils s390x 3.2.1-0ubuntu1 [15.5 kB]
Fetched 76.1 kB in 0s (212 kB/s)   
Selecting previously unselected package libica3:s390x.
(Reading database ... 71930 files and directories currently installed.)
Preparing to unpack .../libica3_3.2.1-0ubuntu1_s390x.deb ...
Unpacking libica3:s390x (3.2.1-0ubuntu1) ...
Selecting previously unselected package libica-utils.
Preparing to unpack .../libica-utils_3.2.1-0ubuntu1_s390x.deb ...
Unpacking libica-utils (3.2.1-0ubuntu1) ...
Setting up libica3:s390x (3.2.1-0ubuntu1) ...
Processing triggers for libc-bin (2.27-3ubuntu1) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Setting up libica-utils (3.2.1-0ubuntu1) ...
```

Press "y" when prompted.

After the libica utility is installed, use the **icainfo** command to check on the CPACF feature code enablement. 
The icainfo command displays which CPACF functions are supported by the implementation inside the libica library. Issue the following command to show which cryptographic algorithms will be hardware-accelerated by the libica driver, and which one will remain software-only implementations.

```
root@crypt04:~# icainfo
      Cryptographic algorithm support      
-------------------------------------------
 function      |  hardware  |  software  
---------------+------------+------------
         SHA-1 |    yes     |     yes
       SHA-224 |    yes     |     yes
       SHA-256 |    yes     |     yes
       SHA-384 |    yes     |     yes
       SHA-512 |    yes     |     yes
      SHA3-224 |    yes     |      no
      SHA3-256 |    yes     |      no
      SHA3-384 |    yes     |      no
      SHA3-512 |    yes     |      no
     SHAKE-128 |    yes     |      no
     SHAKE-256 |    yes     |      no
         GHASH |    yes     |      no
         P_RNG |    yes     |     yes
  DRBG-SHA-512 |    yes     |     yes
        RSA ME |     no     |     yes
       RSA CRT |     no     |     yes
       DES ECB |    yes     |     yes
       DES CBC |    yes     |     yes
       DES OFB |    yes     |      no
       DES CFB |    yes     |      no
       DES CTR |    yes     |      no
      DES CMAC |    yes     |      no
      3DES ECB |    yes     |     yes
      3DES CBC |    yes     |     yes
      3DES OFB |    yes     |      no
      3DES CFB |    yes     |      no
      3DES CTR |    yes     |      no
     3DES CMAC |    yes     |      no
       AES ECB |    yes     |     yes
       AES CBC |    yes     |     yes
       AES OFB |    yes     |      no
       AES CFB |    yes     |      no
       AES CTR |    yes     |      no
      AES CMAC |    yes     |      no
       AES XTS |    yes     |      no
       AES GCM |    yes     |      no
-------------------------------------------
No built-in FIPS support.
```

From the **cpuinfo** output, you can find the features that are enabled in the central processors. 
If the features list has msa listed, it means that CPACF is enabled. Most of the distributions include a generic kernel image for the specific platform. 
These device drivers for the generic kernel image are included as loadable kernel modules because statically compiling many drivers into one kernel causes the kernel image to be much larger. This kernel might be too large to boot on computers with limited memory.

### 3. Loading crypto modules
Let's use the **modprobe** command to load the device driver modules. Initially the Linux system is not yet configured to use  the crypto device driver modules, so you must load them manually. The cryptographic device drivers consists of multiple,
separate modules.
```
root@crypt04:~# modprobe aes_s390
root@crypt04:~# modprobe des_s390
root@crypt04:~# modprobe sha1_s390
root@crypt04:~# modprobe sha256_s390
root@crypt04:~# modprobe sha512_s390
root@crypt04:~# modprobe rng     
root@crypt04:~# modprobe prng
root@crypt04:~# modprobe hmac
root@crypt04:~# modprobe gcm
root@crypt04:~# modprobe xts
```

### 4. Pervasive Encryption readiness assessment
Validate that all the crypto modules are properly loaded. Please issue the following command:
```
root@crypt04:~# lsmod | grep s390
s390_trng              16384  0
ghash_s390             16384  0
aes_s390               24576  0
des_s390               20480  0
des_generic            28672  1 des_s390
sha512_s390            16384  0
sha256_s390            16384  0
sha1_s390              16384  0
sha_common             16384  3 sha512_s390,sha256_s390,sha1_s390
crc32_vx_s390          16384  3
```

Validate that the libica crypto API is working properly. Please issue the following command:
```
root@crypt04:~# icastats
 function     |           hardware       |            software
--------------+--------------------------+-------------------------
              |      ENC    CRYPT   DEC  |      ENC    CRYPT   DEC 
--------------+--------------------------+-------------------------
        SHA-1 |               0          |                0
      SHA-224 |               0          |                0
      SHA-256 |               0          |                0
      SHA-384 |               0          |                0
      SHA-512 |               0          |                0
     SHA3-224 |               0          |                0
     SHA3-256 |               0          |                0
     SHA3-384 |               0          |                0
     SHA3-512 |               0          |                0
    SHAKE-128 |               0          |                0
    SHAKE-256 |               0          |                0
        GHASH |               0          |                0
        P_RNG |               0          |                0
 DRBG-SHA-512 |             168          |                0
       RSA-ME |               0          |                0
      RSA-CRT |               0          |                0
      DES ECB |         0              0 |         0             0
      DES CBC |         0              0 |         0             0
      DES OFB |         0              0 |         0             0
      DES CFB |         0              0 |         0             0
      DES CTR |         0              0 |         0             0
     DES CMAC |         0              0 |         0             0
     3DES ECB |         0              0 |         0             0
     3DES CBC |         0              0 |         0             0
     3DES OFB |         0              0 |         0             0
     3DES CFB |         0              0 |         0             0
     3DES CTR |         0              0 |         0             0
    3DES CMAC |         0              0 |         0             0
      AES ECB |         0              0 |         0             0
      AES CBC |         0              0 |         0             0
      AES OFB |         0              0 |         0             0
      AES CFB |         0              0 |         0             0
      AES CTR |         0              0 |         0             0
     AES CMAC |         0              0 |         0             0
      AES XTS |         0              0 |         0             0
      AES GCM |         0              0 |         0             0
```
**Note:** As you can see, there is already some crypto offload regarding DRBG-SHA-512. That is a good start ;D

**Note 2:** For your information, DRBG stands for Deterministic Random Bits Generator. It is an algorithm for generating a sequence of numbers whose properties approximate the properties of sequences of random numbers.

Your hands-on LAB environment is now properly setup!

## Pervasive Encryption - Enabling OpenSSL and OpenSSH to use the hardware acceleration support

This chapter describes how to use the cryptographic functions of the LinuxONE server to encrypt data in flight. This technique means that the data is encrypted and decrypted before and after it is transmitted. We will use OpenSSL, SCP and SFTP to demonstrate the encryption of data in flight. 
This chapter also shows how to customize the product to use the LinuxONE hardware encryption features. This chapter includes the following sections:
- Preparing to use openSSL
- Configuring OpenSSL
- Testing Hardware Crypto functions with OpenSSL
- Testing Hardware Crypto functions with SCP

### 1. Preparing to use OpenSSL
In the Linux system you use, OpenSSL is already installed, and the system is already enabled to use the cryptographic hardware of the LinuxONE server. We also loaded the cryptographic device drivers and the libica to use the crypto hardware. For the following steps, the following packages are required for encryption:
- openssl
- openssl-libs
- openssl-ibmca

During the installation of Ubuntu, the package openssl-ibmca was not automatically installed and needs to be installed manually. Please issue the following command:
```
root@crypt04:~# sudo apt-get install openssl-ibmca
```
Now all needed packages are successfully installed. At this moment only the default engine of OpenSSL is available. To check it, please issue the following command:
```
root@crypt04:~# openssl engine -c
(dynamic) Dynamic engine loading support
```
### 2. Configuring OpenSSL
To use the ibmca engine and to benefit from the Cryptographic hardware support, the configuration file of OpenSSL needs to be adjusted. To customize OpenSSL configuration to enable dynamic engine loading for ibmca, complete the following steps.
#### 2.1. Locate the OpenSSL configuration file, which in our Ubuntu 16.04.3 LTS distribution is in this subdirectory: 
```
root@crypt06:~# ls /usr/share/doc/openssl-ibmca/examples
```

#### 2.2. Make a backup copy of the configuration file
Locate the main configuration file of openssl. Its name is openssl.cnf. Please issue the following command:
```
root@crypt04:~# ls -la /etc/ssl/openssl.cnf
-rw-r--r-- 1 root root 10771 Apr 25  2018 /etc/ssl/openssl.cnf
```
Make a backup copy of the configuration file. We will modify it later, so just in case, please issue the following command:
```
root@crypt04:~# cp -p /etc/ssl/openssl.cnf /etc/ssl/openssl.cnf.backup
```
Check one more time that everything is alright and secured. Please issue the following command:
```
root@crypt04:~# ls -al /etc/ssl/openssl.cnf*
-rw-r--r-- 1 root root 10771 Apr 25  2018 /etc/ssl/openssl.cnf
-rw-r--r-- 1 root root 10771 Apr 25  2018 /etc/ssl/openssl.cnf.backup
```

#### 2.3. Append the ibmca-related configuration lines to the OpenSSL configuration file
```
root@crypt04:~# tee -a /etc/ssl/openssl.cnf < /usr/share/doc/openssl-ibmca/examples/openssl.cnf.sample
#
# OpenSSL example configuration file. This file will load the IBMCA engine
# for all operations that the IBMCA engine implements for all apps that
# have OpenSSL config support compiled into them.
#
# Adding OpenSSL config support is as simple as adding the following line to
# the app:
#
# #define OPENSSL_LOAD_CONF	1
#
openssl_conf = openssl_def

[openssl_def]
engines = engine_section


[engine_section]
ibmca = ibmca_section


[ibmca_section]

# The openssl engine path for ibmca.so.
# Set the dynamic_path to where the ibmca.so engine
# resides on the system.
dynamic_path = /usr/lib/s390x-linux-gnu/engines-1.1/ibmca.so

engine_id = ibmca
init = 1

#
# The following ibmca algorithms will be enabled by these parameters
# to the default_algorithms line. Any combination of these is valid,
# with "ALL" denoting the same as all of them in a comma separated
# list.
#
# RSA
# - RSA encrypt, decrypt, sign and verify, key lengths 512-4096
#
# DH
# - DH key exchange
#
# DSA
# - DSA sign and verify
#
# RAND
# - Hardware random number generation
#
# CIPHERS
# - DES-ECB, DES-CBC, DES-CFB, DES-OFB,
#   DES-EDE3, DES-EDE3-CBC, DES-EDE3-CFB, DES-EDE3-OFB,
#   AES-128-ECB, AES-128-CBC, AES-128-CFB, AES-128-OFB, id-aes128-GCM,
#   AES-192-ECB, AES-192-CBC, AES-192-CFB, AES-192-OFB, id-aes192-GCM,
#   AES-256-ECB, AES-256-CBC, AES-256-CFB, AES-256-OFB, id-aes256-GCM ciphers
#
# DIGESTS
# - SHA1, SHA256, SHA512 digests
#
default_algorithms = ALL
#default_algorithms = RAND,RSA,DH,DSA,CIPHERS,DIGESTS
```

#### 2.4. Append the ibmca-related configuration lines to the OpenSSL configuration file
The reference to the ibmca section in the OpenSSL configuration file needs to be inserted. 

Therefore, insert the following line as show below at the line 10.

"openssl_conf = openssl_def"

```
#
# OpenSSL example configuration file.
# This is mostly being used for generation of certificate requests.
#

# This definition stops the following lines choking if HOME isn't
# defined.
HOME                    = .
RANDFILE                = $ENV::HOME/.rnd
openssl_conf = openssl_def #<============================================== line inserted
# Extra OBJECT IDENTIFIER info:
#oid_file               = $ENV::HOME/.oid
oid_section             = new_oids

# To use this configuration file with the "-extfile" option of the
# "openssl x509" utility, name here the section containing the
# X.509v3 extensions to use:
# extensions            =
# (Alternatively, use a configuration file that has only
# X.509v3 extensions in its main [= default] section.)
[ --- --- -- --- TRUNCATED -- --- -- -- --]
```

### 3. Checking Hardware Crypto functions
Now that the customization of OpenSSL in done, test whether you can use the LinuxONE hardware cryptographic functions. First, let's check whether the dynamic engine loading support is enabled by default and the engine ibmca is available and used in the installation.
```
root@crypt04:~# openssl engine -c
(dynamic) Dynamic engine loading support
(ibmca) Ibmca hardware engine support
 [RAND, DES-ECB, DES-CBC, DES-OFB, DES-CFB, DES-EDE3, DES-EDE3-CBC, DES-EDE3-OFB, DES-EDE3-CFB, AES-128-ECB, AES-192-ECB, AES-256-ECB, AES-128-CBC, AES-192-CBC, AES-256-CBC, AES-128-OFB, AES-192-OFB, AES-256-OFB, AES-128-CFB, AES-192-CFB, AES-256-CFB, id-aes128-GCM, id-aes192-GCM, id-aes256-GCM, SHA1, SHA256, SHA512]
```

#### 3.1. Testing Pervasive encryption with OpenSSL
Now openSSL is properly configured. All crypto operation passing through openSSL will be hardware accelerated if possible. Let's test it in live. Firt of all, let's have a look of the hardware crypto offload status. Please issue the following command:
```
root@crypt04:~# icastats
 function     |           hardware       |            software
--------------+--------------------------+-------------------------
              |      ENC    CRYPT   DEC  |      ENC    CRYPT   DEC 
--------------+--------------------------+-------------------------
        SHA-1 |               0          |                0
      SHA-224 |               0          |                0
      SHA-256 |               0          |                0
      SHA-384 |               0          |                0
      SHA-512 |               0          |                0
     SHA3-224 |               0          |                0
     SHA3-256 |               0          |                0
     SHA3-384 |               0          |                0
     SHA3-512 |               0          |                0
    SHAKE-128 |               0          |                0
    SHAKE-256 |               0          |                0
        GHASH |               0          |                0
        P_RNG |               0          |                0
 DRBG-SHA-512 |             336          |                0
       RSA-ME |               0          |                0
      RSA-CRT |               0          |                0
      DES ECB |         0              0 |         0             0
      DES CBC |         0              0 |         0             0
      DES OFB |         0              0 |         0             0
      DES CFB |         0              0 |         0             0
      DES CTR |         0              0 |         0             0
     DES CMAC |         0              0 |         0             0
     3DES ECB |         0              0 |         0             0
     3DES CBC |         0              0 |         0             0
     3DES OFB |         0              0 |         0             0
     3DES CFB |         0              0 |         0             0
     3DES CTR |         0              0 |         0             0
    3DES CMAC |         0              0 |         0             0
      AES ECB |         0              0 |         0             0
      AES CBC |         0              0 |         0             0
      AES OFB |         0              0 |         0             0
      AES CFB |         0              0 |         0             0
      AES CTR |         0              0 |         0             0
     AES CMAC |         0              0 |         0             0
      AES XTS |         0              0 |         0             0
      AES GCM |         0              0 |         0             0
```
As you can see, the libica API already detect some crypto offload to the hardware.

Let's go deeper with some openSSL tests. Please issue the following command:
```
root@crypt04:~# openssl speed -evp aes-128-cbc
Doing aes-128-cbc for 3s on 16 size blocks: 20973798 aes-128-cbc's in 2.90s
Doing aes-128-cbc for 3s on 64 size blocks: 20252531 aes-128-cbc's in 2.93s
Doing aes-128-cbc for 3s on 256 size blocks: 15701632 aes-128-cbc's in 2.95s
Doing aes-128-cbc for 3s on 1024 size blocks: 8997398 aes-128-cbc's in 2.97s
Doing aes-128-cbc for 3s on 8192 size blocks: 1713347 aes-128-cbc's in 2.98s
Doing aes-128-cbc for 3s on 16384 size blocks: 876618 aes-128-cbc's in 2.97s
OpenSSL 1.1.0g  2 Nov 2017
built on: reproducible build, date unspecified
options:bn(64,64) rc4(8x,char) des(int) aes(partial) blowfish(ptr) 
compiler: gcc -DDSO_DLFCN -DHAVE_DLFCN_H -DNDEBUG -DOPENSSL_THREADS -DOPENSSL_NO_STATIC_ENGINE -DOPENSSL_PIC -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DRC4_ASM -DAES_ASM -DAES_CTR_ASM -DAES_XTS_ASM -DGHASH_ASM -DPOLY1305_ASM -DOPENSSLDIR="\"/usr/lib/ssl\"" -DENGINESDIR="\"/usr/lib/s390x-linux-gnu/engines-1.1\"" 
The 'numbers' are in 1000s of bytes per second processed.
type             16 bytes     64 bytes    256 bytes   1024 bytes   8192 bytes  16384 bytes
aes-128-cbc     115717.51k   442376.10k  1362582.30k  3102133.18k  4709979.40k  4835861.72k
```

The last line is quite interesting, it shows the encrypion bandwidth of one IFL - encrypting blocks of 8192 bytes of data over and over, as fast as possible. In this case, the observed throughput is roughly 4,8 GB/s (4835861.72 / 1,000,000 bytes).

Let's test now the decryption capabilities. Please issue the following command:
```
root@crypt04:~# openssl speed -evp aes-128-cbc -decrypt
Doing aes-128-cbc for 3s on 16 size blocks: 19572700 aes-128-cbc's in 2.95s
Doing aes-128-cbc for 3s on 64 size blocks: 19311413 aes-128-cbc's in 2.97s
Doing aes-128-cbc for 3s on 256 size blocks: 17775499 aes-128-cbc's in 2.97s
Doing aes-128-cbc for 3s on 1024 size blocks: 13684518 aes-128-cbc's in 2.98s
Doing aes-128-cbc for 3s on 8192 size blocks: 4411895 aes-128-cbc's in 2.98s
Doing aes-128-cbc for 3s on 16384 size blocks: 2285171 aes-128-cbc's in 2.98s
OpenSSL 1.1.0g  2 Nov 2017
built on: reproducible build, date unspecified
options:bn(64,64) rc4(8x,char) des(int) aes(partial) blowfish(ptr) 
compiler: gcc -DDSO_DLFCN -DHAVE_DLFCN_H -DNDEBUG -DOPENSSL_THREADS -DOPENSSL_NO_STATIC_ENGINE -DOPENSSL_PIC -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DRC4_ASM -DAES_ASM -DAES_CTR_ASM -DAES_XTS_ASM -DGHASH_ASM -DPOLY1305_ASM -DOPENSSLDIR="\"/usr/lib/ssl\"" -DENGINESDIR="\"/usr/lib/s390x-linux-gnu/engines-1.1\"" 
The 'numbers' are in 1000s of bytes per second processed.
type             16 bytes     64 bytes    256 bytes   1024 bytes   8192 bytes  16384 bytes
aes-128-cbc     106157.02k   416138.19k  1532164.22k  4702331.02k 12128269.74k 12563839.48k
```
Same computation, as you can see the observed throuthput is 12,5 GB/s. That is normal because the CBC mode of operation associated with the AES encryption algorithm is faster in decryption that in encryption.

Let's check now how much crypto we offload to the hardware. Please issue the following command:
```
root@crypt04:~# icastats
 function     |           hardware       |            software
--------------+--------------------------+-------------------------
              |      ENC    CRYPT   DEC  |      ENC    CRYPT   DEC 
--------------+--------------------------+-------------------------
        SHA-1 |               0          |                0
      SHA-224 |               0          |                0
      SHA-256 |               0          |                0
      SHA-384 |               0          |                0
      SHA-512 |               0          |                0
     SHA3-224 |               0          |                0
     SHA3-256 |               0          |                0
     SHA3-384 |               0          |                0
     SHA3-512 |               0          |                0
    SHAKE-128 |               0          |                0
    SHAKE-256 |               0          |                0
        GHASH |               0          |                0
        P_RNG |               0          |                0
 DRBG-SHA-512 |             676          |                0
       RSA-ME |               0          |                0
      RSA-CRT |               0          |                0
      DES ECB |         0              0 |         0             0
      DES CBC |         0              0 |         0             0
      DES OFB |         0              0 |         0             0
      DES CFB |         0              0 |         0             0
      DES CTR |         0              0 |         0             0
     DES CMAC |         0              0 |         0             0
     3DES ECB |         0              0 |         0             0
     3DES CBC |         0              0 |         0             0
     3DES OFB |         0              0 |         0             0
     3DES CFB |         0              0 |         0             0
     3DES CTR |         0              0 |         0             0
    3DES CMAC |         0              0 |         0             0
      AES ECB |         0              0 |         0             0
      AES CBC |  68515324       77041196 |         0             0
      AES OFB |         0              0 |         0             0
      AES CFB |         0              0 |         0             0
      AES CTR |         0              0 |         0             0
     AES CMAC |         0              0 |         0             0
      AES XTS |         0              0 |         0             0
      AES GCM |         0              0 |         0             0
```
**Note:** We can clearly see here the crypto offload in encryption operations. 68515324 operations were offloaded to the CPACF.
**Note2:** We can clearly see here the crypto offload in decryption operations. 77041196 operations were offloaded to the CPACF.

#### 3.2. Testing Pervasive encryption with scp
The scp command allows you to copy files over ssh connections. This is pretty useful if you want to transport files between computers, for example to backup something. The scp command uses the ssh protocol and they are very much alike. However, there are some important differences.
The scp command can be used in three* ways: to copy from a (remote) server to your computer, to copy from your computer to a (remote) server, and to copy from a (remote) server to another (remote) server. In the third case, the data is transferred directly between the servers; your own computer will only tell the servers what to do. These options are very useful for a lot of things that require files to be transferred
Let's first clean the icastats monitoring. Please issue the following command:
```
root@crypt06:~# icastats -r
```
The previous command reset the icastats monitoring interface, and ease to interpret future result. 

To confirm the reset took place, please issue the following command:
```
root@crypt04:~# icastats
 function     |           hardware       |            software
--------------+--------------------------+-------------------------
              |      ENC    CRYPT   DEC  |      ENC    CRYPT   DEC 
--------------+--------------------------+-------------------------
        SHA-1 |               0          |                0
      SHA-224 |               0          |                0
      SHA-256 |               0          |                0
      SHA-384 |               0          |                0
      SHA-512 |               0          |                0
     SHA3-224 |               0          |                0
     SHA3-256 |               0          |                0
     SHA3-384 |               0          |                0
     SHA3-512 |               0          |                0
    SHAKE-128 |               0          |                0
    SHAKE-256 |               0          |                0
        GHASH |               0          |                0
        P_RNG |               0          |                0
 DRBG-SHA-512 |               0          |                0
       RSA-ME |               0          |                0
      RSA-CRT |               0          |                0
      DES ECB |         0              0 |         0             0
      DES CBC |         0              0 |         0             0
      DES OFB |         0              0 |         0             0
      DES CFB |         0              0 |         0             0
      DES CTR |         0              0 |         0             0
     DES CMAC |         0              0 |         0             0
     3DES ECB |         0              0 |         0             0
     3DES CBC |         0              0 |         0             0
     3DES OFB |         0              0 |         0             0
     3DES CFB |         0              0 |         0             0
     3DES CTR |         0              0 |         0             0
    3DES CMAC |         0              0 |         0             0
      AES ECB |         0              0 |         0             0
      AES CBC |         0              0 |         0             0
      AES OFB |         0              0 |         0             0
      AES CFB |         0              0 |         0             0
      AES CTR |         0              0 |         0             0
     AES CMAC |         0              0 |         0             0
      AES XTS |         0              0 |         0             0
      AES GCM |         0              0 |         0             0
```

Let's now create a 512MB file that we will use later for a secure file transfer. 

Please issue the following command and note that it will take less than a minute:
```
root@crypt04:~# dd if=/dev/urandom of=data.512M bs=64M count=8 iflag=fullblock
8+0 records in
8+0 records out
536870912 bytes (537 MB, 512 MiB) copied, 3.26107 s, 165 MB/s
```
You just created a 512MB file made of random value. The file is named data.512MB.

You can confirm it issuing the following command:
```
root@crypt04:~# ls -hs
total 513M
   0 customdone.UBUN1804  4.0K custom.sh  513M data.512M
```
Now, let's see the hardware crypto offload with the SCP network protocol.

Please issue the following command, yes confirm if prompted, and enter your session password if prompted (azerty11):
```
root@crypt04:~# scp data.512M localhost:/dev/null
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ECDSA key fingerprint is SHA256:OLmh18ohxvJeIGbxf5LDuSfjS3Da16PRBzxz3tM9Bsg.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
root@localhost's password: 
data.512M                                                                       100% 512MB 122.8MB/s   00:04
```

As you can see, the defaut cipher suite transfered 512MB in 3 seconds, so a bandwidth of 122MB/s. Not so bad.

**Note:** AES is accelerated by default and is not reported because not passing through libica device driver.

**Note2:** By default, scp doesn't use the best ciphers of IBM Z and LinuxONE. However, it is possible to specify it manually with -c option.

Let's compare the defaut cipher performance with an optimized cipher. Please issue the following command:
```
root@crypt04:~# scp -c aes256-ctr data.512M localhost:/dev/null
root@localhost's password: 
data.512M                                                                        100%  512MB 270.4MB/s   00:01
```
As you can see, with 270MB/s, we increased the throughput by a factor of 2,25 . So, beware default settings. Make sure to use hardware accelerated ciphers by IBM Z and LinuxONE.

## Pervasive Encryption - Enabling dm-crypt to use the Hardware
The objective of this chapter, is to protect data at rest using dm-crypt in order to encrypt volumes. dm-crypt is very interresting, in that it helps to encrypt data without stopping running applications.

For the following, we will use LVM method to protect data at rest with dm-crypt at volume level. Objective will be to migrate data from an unencrypted volume to a (dm-crypt) encrypted volume. This is a 4 steps approach that doesn't required to reboot or to stop running application. 
4 steps includes the following:
* Step 1: Format a new encrypted volume with dm-crypt
* Step 2: Add dm-crypt based physical volume to volume group
* Step 3: Migrate data from non encrypted volume to encrypted volume
* Step 4: Remove unencrypted volume from the volume group

Let's do this for real now.

In your lab machine environnement there is an application running in a docker container (tomcat server). Please issue the following command:
```
root@crypt04:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
9df9d44cab08        s390x/tomcat:jre9   "catalina.sh run"   2 hours ago         Up 2 hours          0.0.0.0:8080->8080/tcp   unruffled_herschel
```

You can also confirm that the application is running using a web-browser and checking the url:
```
http://<your_lab_machine_ip>:8080/
```
![Image of still running tomcat application](https://github.com/guikarai/PE-LinuxONE/blob/master/images/tomcat-running.png)

Docker application and its data has been mounted in a volume group named vg01. This is the volume we will encryption without impacting running docker application. To assess the docker configuration on disk, please issue the following commands:
```
root@crypt04:~# dmsetup status
vg01-lv01: 0 20963328 linear
```
First of all, the volume group vg01 is not encrypted.

```
root@crypt04:~# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
dasda          94:0    0  6.9G  0 disk 
└─dasda1       94:1    0  6.9G  0 part /
dasdb          94:4    0  1.4G  0 disk 
└─dasdb1       94:5    0  1.4G  0 part [SWAP]
dasdc          94:8    0   10G  0 disk 
└─dasdc1       94:9    0   10G  0 part 
dasdd          94:12   0   10G  0 disk 
└─dasdd1       94:13   0   10G  0 part 
  └─vg01-lv01 253:0    0   10G  0 lvm  /var/lib/docker
```
Second, /var/lib/docker is mounted on the volume group vg01.

### 1. Installing cryptsetup
The cryptsetup feature provides an interface for configuring encryption on block devices (such as /home or swap partitions), using the Linux kernel device mapper target dm-crypt. It features integrated LUKS (Linux Unified Key Setup) support. LUKS standardizes the format of the encrypted disk, which allows different implementations, even from other operating systems, to access and decrypt the disk. 
LUKS adds metadata to the underlying block device, which contains information about the ciphers used and a default of eight key slots that hold an encrypted version of the master key used to decrypt the device. 
You can unlock the key slots by either providing a password on the command line or using a key file, which could, for example, be encrypted with gpg and stored on an NFS share.
First of all, let's install cryptsetup, you can issue the following command:
```
root@crypt04:~# sudo apt-get install cryptsetup
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  cryptsetup-bin
Suggested packages:
  keyutils
The following NEW packages will be installed
  cryptsetup cryptsetup-bin
0 to upgrade, 2 to newly install, 0 to remove and 0 not to upgrade.
Need to get 238 kB of archives.
After this operation, 842 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
```
Press "y" when prompted.

The dm-crypt feature supports various ciphers and hashing algorithms that you can select from the ones that are available in the Kernel and listed in the /proc/crypto procfs file. This also means that dm-crypt takes advantage of the unique hardware acceleration features of IBM Z that increase encryption and decryption speed.
Using the cryptsetup command, create a LUKS partition on the respective disk devices. For full disk encryption, use the AES xts hardware feature. We choose the AES-xts to achieve a security level of reasonable quality with the best encryption mode.

To confirm that is the best choice, you can issue the following command:
```
root@crypt04:~# cryptsetup benchmark
# Tests are approximate using memory only (no storage IO).
PBKDF2-sha1       846991 iterations per second for 256-bit key
PBKDF2-sha256    1235071 iterations per second for 256-bit key
PBKDF2-sha512     720175 iterations per second for 256-bit key
PBKDF2-ripemd160  667033 iterations per second for 256-bit key
PBKDF2-whirlpool  364595 iterations per second for 256-bit key
argon2i       4 iterations, 498666 memory, 4 parallel threads (CPUs) for 256-bit key (requested 2000 ms time)
argon2id      4 iterations, 475368 memory, 4 parallel threads (CPUs) for 256-bit key (requested 2000 ms time)
#     Algorithm | Key |  Encryption |  Decryption
        aes-cbc   128b  2162.4 MiB/s  2907.6 MiB/s
    serpent-cbc   128b    65.2 MiB/s    70.5 MiB/s
    twofish-cbc   128b   130.5 MiB/s   136.8 MiB/s
        aes-cbc   256b  1774.1 MiB/s  2471.4 MiB/s
    serpent-cbc   256b    57.9 MiB/s    62.9 MiB/s
    twofish-cbc   256b   114.4 MiB/s   120.3 MiB/s
        aes-xts   256b  2520.7 MiB/s  2772.3 MiB/s
    serpent-xts   256b    70.5 MiB/s    75.0 MiB/s
    twofish-xts   256b   157.1 MiB/s   146.6 MiB/s
        aes-xts   512b  2776.7 MiB/s  2821.7 MiB/s # <-------- This is the best choice
    serpent-xts   512b    71.8 MiB/s    74.5 MiB/s
    twofish-xts   512b   158.9 MiB/s   149.1 MiB/s

```
### 2. Using dm-crypt Volumes as LVM Physical Volumes
#### 2.1. Initial physical volume assessment
```
root@crypt04:~# pvs
  PV          VG   Fmt  Attr PSize   PFree 
  /dev/dasdc1      lvm2 ---   10.00g 10.00g
  /dev/dasdd1 vg01 lvm2 a--  <10.00g     0 
```

**Note:** There is one physical volume which is not yet part of any volume group. This is the disk we'll use with cryptsetup.
In this example, this device is /dev/dasdc1. Yours may be different.

#### 2.2. Initial volume group assessment
```
root@crypt04:~# vgs
  VG   #PV #LV #SN Attr   VSize   VFree
  vg01   1   1   0 wz--n- <10.00g    0 
```

#### 2.3. Initial logical volume assessment
```
root@crypt04:~# lvs
  LV   VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv01 vg01 -wi-ao---- <10.00g 
```

### 3.1 Step 1 - Formatting and encrypting a new volume
In the following step, we will format and encrypt an existing volume.
![Step1](https://github.com/guikarai/PE-LinuxONE/blob/master/images/step1.png)
For the following and to simplify the lab, we will use a passphrase to generate an encryption key. 

Passphras is a mean, but there are others including:
* gpgkey
* protected key (using the pKEY tool)

Be sure to remember the entered passphrase. You can simply use "cryptlab" to make it simple. The following command will create a cryptographic device mapper device in LUKS encryption mode:

```
root@crypt04:~# cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/dasdc1

WARNING!
========
This will overwrite data on /dev/dasdc1 irrevocably.

Are you sure? (Type uppercase yes): YES
Enter passphrase for /dev/dasdc1: #<--- cryptlab OR your passphrase
Verify passphrase:                #<--- cryptlab OR your passphrase
```

Next, you have to open the volume onto the device mapper. This is the stage at which you will be prompted for your passphrase. You can choose the name that you want your partition mapped under. We proposed to you "dockercrypt".
```
root@crypt04:~# cryptsetup luksOpen /dev/dasdc1 dockercrypt
Enter passphrase for /dev/dasdc1:     #<--- cryptlab OR your passphrase
```

### 3.2 Step 2 - Add dm-crypt based physical volume to volume group
In this second step, we will add the encrypted volume into the existing volume group. Both an encrypted volume and unencrypted volume will be part of the same volume groupe.
![Step2](https://github.com/guikarai/PE-LinuxONE/blob/master/images/step2.png)

Now we will use the device as phisical volume. Please issue the following command:
```
root@crypt06:~# pvcreate /dev/mapper/dockercrypt 
  Physical volume "/dev/mapper/dockercrypt" successfully created.
```

Now we will add in the volume group the new physical device /dev/mapper/dockercrypt. Please issue the following command:
```
vgextend vg01 /dev/mapper/dockercrypt 
  Volume group "vg01" successfully extended
```

Let's check the new volume group status. As you can see, there is now 2 physical volume in the volume group.
```
root@crypt04:~# vgs
  VG   #PV #LV #SN Attr   VSize  VFree  
  vg01   2   1   0 wz--n- 19.99g <10.00g
```

Checking physical volume, we can see that the encrypted new one is there aside from the initial one. Both part of the same volume group vg01.
```
root@crypt04:~# pvs
  PV                      VG   Fmt  Attr PSize   PFree  
  /dev/dasdc1                  lvm2 ---   10.00g  10.00g
  /dev/dasdd1             vg01 lvm2 a--  <10.00g      0 
  /dev/mapper/dockercrypt vg01 lvm2 a--  <10.00g <10.00g
```

You can use dmsetup status to check if there are any LUKS-encrypted partitions. The output should look something like:
```
root@crypt04:~# dmsetup status
dockercrypt: 0 20967872 crypt 
vg01-lv01: 0 20963328 linear
```
The line marked "crypt" shows that PV has been encrypted. You can see which filesystems are on that via the lvm tools.


### 3.3 Step 3 - Migrate data from non encrypted volume to encrypted volume
In this third step, we will migrate unencrypted data in the uncrypted physical volume to the encrypted physical volume. This operation thanks to the **pvmove** switch update from the source PV to the destination PV once completed. This command doesn't halt running application. This operation can take some time. 
![Step3](https://github.com/guikarai/PE-LinuxONE/blob/master/images/step3.png)

Please issue to following command to proceede:
```
root@crypt04:~# pvmove /dev/dasdd1 /dev/mapper/dockercrypt
  /dev/dasdd1: Moved: 0.04%
  /dev/dasdd1: Moved: 18.87%
  /dev/dasdd1: Moved: 37.55%
  /dev/dasdd1: Moved: 56.00%
  /dev/dasdd1: Moved: 75.07%
  /dev/dasdd1: Moved: 93.32%
  /dev/dasdd1: Moved: 100.00%
```

### 3.4 Step 4 - Remove unencrypted volume from the volume group
Fourth and last step. It is time to remove from the volume group, the volume that is unencrypted, we don't need it anymore.
![Step4](https://github.com/guikarai/PE-LinuxONE/blob/master/images/step4.png)

Do do so, we will use the **vgreduce**. Please issue the following command:
```
root@crypt04:~# vgreduce vg01 /dev/dasdd1
  Removed "/dev/dasdd1" from volume group "vg01"
```

Let's check the new configuration of LV, VG and PV: Please issue the following command:
```
root@crypt04:~# lvs
  LV   VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv01 vg01 -wi-ao---- <10.00g                                                    
```

```
root@crypt04:~# vgs
  VG   #PV #LV #SN Attr   VSize   VFree
  vg01   1   1   0 wz--n- <10.00g    0 
 
```

```
root@crypt04:~# pvs
  PV                      VG   Fmt  Attr PSize   PFree 
  /dev/dasdc1                  lvm2 ---   10.00g 10.00g
  /dev/dasdd1                  lvm2 ---   10.00g 10.00g
  /dev/mapper/dockercrypt vg01 lvm2 a--  <10.00g     0 
```

Now, application run 100% on a encrypted volume. Let's check if application is still running. Please issue the following command:
```
root@crypt04:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
9df9d44cab08        s390x/tomcat:jre9   "catalina.sh run"   2 hours ago         Up 2 hours          0.0.0.0:8080->8080/tcp   unruffled_herschel
```

You can also confirm that the application is running using a web-browser and checking the url:
```
http://<your_lab_machine_ip>:8080/
```
If you see the following, you did it!
![Image of still running tomcat application](https://github.com/guikarai/PE-LinuxONE/blob/master/images/tomcat-running.png)

To guaranty you that docker runs on the encrypted devices, please issue the following command:
```
root@crypt04:~# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
dasda            94:0    0  6.9G  0 disk  
└─dasda1         94:1    0  6.9G  0 part  /
dasdb            94:4    0  1.4G  0 disk  
└─dasdb1         94:5    0  1.4G  0 part  [SWAP]
dasdc            94:8    0   10G  0 disk  
└─dasdc1         94:9    0   10G  0 part  
  └─dockercrypt 253:1    0   10G  0 crypt 
    └─vg01-lv01 253:0    0   10G  0 lvm   /var/lib/docker
dasdd            94:12   0   10G  0 disk  
└─dasdd1         94:13   0   10G  0 part 
```

To be sure that there is a prompt after after a reboot, please create /etc/crypttab with the following content:
```
root@crypt04:~# vi /etc/crypttab
dockercrypt /dev/disk/by-path/ccw-0.0.0201-part1 none
```

You just finished the Pervasive encryption LAB. Thank you, and see you soon.

