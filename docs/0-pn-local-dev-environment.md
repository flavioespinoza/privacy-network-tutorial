# Privacy Network - Local Dev Environment

`Privacy Networks` and the` Universal Trust Model` enable individuals and organizations to easily pool, share, transact, and re-use their most sensitive, regulated and proprietary resources, and transform them into precision personalized services and process optimization networks, whilst ensuring the necessary privacy, compliance, and business policies are enforced.

## Docker Network

This project uses `docker-compose` and requires a `docker network`

### Create Docker Network

Create a new `docker network` named `my-pn-network`

```bash
cd ~
docker network create my-pn-network
```

### Inspect Docker Network

View all containers running on the `my-pn-network`

```bash
docker network inspect my-pn-network
```

### Inspect Docker Containers

Verify your `my-pn-network` is setup and running

```bash
docker network ls
```

You should see `my-pn-network` in the list

```shell
 NETWORK ID          NAME                DRIVER              SCOPE
 49b65ef10b02        bridge              bridge              local
`d0c1bbe049d0        my-pn-network       bridge              local` # <-- You should see this line
 6996893549d5        none                null                local
```

## Local WebShield Dev Folder

> You will need to setup a specific directory to configure and run a `Privacy Network` on your local dev environment

### Setup Local WebShield Dev Directory

 CD into your `root directory` and create a `webshield-dev` directory
 
```bash
cd ~
mkdir webshield-dev
```

### Clone GitHub repository

CD into the `webshield-dev` directory and clone the `privacy-network` git repository

```bash
cd ~/webshield-dev
git clone https://github.com/webshield-dev/privacy-networks.git
```

## Next Step

- [1. Privacy Network - MyRA Resource Authority](1-pn-myra.md)
