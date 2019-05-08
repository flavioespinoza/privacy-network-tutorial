# Privacy Network - Testnet Tutorial Directory
> We will setup a new `testnet-tutorial` directory where you will setup and run your local privacy network

- The `testnet-tutorial` directory is ignored via the `.gitignore` file and will not be pushed to `GitHub`
- We will use the folder structure from `testnet` as the scaffolding for a new `testnet-tutorial` directory
- This will duplicate the `folders only` and `not the files`

## Testnet Tutorial Directory - Create
Create the destination directory `testnet-tutorial`
```bash
mkdir ~/webshield-dev/privacy-networks/testnet-tutorial
```

CD into the source directory `testnet`
```bash
cd ~/webshield-dev/privacy-networks/testnet
```

Reference the folder structure in the `testnet` directory and duplicate it the `testnet-tutorial` directory
```bash
find . -type d | cpio -pdvm ~/webshield-dev/privacy-networks/testnet-tutorial
```

## Testnet Tutorial Directory - Verify
Verify folder structure was duplicated in the testnet-tutorial directory
```bash
tree ~/webshield-dev/privacy-networks/testnet-tutorial
```

```shell
privacy-networks/testnet-tutorial/
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

## Testnet Tutorial Directory - Remove
> After you have finished this tutorial you can remove the `testnet-tutorial` directory

Remove the `testnet-tutorial` directory after you have finished this tutorial

```bash
sudo rm -r ~/webshield-dev/privacy-networks/testnet-tutorial/
```
