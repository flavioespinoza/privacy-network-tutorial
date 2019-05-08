# 6. Privacy Domain - Client

This section creates a privacy domain with outbound pipes that are connected to the PN enabled test service pipe endpoints setup in the previous section.

<br />

**Steps** The minimum steps
  1. Create docker-compose for domain
  2. Configure the local resource service docker application - could be shared across domains in same admin
  3. Provision and configure the local trust authority used to provision pipes - could be shared across domains in same admin
  4. Provision the privacy domain for the test service
  5. Configure the privacy agent
  6. Configure the pipe endpoints in the agent [NONE]
  7. Configure the outbound pipes in the agent
  8. Start services inside the domain and test locally

## 6.1 Create docker-compose for domain

## 6.2  Configure the local resource service
TODO

## 6.3 Setup TA
This is used to query the management plan and make pipe decisions, the latter requires that it has a DID. Note if in same admin control this could have been shared across the domains.

Steps are
  1. Generate the signing and TLS keys
  2. Create the did and diddoc - `did:wsutm:localhost-trust-authority-client`
  3. Add service config to the shared config file - 'client/config/run_in_docker_config.yaml'
  4. provision test service TA into the network
  5. Add to domains docker-compose 'client/docker-compose.yaml'

### 6.3.1 Generate the local TA signing and TLS keys

#### 6.3.1.1 Generate the signing RSA key
From the 'client/ta/keys' directory run

  `bash ../../../../tools/genkeys/generateRSA.bash`

The certificate is not used so hit return for all prompts except Common Name *ta.client.not.used*

Rename the files so prefixed with tls
  `mv rsa.cert.pem sign.rsa.cert.pem; mv rsa.key.pem sign.rsa.key.pem; mv rsa.public.key.pem sign.rsa.public.key.pem`

#### 6.3.1.2 Generate the TLS RSA key
From the 'client/ta/keys' directory run

`bash ../../../../tools/genkeys/generateRSA.bash`

The certificate is not used so hit return for all prompts except Common Name *tls.ta.client.not.used*

Rename the files so prefixed with tls
  `mv rsa.cert.pem tls.rsa.cert.pem; mv rsa.key.pem tls.rsa.key.pem; mv rsa.public.key.pem tls.rsa.public.key.pem`


###  6.3.2 Create the local TA did and diddoc

1. The local TA DID is: `did:wsutm:localhost-trust-authority-client_ta`

2. Create MyTA DID document
  - Copy did document template to `client/ta/init_template/ta-diddoc.yaml`
    - Set the DID to `did:wsutm:localhost-trust-authority-client` in the template
    - Copy the contents of 'client/ta/keys/sign.rsa.public.key.pem' into the template

### 6.3.3 Add the local TA configuration to the shared config
Add the following to the shared config in  'client/config/run_in_docker_config.yaml'

Notes
  - Set the DID for id and key
  - added cors support as needed by local trust manager for viewing
  - added bearer_token_wl_path as used to protect local tms endpoints to only pa can call

```yaml
trust_model_service:
  did: did:wsutm:localhost-trust-authority-client
  public_key_id: did:wsutm:localhost-trust-authority-client#keys-RSA
  private_key:
    type: file
    key_type: RSA
    value: /keys/sign.rsa.key.pem
  listen_port: ':443'
  base_address: '/tms'
  bearer_token_wl_path: /config/tms_bearer_tokens_wl.yaml
  tls:
    enabled: true
    certificate:
      type: file
      value: /keys/tls.rsa.cert.pem
    private_key:
      type: file
      key_type: RSA
      value: /keys/tls.rsa.key.pem
  cors_support:
    enabled: true
    allow_origin: '*'
    allow_headers: 'Origin, X-Requested-With, Content-Type, Accept'
```

### 6.3.4 Provision the TA
The following command run from the dir 'testnet' provisions the local TA into the network, see regenMetadata.bash

```bash
if docker run \
  --mount type=bind,source="$(pwd)/client/config",destination="/config" \
  --mount type=bind,source="$(pwd)/client/ta/keys",destination="/keys" \
  --mount type=bind,source="$(pwd)/db",destination="/db" \
  --mount type=bind,source="$(pwd)/client/ta",destination="/ta" \
  webshielddev/trust-authority \
  -init \
  -config /config/run_in_docker_config.yaml \
  -ddt /ta/init_template/diddoc_template.yaml; then
  echo "Client TA Initialzed - Successful"
else
  echo "*****Client TA Initialzed - FAILED *****"
  exit 1
fi
```


## 6.4 Provision the client privacy domain
Create outbound pipes for user and query that are connected to test service pipe endpoints

Steps are
  1. Create the domain's signing key
  2. Create the domain's did and diddoc
  3. Create the test resource server privacy domain description that describes the pipe protected http endpoints
  4. Use MyRA to provision the test resource server privacy domain into the network

### 6.4.1 Generate the privacy domain signing key
From the 'client-pd/pd-artifacts/keys' directory run

  `bash ../../../../tools/genkeys/generateRSA.bash`

The certificate is not used so hit return for all prompts except Common Name *client.pd.not.used*

Rename the files so prefixed with tls
  `mv rsa.cert.pem sign.rsa.cert.pem; mv rsa.key.pem sign.rsa.key.pem; mv rsa.public.key.pem sign.rsa.public.key.pem`


### 6.4.2 Create the privacy domain DID and DID Document

1. The privacy domain DID is: `did:wsutm:localhost-resource-pd-client`

2. Create the privacy domain PD DID Document
  - Copy did document template to 'client-pd/pd-artifacts/client_pd_diddoc.yaml' and change
    - Set the DID to `did:wsutm:localhost-resource-pd-client` in the template
    - Copy the contents of 'pd-artifacts/keys/sign.rsa.public.key.pem' into the template

### 6.4.3 Create the privacy domain definition
Create the privacy domain definition - there are NO endpoints shared

The file is created in `client-pd/pd-artifacts/client_pddef_docker_endpoints.yaml` and contains the content shown below, some notes

```yaml
version: '1'
description: Describes the client - there are NO endpoints shared
privacy_domain_def:
    context: dummy context
    id: 'provision-will-auto-set-using-did'
    type: PrivacyDomain
    name: client-of-test-service
    services:
```

### 6.4.4 Use MyRA to provision the privacy domain
Acting as MyRA provision the service privacy domain into the network

Notes
 - flags
    - "-issuePD" tells RA it is issuing a domain
    - "-ddt" the did doc of the domain
    - "-rdt" the privacy domain definition

```bash
if docker run \
  --mount type=bind,source="$(pwd)/myra/config",destination="/config" \
  --mount type=bind,source="$(pwd)/myra/keys",destination="/keys" \
  --mount type=bind,source="$(pwd)/db",destination="/db" \
  --mount type=bind,source="$(pwd)/client-pd/pd-artifacts",destination="/pd-artifacts" \
  webshielddev/resource-authority \
  -issuePD \
  -config /config/run_in_docker_config.yaml \
  -ddt /pd-artifacts/client_pd_diddoc.yaml \
  -rdt ./pd-artifacts/client_pddef_docker_endpoints.yaml; then
  echo "myRA provision Client PD Resource - Successful"
else
  echo "***** myRA provision Client PD Resource - FAILED *****"
  exit 1
fi
```
# 6.5 Configure the privacy agent - no pipes
Steps are
 1. Generate TLS certs/keys for agent-2-agent and adapter-2-agent ports
 2. Add privacy agent config to common config file - NO pipes
 3. Add privacy agent to privacy domains docker-compose

### 6.5.1 Generate TLS certs/keys for agent-2-agent and adapter-2-agent ports

1. Agent-2-Agent TLS - From the 'client-pd/pa/keys' directory run

 `bash ../../../../tools/genkeys/generateRSA.bash`

The certificate is not used so hit return for all prompts except Common Name *tls.agent.client.not.used*

Rename the files so prefixed with tls
 `mv rsa.cert.pem tls.agent.rsa.cert.pem; mv rsa.key.pem tls.agent.rsa.key.pem; mv rsa.public.key.pem tls.agent.rsa.public.key.pem`

 2. Adapter-2-Agent TLS - From the 'client-pd/pa/keys' directory run

  `bash ../../../../tools/genkeys/generateRSA.bash`

 The certificate is not used so hit return for all prompts except Common Name *tls.adapter.client.not.used*

 Rename the files so prefixed with tls
  `mv rsa.cert.pem tls.adapter.rsa.cert.pem; mv rsa.key.pem tls.adapter.rsa.key.pem; mv rsa.public.key.pem tls.adpater.rsa.public.key.pem`

### 6.5.2 Create privacy agent config - no pipes
The 'templates/skeleton_privacy_agent_docker_config.yaml' file contains a privacy agent configuration that does not specify any pipes copy "privacy_agent" into the privacy domains shared config file 'client_pd/config/run_in_docker_config.yaml'

Perform the following updates
  - Set "myprivacy_domain.pdid: did:wsutm:localhost-resource-pd-client"
  - Set "public_key_id: did:wsutm:localhost-resource-pd-client#keys-RSA"
  - Set the private key to the signing key in 'pd-artifacts/keys' - using Docker volumes '/pd/keys/sign.rsa.key.pem'
  - Set default_http.agent_2_agent TLS cert/key to '/keys/tls.agent.rsa....'
  - Set default_http.adapter_2_agent TLS cert/key to '/keys/tls.adapter.rsa....'

### 6.5.3 Add privacy agent to domains docker-compose
The following needs to be added, once started the PA status can be checked with  `curl -k https://localhost:42006/pa/v1/status`

```yaml
client_pa:
  image:  webshielddev/privacy-agent:latest
  container_name: client_pa
  command: ["-config", "/config/run_in_docker_config.yaml"]
  networks:
    - my-pn-network
  volumes:
    - ./config:/config
    - ./pd-artifacts:/pd
    - ./pa/keys:/keys
    - ./copydb:/db
  ports:
    - "42005:443"
    - "42006:444"
  logging:
    driver: json-file
    options:
      max-size: '1m'
      max-file: '3'
```

## 6.6 Configure the outbound privacy pipe in the agent
In the privacy agent section of 'client_pd/config/run_in_docker_config.yaml' going to add the outbound pipe endpoints and connect test service endpoints

Once configure the PA status should show 2 outbound pipes,  `curl -k https://localhost:42006/pa/v1/status`

```yaml
outbound_pipes:
  user:
    name: user
    description: User outbound pipe
    input:
      pipe_application_url:
        - /user
        - /user/:id
    destination_pd_endpoint:
      description: test service pd
      pdid: did:wsutm:localhost-resource-pd-test-service
      service: user
      new_tb_config_id: default
      verify_tb_config: default
  query:
    name: query
    description: Query outbound pipe
    input:
      pipe_application_url:
        - /query
    destination_pd_endpoint:
      description: test service pd
      pdid: did:wsutm:localhost-resource-pd-test-service
      service: query
      new_tb_config_id: default
      in_tb_config: default

```