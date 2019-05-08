# 5. Privacy Domain - Resource Test Service
This section `PN Enables` a 'Resouce Test Service' from docker image. This Resouce Test Service 

<br />

PN enabling a local service or data source involves
  1. Provisioning the service or data source into the networks management plane as a Privacy Domain. This gives it identity in the management plane and allows it to describe what it is prepared to share.
  2. If sharing a services/data sources/other http endpoints with the network then create the necessary privacy pipe endpoints, this are also part of the description in step (1).
  3. If sharing data by pushing it to another PN enabled application then create the necessary outbound privacy pipes
  4. If want to use a shared service in another domain then create the necessary outbound privacy pipes


## 5.1 Create docker-compose for domain and add the test service
Open `docker-compose.yaml` file in `service-pd` directory

<br />

 Make sure `services.test_service.image` is set to` webshielddev/resource-test-service:latest`

```yaml
version: '3'
services:
  test_service:
    image:  webshielddev/resource-test-service:latest
    container_name: test_service
    networks:
      - my-pn-network
    ports:
      - "49800:8080"
    logging:
      driver: json-file
      options:
        max-size: '1m'
        max-file: '3'
```

## 5.2 Configure the local resource authority service
This is only used to view resources not to author new resources hence NO DID is generated for this


### 5.2.1 TLS Keys - Generate

CD into `service-pd/rs/keys` directory

```bash
cd ~/webshield-dev/privacy-networks/testnet/service-pd/rs/keys
```

Run bash file to generate `RSA` keys

```bash
bash ../../../../tools/genkeys/generateRSA.bash
```

Since the certificate is not used hit the `Return-key` for all propts except **Common Name** which we will name `rs.pd.not.used`

```bash
# Enter this info
Common Name (eg, fully qualified host name) []:`rs.pd.not.used`
```

Rename all files with the prefix `tls`

```bash
cd ~/webshield-dev/privacy-networks/testnet/service-pd/rs/keys
mv rsa.cert.pem tls.rsa.cert.pem
mv rsa.key.pem tls.rsa.key.pem
mv rsa.public.key.pem tls.rsa.public.key.pem
```

Verify keys are changed by viewing tree chart of `/service-pd/rs/keys` directory

```bash {.copy-clip}
tree ~/webshield-dev/privacy-networks/testnet/service-pd/rs/keys
```

```bash {.copy-clip}
# Output

~/webshield-dev/privacy-networks/testnet/service-pd/rs/keys
├── tls.rsa.cert.pem
├── tls.rsa.key.pem
└── tls.rsa.public.key.pem
```

### 5.2.2 Resource Service - Docker Config
Add the `resource_authority.service` configuration to the `service-pd/config/run_in_docker_config.yaml` file

```yaml
...
resource_authority:
  service:
    listen_port: ':443'
    bearer_token_wl_path: /config/tms_ra_bearer_tokens_wl.yamls
    tls:
      enabled: true
      certificate:
        type: file
        value: /keys/tls.rsa.cert.pem
      private_key:
        type: file
        key_type: RSA
        value:  /keys/tls.rsa.key.pem
    cors_support:
      enabled: true
      allow_origin: '*'
      allow_headers: 'Origin, X-Requested-With, Content-Type, Accept'
...
```

### 5.2.3 Resource Service - Test Service - Docker Compose
Add the `services.test_service_rs` configuration to the `service-pd/docker-compose.yml`
```yaml
services:
...
  test_service_rs:
    image:  webshielddev/resource-authority:latest
    container_name: test_service_rs
    command: ["-service", "-config", "/config/run_in_docker_config.yaml"]
    networks:
      - my-pn-network
    volumes:
      - ./config:/config
      - ./rs/keys:/keys
      - ./copydb:/db
    ports:
      - "41000:443"
    logging:
      driver: json-file
      options:
        max-size: '1m'
        max-file: '3'
...
```

## 5.3 Trust Authority - Provision

This is used to query the management plan and make pipe decisions, the latter requires that it has a `DID`. Note if in same admin control this could have been shared across the domains.

### 5.3.1 TA Signing & TLS RSA Keys - Generate

Generate Keys

#### TA Signing Keys - Generate
CD into `service-pd/ta/keys` directory


```bash {.copy-clip}
cd ~/webshield-dev/privacy-networks/testnet/service-pd/ta/keys
```

Run bash script to generate `RSA` keys

```bash {.copy-clip}
bash ../../../../tools/genkeys/generateRSA.bash
```

Since the certificate is not used hit the `Return-key` for all propts except **Common Name** which we will name `ta.test_service.not.used`

```bash {.copy-clip}
# Enter this info
Common Name (eg, fully qualified host name) []:`ta.test_service.not.used`
```

Rename all files with the prefix `sign`

```bash {.copy-clip}
cd ~/webshield-dev/privacy-networks/testnet/service-pd/ta/keys
mv rsa.cert.pem sign.rsa.cert.pem
mv rsa.key.pem sign.rsa.key.pem
mv rsa.public.key.pem sign.rsa.public.key.pem
```

Verify keys are changed by viewing tree chart of `service-pd/ta/keys` directory

```bash
tree ~/webshield-dev/privacy-networks/testnet/service-pd/ta/keys
```

```bash
# Output

~/webshield-dev/privacy-networks/testnet/service-pd/ta/keys
├── sign.rsa.cert.pem
├── sign.rsa.key.pem
└── sign.rsa.public.key.pem
```

#### TLS RSA Keys - Generate
CD into `service-pd/ta/keys` directory


```bash {.copy-clip}
cd ~/webshield-dev/privacy-networks/testnet/service-pd/ta/keys
```

Run bash script to generate `TLS RSA` keys

```bash {.copy-clip}
bash ../../../../tools/genkeys/generateRSA.bash
```

Since the certificate is not used hit the `Return-key` for all propts except **Common Name** which we will name `tls.ta.test_service.not.used`

```bash {.copy-clip}
# Enter this info
Common Name (eg, fully qualified host name) []:`tls.ta.test_service.not.used`
```

Rename files with the prefix `tls`

```bash {.copy-clip}
cd ~/webshield-dev/privacy-networks/testnet/service-pd/ta/keys
mv rsa.cert.pem tls.rsa.cert.pem
mv rsa.key.pem tls.rsa.key.pem 
mv rsa.public.key.pem tls.rsa.public.key.pem
```

Verify keys are changed by viewing tree chart of `service-pd/ta/keys` directory

```bash
tree ~/webshield-dev/privacy-networks/testnet/service-pd/ta/keys
```

```bash
# Output

~/webshield-dev/privacy-networks/testnet/service-pd/ta/keys
├── sign.rsa.cert.pem
├── sign.rsa.key.pem
├── sign.rsa.public.key.pem
├── tls.rsa.cert.pem
├── tls.rsa.key.pem
└── tls.rsa.public.key.pem
```

## 5.3.2 Trust Authority - DID & DID Doc
Remove existing `ta_diddoc.yaml` file

```bash {.copy-clip}
rm -f ~/webshield-dev/privacy-networks/testnet/service-pd/ta/init_template/ta_diddoc.yaml
```

Copy `diddoc_template.yaml` template to the `service-pd/ta/init_template` directory and name it `ta_diddoc.yaml`

```bash
cp ~/webshield-dev/privacy-networks/templates/diddoc/diddoc_template.yaml \
~/webshield-dev/privacy-networks/testnet/service-pd/ta/init_template/ta_diddoc.yaml
```

Open the file `service-pd/ta/init_template/ta_diddoc.yaml` and replace values where indicated

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
did:wsutm:localhost-trust-authority-test-service
```

#### Replace RSA Public Key
Copy the `sign.rsa.public.key` to the clipboard

```bash
pbcopy < ~/webshield-dev/privacy-networks/testnet/service-pd/ta/keys/sign.rsa.public.key.pem
```

If you do not have `pbcopy` installed use `cat` to copy to the `Terminal` output

```bash
cat ~/webshield-dev/privacy-networks/testnet/service-pd/ta/keys/sign.rsa.public.key.pem
```

Replace here

```yaml
    publicKeyPem: |
      -----BEGIN PUBLIC KEY-----
        REPLACE WITH SIGNING RSA PUBLIC KEY
      -----END PUBLIC KEY-----
```
Save file

#### Trust Authority - The DID Doc

<div class='md-alert mb-12'>DO NOT COPY -- FOR REFRENCE ONLY</div>

Your new `ta_diddoc.yaml` should look something like this when you are done

```yaml 
context: https://w3id.org/did/v1
id: did:wsutm:localhost-trust-authority-test-service
name: DID Doc Local Trust Authority
comment: using for local machine
publicKey:
  - id: did:wsutm:localhost-trust-authority-test-service#keys-RSA
    name: a RSA key
    publicKeyPem: |
      -----BEGIN PUBLIC KEY-----
      MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAsgc6JXFMz4KTF3ED0D96
      5/pZla6Zttkzr2J2NUEBFY1m+n5WjVmBWBzpsi0QE1kEbaVz9+/1W/jlWZGNQrlR
      8xu1BbTbXESybwtoaqQMVk43zBHOmtTCK1G1Jwlm4XtLG7tqV6bvOQbACn4j5CXa
      tQ3OoJwMMDW8quZsORlyE/rOdctYaB1r+gtpcm7BokajJm7Qj+BgnJN7v1SkX7Jn
      Rs34GFkpUnhqhVmJwyCKBerlH6kZZx0qhDUdmJj92RJPJMgSVqq69KrmpiHjj3R0
      sfVPODeoHEIn0uP/9Ifkmx/8gVFBK7PKxaZEU/t7mu07wzxTlGPpGovUgn3lJjMh
      FLuGUE5vSypK4LztDykApZFKpfNY03lf4R9eJPnobFaaSQqmi9deRzyZnzsvaDuL
      QV56487e9OCKeEKga2NosreiDRIHNKafUfl5v7RqwBV9o0GW3bnqH368MO18MY2p
      4L51PO+54bjd16DhU0XLOAt5bX3KxZQYAGNx28ZYE7CeVa77Jjx3gk6uLUlngblh
      XlTHDXMXZUmmBhZi1ocLNSZadOT0FOsGLJ5zW8Vl89Q6qRb4Ier2ahQnw2vW4trv
      TRoDz2AP2/qS4WcsyikD7bjaNt2QdS/jvJwo44wK6apw9t/KxpF3no+YzPs4dfo0
      u474VlvSHE07IW0hTQiM7gMCAwEAAQ==
      -----END PUBLIC KEY-----
authentication:
  - type: rsaSignatureAuthentication2018
    publicKey: did:wsutm:localhost-trust-authority-test-service#keys-RSA
service:
```

### 5.3.3 Add the local TA configuration to the shared config

Open the `service-pd/config/run_in_docker_config.yaml` file

<div class='md-alert mb-12'>DO NOT COPY -- FOR REFRENCE ONLY</div>

Under the `trust_model_services` section you should see the `Trust Authority` congiguration for the `test_service` 

```yaml
...
trust_model_service:
  did: did:wsutm:localhost-trust-authority-test-service
  public_key_id: did:wsutm:localhost-trust-authority-test-service#keys-RSA
  private_key:
    type: file
    key_type: RSA
    value: /keys/sign.rsa.key.pem
  listen_port: ':443'
  base_address: '/tms'
  bearer_token_wl_path: /config/ta_bearer_tokens_wl.yaml
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
...
```

### 5.3.4 Provision the TA

> This will be done in a future step when we run the `buildAll.bash` script from the `testnet` directory

## 5.4 Privacy Domain - Provision for the Resouce Test Service

The test service shares two http endpoints (user and query) to the network that are designed to manage/query user records and in its response's to return the user info and all inbound request headers and body. The latter can be used to see what info the privacy agent actually passed to the service.

### 5.4.1 Privacy Domain - Generate RSA Signing Key

CD into `service-pd/pd-artifacts/keys` directory

```bash
cd ~/webshield-dev/privacy-networks/testnet/service-pd/pd-artifacts/keys
```

Run bash script to genereate `RSA` sigining key

```bash
bash ../../../../tools/genkeys/generateRSA.bash
```

The certificate is not used so hit the `Return-key` for all prompts except **Common Name** which we will call `test-service.pd.not.used`

```bash
# Enter this info
Common Name (eg, fully qualified host name) []:`test-service.pd.not.used`
```

Rename keys with prifix `sign`

```bash
cd ~/webshield-dev/privacy-networks/testnet/service-pd/pd-artifacts/keys
mv rsa.cert.pem sign.rsa.cert.pem
mv rsa.key.pem sign.rsa.key.pem
mv rsa.public.key.pem sign.rsa.public.key.pem
```

### 5.4.2 Privacy Domain - DID & DID Doc

Remove existing `test_service_pd_diddoc.yaml` file

```bash {.copy-clip}
rm -f ~/webshield-dev/privacy-networks/testnet/service-pd/pd-artifacts/test_service_pd_diddoc.yaml
```

Copy `diddoc_template.yaml` template to the `service-pd/pd-artifacts` directory and name it `test_service_pd_diddoc.yaml`

```bash
cp ~/webshield-dev/privacy-networks/templates/diddoc/diddoc_template.yaml \
~/webshield-dev/privacy-networks/testnet/service-pd/pd-artifacts/test_service_pd_diddoc.yaml
```

Open the file `service-pd/pd-artifacts/test_service_pd_diddoc.yaml` and replace values where indicated

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
did:wsutm:localhost-resource-pd-test-service
```

#### Replace RSA Public Key
Copy the `sign.rsa.public.key` to the clipboard

```bash
pbcopy < ~/webshield-dev/privacy-networks/testnet/service-pd/pd-artifacts/keys/sign.rsa.public.key.pem
```

If you do not have `pbcopy` installed use `cat` to copy to the `Terminal` output

```bash
cat ~/webshield-dev/privacy-networks/testnet/service-pd/pd-artifacts/keys/sign.rsa.public.key.pem
```

Replace here

```yaml
    publicKeyPem: |
      -----BEGIN PUBLIC KEY-----
        REPLACE WITH SIGNING RSA PUBLIC KEY
      -----END PUBLIC KEY-----
```
Save file

#### Privacy Domain - The DID Doc

<div class='md-alert mb-12'>DO NOT COPY -- FOR REFRENCE ONLY</div>

Your new `test_service_pd_diddoc.yaml` should look something like this when you are done

```yaml
context: https://w3id.org/did/v1
id: did:wsutm:localhost-resource-pd-test-service
name: Privacy Domain for the Resource Test Service
comment: using for local machine
publicKey:
  - id: did:wsutm:localhost-resource-pd-test-service#keys-RSA
    name: a RSA key
    publicKeyPem: |
      -----BEGIN PUBLIC KEY-----
      MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEA4qTAyENEIBihHxe9io0/
      YXG34kduWgJQvXMsWCKmCkTYdpwO1aYwgewFRlY39BsV1QPzxRgBjN1Jm5FtPwma
      REUlRo/CS8J6KHHT6xUfnXdZNThB2rP0jC49hmNfICg6mNYxapKhDL3QJ88w4tpM
      ngynyg4/OefJ0WFMTnbRNVcA2sAKOoq8ReZW47oW6TQ03Z3yP7Ih3ry4fiUtYiOL
      dK1hES62yTbo+yvbaC+9QNTJPgucgZPWDuyywZZDp8j7zGiAYuPIqmA9Yl+4b+yu
      oWipDn9s2q32kwnsKogSa2Oi1yGGC9xuHuO741d5+5aO4Nun0b/KgCy7MtR+FsV+
      sCDvF8RG2XWL0r8sla01/1cH1MQ+hXnniM5rLi8oyWW5XbGDxwgNt3tz6m4emqHI
      AwSMJxF6kKAE/8chR1TfyLWuVMNThlcC9QP+YqKxrbnmr6YC/MJrQYqnLF4UnM8O
      AhI+Rd9zo+rROPHVAUZpaQTBJQAxCW6bQrWVX0SijZMrhKw4YKSUpYOzJmG3zmYG
      tNY8dKxsLzmu+NssP2lQgA4SoK2dmIcZZzo1bKV2IwqBwrTtdGEz4lQAdSr3b/0V
      byJJoHD2qVcqfZfS3dogw45NILHI0SGkcKhPr+m1kBY6cZApYp9a5tLVDsulnDUN
      7wQrl7iN0esEOcwdStoycb0CAwEAAQ==
      -----END PUBLIC KEY-----
authentication:
  - type: rsaSignatureAuthentication2018
    publicKey: did:wsutm:localhost-resource-pd-test-service#keys-RSA
service:
```

### 5.4.3 Privacy Domain - Definition
Create the `Privacy Domain` definition that describes the pipe protected http endpoints that are published to the network

<br />

The file is created in `service-pd/pd-artifacts/test_service_pddef_docker_endpoints.yaml` and contains the content shown below, some notes

- The endpoints assume a docker network of `my-pn-network` running a privacy agent named `test_service_pa`
- For each endpoint a `pong` endpoint is also defined -- only needed if want to support `ping`

```yaml
version: '1'
description: Describes the test service and its http endpoints
privacy_domain_def:
    context: dummy context
    id: 'provision-will-auto-set-using-did'
    type: PrivacyDomain
    name: test-service
    services:
      - id: user
        description: Create user privacy pipe endpoint
        name: user
        type:
          - Service
        service_endpoint: 'https://test_service_pa:443/test_service/user'
        pong_endpoint: 'https://test_service_pa:443/pa/v1/agent_2_agent/privacy_pipe/user/pong'
        openapi_definition:
          openapi: 3.0
          paths:
            '/':
              get:
                operationid: get_user
                request_body:
                  content:
                    'application/json':
              delete:
                operationid: delete_user
                request_body:
                  content:
                    'application/json':
              patch:
                operationid: patch_user
                request_body:
                  content:
                    'application/json':
              post:
                operationid: post_user
                request_body:
                  content:
                    'application/json':
              put:
                operationid: put_user
                request_body:
                  content:
                    'application/json':
      - id: query
        description: Create the query user privacy pipe endpoint
        name: query
        type:
          - Service
        service_endpoint: 'https://test_service_pa:443/test_service/query'
        pong_endpoint: 'https://test_service_pa:443/pa/v1/agent_2_agent/privacy_pipe/query/pong'
        openapi_definition:
          openapi: 3.0
          paths:
            '/':
              get:
                operationid: query_user
                request_body:
                  content:
                    'application/json':
```

### 5.4.4 Privacy Domain - Provision

This will be done in a future step when we run the `buildAll.bash` script from the `testnet` directory

## 5.5 Privacy Agent - Config - No Pipes

### 5.5.1 Generate TLS certs/keys for agent-2-agent and adapter-2-agent ports

#### Agent-2-Agent TLS 

CD into `service-pd/pa/keys` directory

```bash
cd ~/webshield-dev/privacy-networks/testnet/service-pd/pa/keys
```

Run bash script to generate `RSA` keys

```bash
bash ../../../../tools/genkeys/generateRSA.bash
```

The certificate is not used so hit return for all prompts except **Common Name** which we wil call `tls.agent.test_service.not.used`

```bash
# Enter this info
Common Name (eg, fully qualified host name) []:`tls.agent.test_service.not.used`
```

Rename files with the prefix `tls.agent`

```bash
cd ~/webshield-dev/privacy-networks/testnet/service-pd/pa/keys
mv rsa.cert.pem tls.agent.rsa.cert.pem
mv rsa.key.pem tls.agent.rsa.key.pem
mv rsa.public.key.pem tls.agent.rsa.public.key.pem
```

#### Adapter-2-Agent TLS

CD into `service-pd/pa/keys` directory

```bash
cd ~/webshield-dev/privacy-networks/testnet/service-pd/pa/keys
```

Run bash script to generate `RSA` keys

```bash
bash ../../../../tools/genkeys/generateRSA.bash
```

The certificate is not used so hit return for all prompts except **Common Name** which we wil call `tls.adapter.test_service.not.used`

```bash
# Enter this info
Common Name (eg, fully qualified host name) []:`tls.adapter.test_service.not.used`
```

Rename files with the prefix `tls.adapter`

```bash
cd ~/webshield-dev/privacy-networks/testnet/service-pd/pa/keys
mv rsa.cert.pem tls.adapter.rsa.cert.pem
mv rsa.key.pem tls.adapter.rsa.key.pem
mv rsa.public.key.pem tls.adpater.rsa.public.key.pem
```


### 5.5.2 Privacy Agent - Config - No Pipes
Open `service_pd/config/run_in_docker_config.yaml` file

<div class='md-alert mb-12'>DO NOT COPY -- FOR REFRENCE ONLY</div>

Under the  `privacy_agent` property is the configuration for the `service-pd` Privacy Agent

```yaml
...
privacy_agent:
  description: service-pd privacy agent
  myprivacy_domain:
    pdid: did:wsutm:localhost-resource-pd-test-service
    public_key_id: did:wsutm:localhost-resource-pd-test-service#keys-RSA
    private_key:
      type: file
      key_type: RSA
      value: /pd/keys/sign.rsa.key.pem
  local_tms_endpoint:
    description: the ta running as a service that should use for pipe authorization decision
    authorize: 'https://test_service_ta:443/tms/v1/authorize'
  cors_support:
    description: the agent exposes a graphQL query endpoint for trust manager
    enabled: true
    allow_origin: '*'
    allow_headers: 'Origin, X-Requested-With, Content-Type, Accept'
  pipe_md:
    description: applies to all pipes unless override at pipe level
    default_new_tb:
      description: defaults when create a new trust block
      verify_credential: true
      verify_criteria: true
      pd_criteria_id:
        graph:
      data_criteria_id:
        graph:
      data_credential_id:
        graph:
    default_inbound_tb:
      description: critiera that restrict what in-bound data is accepted
      verify_credential: true
      pd_criteria_id:
        graph:
    default_http:
      description: configures all pipe related http connections
      agent_2_agent:
        description: agent-2-agent config
        client:
          description: always connect using TLS
          tls:
            enabled: true
        server:
          description: provide a TLS endpoint - no mutual auth
          listen_port: ':443'
          tls:
            enabled: true
            certificate:
              type: file
              value: '/keys/tls.agent.rsa.cert.pem'
            private_key:
              type: file
              key_type: RSA
              value: '/keys/tls.agent.rsa.key.pem'
      adapter_2_agent:
        description: local adapter or service to agent - no mutual auth
        listen_port: ':444'
        tls:
          enabled: true
          certificate:
            type: file
            value: '/keys/tls.adapter.rsa.cert.pem'
          private_key:
            type: file
            key_type: RSA
            value: '/keys/tls.adapter.rsa.key.pem'
...
```

### 5.5.3 Privacy Domains - Privacy Agent - Docker Compose

Open the `service-pd/docker-compose.yml`

<div class='md-alert mb-12'>DO NOT COPY -- FOR REFRENCE ONLY</div>

Under `services.test_services_pa` you can see that the `Privacy Agent` has been added to the `docker-compose` file -- We will check its status once the `test_service_pa` is loaded 

```bash
services:
...
  test_service_pa:
    image:  webshielddev/privacy-agent:latest
    container_name: test_service_pa
    command: ["-config", "/config/run_in_docker_config.yaml"]
    networks:
      - my-pn-network
    volumes:
      - ./config:/config
      - ./pd-artifacts:/pd
      - ./pa/keys:/keys
      - ./copydb:/db
    ports:
      - "41005:443"
      - "41006:444"
    logging:
      driver: json-file
      options:
        max-size: '1m'
        max-file: '3'
...
```

## 5.6 Privacy Pipe - Privacy Agent - Configure Endpoints
Under the `privacy_agent` section of `service_pd/config/run_in_docker_config.yaml` we can see the pipe endpoints and that they are connected to the local `test_service` endpoints

<div class='md-alert mb-12'>DO NOT COPY -- FOR REFRENCE ONLY</div>

The Privacy Agent status should show 2 endpoints when we load the `privacy_agent` -- We will check its status once the `privacy_agent` is loaded
```yaml
privacy_agent:
  ...
  pipe_endpoints:
    user:
      name: user
      description: route onto the local test service user
      input:
        pipe_application_url:
          - /user
        verify_tb_config_id: default
        new_tb_config_id: default
      pd_service:
        description: connect to local docker instance of test service
        service_name: test_service_user
        local_endpoint: 'http://test_service:8080'
        verb_2_path:
          delete: /user/:id
          get: /user/:id
          post: /user
          put: /user/:id
          patch: /user/:id
    query:
      name: query
      description: route onto the local test service query
      input:
        pipe_application_url:
          - /query
        verify_tb_config_id: default
        new_tb_config_id: default
      pd_service:
        description: connect to local docker instance of test service
        service_name: testrs_query
        local_endpoint: 'http://test_service:8080'
        verb_2_path:
...
```

## Next Step

[6. Privacy Domain - Client](6-pn-pd-client.md)