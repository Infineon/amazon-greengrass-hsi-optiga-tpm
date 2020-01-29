# AWS IoT Greengrass Hardware Security Integration: Provide hardware-based endpoint device security with Infineon's OPTIGA&trade; TPM SLx 9670


## Introduction

This document contains step-by-step instructions on how to setup the Open Source TPM Software Stack 2.0 (TSS 2.0)
in combination with the tpm2-pkcs11 provider and related software on a Raspberry Pi&reg; 3 Linux environment to use the
Trusted Platform Module OPTIGA&trade; TPM SLx 9670 TPM2.0 from Infineon Technologies as a Hardware Security Module (HSM) for AWS IoT Greengrass
using the PKCS#11 Hardware Security Integration (HSI).

The OPTIGA&trade; TPM SLx 9670 TPM2.0 product family with SPI interface consists of 3 different products:

 * OPTIGA&trade; TPM SLB 9670, standard security applications
 * OPTIGA&trade; TPM SLI 9670, automotive security applications
 * OPTIGA&trade; TPM SLM 9670, industrial security applications

We refer with "OPTIGA&trade; TPM SLx TPM2.0" to all of the above 3 variants of the OPTIGA&trade; TPM2.0 product family with SPI interface.

The described steps to use an OPTIGA&trade; TPM as a an Hardware Security Module for AWS IoT Greengrass on an Raspberry Pi&reg; 3 Linux Environment can be performed with one of the Infineon Iridium SLx 9670 TPM2.0 SPI Boards, listed in the Table below:

| Supported TPM                    |  Order Type             | Order number | FW Version |
|----------------------------------|-------------------------|--------------|------------|
|OPTIGA&trade; TPM SLB 9670 TPM2.0 | IRIDIUM 9670 TPM2.0     | SP001596592  | 7.85       |
|OPTIGA&trade; TPM SLI 9670 TPM2.0 | IRIDIUM SLI 9670 TPM2.0 | SP004232000  | 13.11      |
|OPTIGA&trade; TPM SLM 9670 TPM2.0 | IRIDIUM SLM 9670 TPM2.0 | SP004232004  | 13.11      |

The 3 Infineon Iridium Boards are referred in the following as "Infineon Iridium SLx 9670 TPM2.0 SPI Board".

The Software has been tested with Infineon TPMs implementing TCG Revision 1.38 or higher.

Iridium Boards with OPTIGA&trade; TPM SLB 9670 might have a lower firmware (7.40 or 7.63) and may need to be upgraded first.
Iridium Boards with OPTIGA&trade; TPM SLI 9670 and OPTIGA&trade; TPM SLM 9670 should have FW 13.11.

Please refer to eltt2 section below on how to check the version of your TPM.

### Intended Audience
This document is intended for customers who want to increase the security level of their AWS IoT Greengrass deployments
using an OPTIGA&trade; TPM SLx9670 TPM2.0 from Infineon as a Hardware Security Module, leveraging the capabilities of the 
AWS Greengrass Hardware Security Intergration (HSI) via the tpm2-pkcs11 provider library.

### Getting Started with AWS IoT Greengrass
In case you are not yet familiar with AWS IoT Greengrass please have a look at:
- https://aws.amazon.com/de/greengrass/ 
- https://docs.aws.amazon.com/greengrass/latest/developerguide/what-is-gg.html

For more details on AWS IoT Greengrass HSI please refer to https://docs.aws.amazon.com/greengrass/latest/developerguide/hardware-security.html.

It is strongly recommended to follow the AWS IoT Greengrass tutorial before using this guide to support the use of OPTIGA&trade; TPM with HSI



### Quality and Limitations:
The PKCS11 Library (tpm2-pkcs11) and the underlying TPM2 Software Stack (tpm2-tss) are part of the Open Source Project http://tpm2-software.github.io 
which is supported, developed and sponsored by Infineon and many others.

The software:
- is only tested to a limited extend and might not work as expected.
- is provided as-is, without any waranty and liability.
- provides the required functionality for __Greengrass Device Tester 1.3.1__ and __IoT Greengrass 1.8.x and 1.9.x__
- has NOT been tested for any additional functionality.

Only RSA 2K Keys and ECC_NIST_P256 keys are supported.

## Preparation and Hardware Setup

- Download latest Raspbian (2018-11) and flash onto SD Card.
- Plugin OPTIGA&trade; TPM SLx 9670 Iridium Board on Raspberry Pi Header.
  - The chips must be facing the outside of the Raspberry Pi.
  - Pin 1 of the Iridium must align with Pin 1 of the Raspberry Pi.
  - Pin 1 is also marked by a rectangular solder pad on the Iridium board.
- Plugin SD Card, Monitor, Keyboard, Mouse into Raspberry Pi and power it up.
- Follow basic Raspberry Pi Setup instructions, especially Wifi and User Password.
- Use 'raspi-setup' to enable SSH and SPI.
- Update your system with `sudo apt update && sudo apt upgrade`.
- Install latest kernel via `sudo rpi-update`.
- Edit */boot/config.txt* and add the following line:

    ```dtoverlay=tpm-slb9670```

  (this tpm-slb9670 overlay applies to SLB 9670, SLI 9670 and SLM 9670).
- Reboot your Raspberry Pi and check that */dev/tpm0* is available.
- Follow the AWS IoT Greengrass Tutorial to setup everything correctly.

## Check TPM Functionality with eltt2
eltt2 is a small test utility provided by Infineon Technologies AG and is available on github:

    git clone https://github.com/infineon/eltt2
    cd eltt2
    make
    sudo ./eltt2 -g
    cd ..

The output should look similar to this:

    TPM capability information of fixed properties:
    =========================================================
    TPM_PT_FAMILY_INDICATOR:        2.0
    TPM_PT_LEVEL:                   0
    TPM_PT_REVISION:                138
    TPM_PT_DAY_OF_YEAR:             8
    TPM_PT_YEAR:                    2018
    TPM_PT_MANUFACTURER:            IFX
    TPM_PT_VENDOR_STRING:           SLI9670
    TPM_PT_VENDOR_TPM_TYPE:         0
    TPM_PT_FIRMWARE_VERSION:        13.11.4555.0

This means your OPTIGA&trade; TPM works as expected.
It also shows the Firmware Version of the TPM.


## Install TPM Software Stack and Tools
### Install preconditions
    sudo apt update && sudo apt upgrade

    sudo apt -y install autoconf automake libtool pkg-config gcc libssl-dev \
      libcurl4-gnutls-dev libdbus-1-dev libglib2.0-dev autoconf-archive libcmocka0 \
      libcmocka-dev net-tools build-essential git pkg-config gcc g++ m4 libtool \
      automake libgcrypt20-dev libssl-dev uthash-dev autoconf doxygen pandoc \
      libsqlite3-dev python-yaml p11-kit opensc gnutls-bin libp11-kit-dev \
      python3-yaml cscope

    sudo apt-get build-dep libengine-pkcs11-openssl1.1

### Download Repositories
    git clone https://github.com/tpm2-software/tpm2-tss
    git clone https://github.com/tpm2-software/tpm2-tools
    git clone https://github.com/tpm2-software/tpm2-abrmd
    git clone https://github.com/tpm2-software/tpm2-pkcs11
    git clone https://github.com/OpenSC/libp11.git


### Install libp11 in a recent version
Unfortunately the version of libp11 pkcs11 engine for openssl provided on Rasbian Stretch is too old (0.4.4) and not compatible with this software.
So we have to install it manually from the repositories.

Compile and install the correct version:

    cd libp11
    git checkout libp11-0.4.9
    ./bootstrap
    ./configure
    make -j4
    sudo make install
    cd ..

### Download autoconf-archive and deploy to the projects
Unfortunately the version of autoconf-archive provided on Rasbian Stretch is too old and not compatible with this software.
We have to download it manually and copy it to the respective repositories.


    wget https://github.com/autoconf-archive/autoconf-archive/archive/v2018.03.13.tar.gz
    tar -xvf v2018.03.13.tar.gz
    cp -r autoconf-archive-2018.03.13/m4/ tpm2-tss/
    cp -r autoconf-archive-2018.03.13/m4/ tpm2-abrmd/
    cp -r autoconf-archive-2018.03.13/m4/ tpm2-tools/
    cp -r autoconf-archive-2018.03.13/m4/ tpm2-pkcs11/


### Install tpm2-tss
    cd tpm2-tss
    git checkout 740653a12e203b214cba2f07b5395ffce74dfc03
    ./bootstrap -I m4
    ./configure --with-udevrulesdir=/etc/udev/rules.d --with-udevrulesprefix=70-
    make -j4
    sudo make install
    sudo useradd --system --user-group tss
    sudo udevadm control --reload-rules && sudo udevadm trigger
    sudo ldconfig
    cd ..

### Install tpm2-abrmd
    cd tpm2-abrmd
    git checkout 2.1.1
    ./bootstrap -I m4
    ./configure --with-dbuspolicydir=/etc/dbus-1/system.d \
      --with-systemdsystemunitdir=/lib/systemd/system \
      --with-systemdpresetdir=/lib/systemd/system-preset \
      --datarootdir=/usr/share
    make -j4
    sudo make install
    sudo ldconfig
    sudo pkill -HUP dbus-daemon
    sudo systemctl daemon-reload
    sudo systemctl enable tpm2-abrmd.service
    sudo systemctl start tpm2-abrmd.service
    dbus-send --system --dest=org.freedesktop.DBus --type=method_call \
      --print-reply /org/freedesktop/DBus org.freedesktop.DBus.ListNames \
      | grep "com.intel.tss2.Tabrmd" || echo "ERROR: abrmd was not installed correctly!"
    cd ..

### Install tpm2-tools
    cd tpm2-tools
    git checkout 3e8847c9a52a6adc80bcd66dc1321210611654be
    ./bootstrap -I m4
    ./configure
    make -j4
    sudo make install
    cd ..

### Install tpm2-pkcs11
    sudo mkdir -p /opt/tpm2-pkcs11
    sudo chmod 777 /opt/tpm2-pkcs11
    cd tpm2-pkcs11/
    git checkout a82d0709c97c88cc2e457ba111b6f51f21c22260
    ./bootstrap -I m4
    ./configure --enable-esapi-session-manage-flags --with-storedir=/opt/tpm2-pkcs11
    make -j4
    sudo make install
    cd ..


## Using the PKCS11 Provider for Greengrass HSI.

In this example, the keystore is created under /opt/tpm2-pkcs11.
It is assumed, that the location is read-/writeable to the user.

### Initializing Keystore and Token
    cd tpm2-pkcs11/tools/

#### Init Keystore
    ./tpm2_ptool.py init --pobj-pin=123456 --path=/opt/tpm2-pkcs11/

The used options are:

    --pobj-pin POBJ_PIN   The authorization password for adding secondary objects under the primary object.
    --path PATH           The location of the store directory.


#### Init Token
    ./tpm2_ptool.py addtoken --pid=1 --pobj-pin=123456 --sopin=123456 --userpin=123456 --label=greengrass --path=/opt/tpm2-pkcs11/

The used options are:

      --pid PID             The primary object id to associate with this token.
      --sopin SOPIN         The Administrator pin. This pin is used for object recovery.
      --userpin USERPIN     The user pin. This pin is used for authentication for object use.
      --pobj-pin POBJ_PIN   The primary object password. This password is use for authentication to the primary object.
      --label LABEL         A unique label to identify the profile in use, must be unique.
      --path PATH           The location of the store directory.


#### Add a key:
    ./tpm2_ptool.py addkey --algorithm=rsa2048 --label=greengrass --userpin=123456 --key-label=greenkey --path=/opt/tpm2-pkcs11/

The used options are

    --id ID               The key id. Defaults to a random 8 bytes of hex.
    --sopin SOPIN         The Administrator pin.
    --userpin USERPIN     The User pin.
    --label LABEL         The tokens label to add a key too.
    --algorithm {rsa2048,ecc256}
                          The type of the key. Only RSA 2048 and ECC 256 are supported.
    --key-label KEY_LABEL
                          The key label to identify the key. Defaults to an integer value.
    --path PATH           The location of the store directory.

### Find out the P11/PKCS#11 URL
Greengrass and other tools use a pkcs11 url to find the token/key object.
This URL can be determined using `p11tool`:

    p11tool --list-token-urls

This will yield a result similar to:

    pkcs11:model=p11-kit-trust;manufacturer=PKCS%2311%20Kit;serial=1;token=System%20Trust
    pkcs11:model=SLI9670;manufacturer=Infineon;serial=0000000000000000;token=greengrass

The URL for the private key can then be determined using:

    p11tool --list-privkeys pkcs11:manufacturer=Infineon
    Object 0:
        URL: pkcs11:model=SLI9670;manufacturer=Infineon;serial=0000000000000000;token=greengrass;id=%37%33%61%36%62%30%31%37%39%66%39%33%39%38%62%38;object=greenkey;type=private
        Type: Private key
        Label: greenkey
        Flags: CKA_NEVER_EXTRACTABLE; CKA_SENSITIVE;
        ID: 37:33:61:36:62:30:31:37:39:66:39:33:39:38:62:38


The URL can be trimmed of certain components, as long as it remains unique, e.g.

    pkcs11:model=SLI9670;manufacturer=Infineon;token=greengrass;object=greenkey;type=private

The Pin can be appended to the URL:

    pkcs11:model=SLI9670;manufacturer=Infineon;token=greengrass;object=greenkey;type=private;pin-value=123456

This will be the URL we will use for the Greengrass configuration.


### Generate a Certificate Signing Request

    openssl req -engine pkcs11 -new -key "pkcs11:model=SLI9670;manufacturer=Infineon;token=greengrass;object=greenkey;type=private;pin-value=123456" -keyform engine -out /tmp/req.csr

Please answer the questions OpenSSL is asking you for the Certificate Signing Request - these information will be incorporated into the certificate.

Once completed, login to AWS and navigate to the AWS IoT Section.

Under the `Security -> Certificates` tab on the left menu, create a new certificate (Right upper corner `create`).

In the menu chose `Create with CSR` and select the `.csr` file you created using openssl. (e.g. `/tmp/req.csr`)

Download the `root.ca.crt` and the resulting `xxxxxx-certificate.pem.crt`, where _xxxxxx_ stands for a unique id, and copy both to `/greengrass/certs/`

Before closing the window, please be sure to activate the certificate and attach it to an object/policy in the dialogue on the AWS Greengrass Security Page.


### Configure and run Greengrass with HSI
To enable and use the TPM as HSI, we need to enable it in the greengrass config.
For this we need to edit `/greengrass/config/config.json` and replace the configuration with the following content:
```yaml
    {
        "crypto": {
            "caPath": "file:///greengrass/certs/root.ca.crt",
            "PKCS11": {
                "OpenSSLEngine": "/usr/lib/arm-linux-gnueabihf/engines-1.1/pkcs11.so",
                "P11Provider": "/usr/lib/arm-linux-gnueabihf/pkcs11/libtpm2_pkcs11.so",
                "SlotLabel": "greengrass",
                "SlotUserPin": "123456"
            },
            "principals": {
                "IoTCertificate": {
                    "certificatePath": "file:///greengrass/certs/_xxxxxx_-certificate.pem.crt",
                    "privateKeyPath": "pkcs11:model=SLI9670;manufacturer=Infineon;token=greengrass;object=greenkey;type=private;pin-value=123456"

                },
                "MQTTServerCertificate": {
                    "certificatePath": "file:///greengrass/certs/_xxxxxx_-certificate.pem.crt",
                    "privateKeyPath": "pkcs11:model=SLI9670;manufacturer=Infineon;token=greengrass;object=greenkey;type=private;pin-value=123456"
                }
            }
        },
        "coreThing" : {
            "thingArn" : "arn:aws:iot:eu-central-1:ZZZZZZZZZZZZZZZ:thing/Greengrass-Test_Core",
            "iotHost" : "YYYYYYYYYYYYYYY.iot.eu-central-1.amazonaws.com",
            "ggHost" : "greengrass.iot.eu-central-1.amazonaws.com",
            "keepAlive" : 600
        },
        "runtime" : {
            "cgroup" : {
                "useSystemd" : "yes"
            }
        },
        "managedRespawn" : false
    }
```

Please adjust the `certificatePath`, `privateKeyPath`, `thingArn` and `iotHost` accordingly.

After this you can start your greengrass daemon as usual:

    cd /greengrass/ggc/core
    sudo ./greengrassd start

## Troubleshooting
### Greengrass is not starting
Please validate that your environment is prepared for Greengrass (especially memory cgroups are on), by following the regular greengrass tutorials without hsi.

Please also validate that the permissions and user groups are set up correctly.

### /dev/tpm0 is not showing up
Please make sure that you are running the latest kernel from `rpi-update`, that the SPI support is turned on using `raspi-setup` and that the overlay is enabled in `/boot/config.txt`

Please also ensure that the Iridium board is plugged in correctly.

### Debug PKCS11 Provider
In order to enable more verbose logging an environment variable can be set:

    TPM2_PKCS11_LOG_LEVEL=2

Also the pkcs11-spy from libp11 can be used to get a deeper understanding of the PKCS#11 calls.



## Trademarks
All references product or service names and trademarks are the property of their respective owners.


## Important Notice
The information contained in this application note is given as a hint for the implementation of the product only and shall in no event be regarded as a description or warranty of a certain functionality, condition or quality of the product. Before implementation of the product, the recipient of this application note must verify any function and other technical information given herein in the real application. Infineon Technologies hereby disclaims any and all warranties and liabilities of any kind (including without limitation warranties of non-infringement of intellectual property rights of any third party) with respect to any and all information given in this application note. 

The data contained in this document is exclusively intended for technically trained staff. It is the responsibility of customer’s technical departments to evaluate the suitability of the product for the intended application and the completeness of the product information given in this document with respect to such application.    
	
For further information on the product, technology, delivery terms and conditions and prices please contact your nearest Infineon Technologies office (www.infineon.com).

## Warnings
Due to technical requirements products may contain dangerous substances. For information on the types in question please contact your nearest Infineon Technologies office.

Except as otherwise explicitly approved by Infineon Technologies in a written document signed by authorized representatives of Infineon Technologies, Infineon Technologies’ products may not be used in any applications where a failure of the product or any consequences of the use thereof can reasonably be expected to result in personal injury.


## In case of further questions:
Please raise an issue on github or contact dsscustomerservice@infineon.com
