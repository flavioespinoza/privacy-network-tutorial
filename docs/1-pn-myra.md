<!-- ---
html:
  embed_local_images: false
  embed_svg: true
  offline: false
  toc: undefined

print_background: false

export_on_save:
  html: true
--- -->

# Overview & Setup
This document walks thru the detailed steps of PN enabling a test service and a client of the test service on your local machine, in the process one sets up a local privacy network consisting of two privacy domains.

## 1. Privacy Network - MyRA Resource Authority

This document walks thru the detailed steps of provisioning a Resouce Authority on you local dev environment

### Management Plane Metadata

If the whole network is running on your local machine then a local file system can be used otherwise a file system that can be seen by all domains in the network needs to be used (or the metadata copied around). The repo is accessed by all the WebShield software running in the network.

### Metadata - Create Repo

Create

```bash 
cd ~/webshield-dev/privacy-networks/testnet
mkdir db && cd db
mkdir diddoc
mkdir resource
mkdir credential
```

Verify
```bash  {cmd=true}
cd ~/webshield-dev/privacy-networks/testnet
tree db
```

### Metadata - copydb Directory

- Create directory under `service-pd` and `client-pd` to copy metadata to

Create in `service-pd`

```bash {.copy-clip}
cd ~/webshield-dev/privacy-networks/testnet/service-pd
mkdir copydb
```

Create in `client-pd`

```bash {.copy-clip}
cd ~/webshield-dev/privacy-networks/testnet/client-pd
mkdir copydb
```

## 2. Testnet Directory Structure
Local directory structure for all artifacts -- directories only

```shell
privacy-networks/testnet/
│
├── db `holds the metadata repo`
│   ├── credential
│   ├── diddoc
│   └── resource
│
├── myra `resource authoritry used to issue all the resource`
│   ├── config
│   ├── init-template
│   └── keys `the signing key`
│
├── myta `used to issue credentials to resource`
│   ├── config
│   ├── init-template
│   └── keys `the signing key`
│
├── client-pd  `holds the tests that invoke the resource server over pipes`
│   ├── config `holds config for client-pd services`
│   ├── copydb `holds a copy of db so can be mounted as volume by docker-compose from this dir`
│   │   ├── credential
│   │   ├── diddoc
│   │   └── resource
│   │
│   ├── pa
│   │   └── keys
│   │
│   ├── pd-artifacts
│   │   └── keys
│   │
│   └── ta `used for local pipe authorization`
│       ├── init_template
│       └── keys
│
└── service-pd `holds the PN enabled resource test service`
    ├── config `holds config for service-pd services`
    ├── copydb `holds a copy of db so can be mounted as volume by docker-compose from this dir`
    │   ├── credential
    │   ├── diddoc
    │   └── resource
    │
    ├── pa
    │   └── keys
    │
    ├── pd-artifacts
    │   └── keys
    │
    ├── rs `used to query resources`
    │   └── keys
    │
    └── ta `used for local pipe authorization`
        ├── init_template
        └── keys

```

To see a current directory tree with all directories and files run the command below

```bash
tree ~/webshield-dev/privacy-networks/testnet
```


## 3. MyRa Resource Authority

Provision a Resource Authority on you local dev environment

### MyRA - RSA Keys - Generate

```bash
cd ~/webshield-dev/privacy-networks/testnet/myra/keys 
bash ../../../tools/genkeys/generateRSA.bash
```

Since the certificate is not used on your local dev environment hit the Enter-key for each prompt except **Common Name** which we will name `myra.not.used`

```bash
# Enter this info
Common Name (eg, fully qualified host name) []: `myra.not.used` # <-- Enter this info only
```

### MyRA - RSA Keys - Rename

Rename each key with the prefix `sign`

```bash {.copy-clip}
cd ~/webshield-dev/privacy-networks/testnet/myra/keys
mv rsa.cert.pem sign.rsa.cert.pem
mv rsa.key.pem sign.rsa.key.pem
mv rsa.public.key.pem sign.rsa.public.key.pem
```

### MyRA - DID Doc
For identity and resolving identities to a trusted public key and descriptive metadata the implementation uses the decentralized identifiers (`DID`) standard that can be resolved to `DID Doc`s
https://w3c-ccg.github.io/did-spec/. 
The `DID Doc` contains both metadata about the entity and its public key information. The DID can be resolved to a DID Document


#### Naming Conventions
The testnet does not use a `DLT` hence the `DID`s are `wsutm` identifiers that for the test network are `currently handcrafted` using the following patterns:


#### Resource Authority & Trust Authority (RA & TA) 

- **Pattern:** 
did:wsutm:`<domain-name>`-`<authority-type>`-authority-`<authority-name>`

- **Example:** 
did:wsutm:localhost-resource-authority-myra

#### Resources

- **Pattern:** 
did:wsutm:`<domain-name>`-resource-`<type>`-`<id>`

- **Example:** 
did:wsutm:localhost-resource-pd-test-service

### MyRA - DID Doc - Template

We will use the `diddoc_template.yaml` below to create our `ra_diddoc.yaml` file

<div class='md-alert mb-12'>DO NOT COPY -- FOR REFRENCE ONLY</div>

**diddoc_template.yaml**

<!-- @import "../templates/diddoc/diddoc_template.yaml" -->

### MyRA - DID Doc - Remove Existing

Delete existing `ra_diddoc.yaml` file if it exists

```bash {.copy-clip}
rm -f ~/webshield-dev/privacy-networks/testnet/myra/init-template/ra_diddoc.yaml
```

### MyRA - DID Doc - Copy Template
Copy the `diddoc_template.yaml` into the `init-template` directory and name it `ra_diddoc.yaml`

```bash {.copy-clip}
cp ~/webshield-dev/privacy-networks/templates/diddoc/diddoc_template.yaml \
~/webshield-dev/privacy-networks/testnet/myra/init-template/ra_diddoc.yaml
```

### MyRA - DID Doc - Replace Values

Open file `/privacy-networks/testnet/myra/init-template/ra_diddoc.yaml` and replace values where indicated

<div class='md-label'></div>

```yaml
context: https://w3id.org/did/v1
id: REPLACE-WITH-DID
name: DID document template replace the REPLACE items accordingly
comment: using for local machine
publicKey:
  - id: REPLACE-WITH-DID#keys-RSA
    name: a RSA key
    publicKeyPem: |
      -----BEGIN PUBLIC KEY-----
        REPLACE WITH SIGNING RSA PUBLIC KEY
      -----END PUBLIC KEY-----
authentication:
  - type: rsaSignatureAuthentication2018
    publicKey: REPLACE-WITH-DID#keys-RSA
service:
```


#### Replace DID
Copy DID below and replace where is says `REPLACE-WITH-DID`
```bash
did:wsutm:localhost-resource-authority-myra
```

#### Replace RSA Public Key
Copy the `sign.rsa.public.key` to the clipboard

```bash
pbcopy < ~/webshield-dev/privacy-networks/testnet/myra/keys/sign.rsa.public.key.pem
```

If you do not have `pbcopy` installed use `cat` to copy to the `Terminal` output
```bash {.copy-clip}
cat ~/webshield-dev/privacy-networks/testnet/myra/keys/sign.rsa.public.key.pem
```

Replace here

```yaml
    publicKeyPem: |
      -----BEGIN PUBLIC KEY-----
        REPLACE WITH SIGNING RSA PUBLIC KEY
      -----END PUBLIC KEY-----
```
Save file

### MyRA - The DID Doc

<div class='md-alert mb-12'>DO NOT COPY -- FOR REFRENCE ONLY</div>

Your new `ra_diddoc.yaml` should look something like this when you are done

```yaml
context: https://w3id.org/did/v1
id: did:wsutm:localhost-resource-authority-myra
name: RSA DID Doc Localhost
comment: using for local machine
publicKey:
  - id: did:wsutm:localhost-resource-authority-myra#keys-RSA
    name: a RSA key
    publicKeyPem: |
      -----BEGIN PUBLIC KEY-----
      MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAvanGD5iWvLh/MRd/01aN
      OpZYWcketiiiOXf/aXsYXVedKJNPVuKbX2WdmKGkM4dw0xSTzzRFWJFpcJPU/DvT
      K5vctzspthAMqb1rMZ9j4e2xV696RqIndr26qtsNMOdMhW5ALkka80biwmsrDcNI
      CcOQzNU8qLLtlOeCfaRdwp/IQ4t59XKs6K4STvHdkDI+f9eMV/7Tz4IG0q91ivO+
      LBmbAHYSmuZ+tXIrHqPHRruakfZ8nLUbg2gn/sz3hba2wHhM9IDFd5+Wl8JVrkCa
      r7DckWaIBNleVrAc7pB2HL7PjcKoMKfC6nH4UB06UP51tpCIpZ2zonke6toAKZFd
      iID/91ibyidnFBKC9aXMuGJ1wpOvPeqK/KO4p5PqXX7l9YfxGyebra3vsV5zt5c6
      oiOwslBv3RFQTj1yz1QVHthRO1L+enulezx4W6AzEhMcZEq4XKKG6Bu9ZOiOsvPf
      QUpB32JReIHqrzRT7o3sEIvlQ/ZnTPgjLn7hz6YFrNf2gysYLO856vS8LTT4c1FN
      DbA0KXcWr0SuJCegYMlyydgZJ7y5D3dXO/n3HZ+M8wwdt/KkNuHcM18Z4ekS8B9m
      b0dI7j7dB5CO3ozr/lU/mHrd7A05BIKLU+jWsvge7nUedUqqfohUFQm5L4kw7UGf
      zUydcUFCQv+VCPbMcXfa0VMCAwEAAQ==
      -----END PUBLIC KEY-----
authentication:
  - type: rsaSignatureAuthentication2018
    publicKey: did:wsutm:localhost-resource-authority-myra#keys-RSA
service:
```


### MyRA - The Docker Config File

Open `privacy-networks/testnet/myra/config/run_in_docker_config.yaml`

<div class='md-alert mb-12'>DO NOT COPY -- FOR REFRENCE ONLY</div>


This is the configuration `docker` will use when we provision `MyRA` to the network

```yaml
version: '1'
app_version: 1.0.0
description: Resource Authority - file paths assume Docker volumes
resource_authority:
  did: did:wsutm:localhost-resource-authority-myra
  public_key_id: did:wsutm:localhost-resource-authority-myra#keys-RSA
  private_key:
    type: file
    key_type: RSA
    value: '/keys/sign.rsa.key.pem'
block:
  block_type: Logger
shared_files_dir:
  description: repo has been mounted at /db using Docker volumes
  dd_dir: '/db/diddoc'
  resource_dir: '/db/resource'
  credential_dir: '/db/credential'
```

### MyRA - Provision to the Network

We will do this in a future step when we run the `buildAll.bash` script in the `testnet` directory

## 4. Provision the MyTa
TODO - add when create trust credentials

## 5. Privacy Domain - Resource Test Service
This is covered in [link](5-pn-pd-test-service.md)

## 6. Privacy Domain - Client
This is covered in [link](6-pn-pd-client.md)

## 7. Privacy Network - Run & Test
This is covered in [link](7-pn-run-test.md)

## 8. Adding Trust and testing
TODO

## 9. Install Trust Manager
TODO




