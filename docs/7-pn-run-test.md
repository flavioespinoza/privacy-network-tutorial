# 7. Privacy Network - Run & Test
Here we will generate all metadata, provision all services and authoriites, and run the local privacy network

## Build All
CD into `testnet` directory
```bash
cd ~/webshield-dev/privacy-networks/testnet
```

Run `buildAll.bash` script to generate all metadata and provision authorities
```bash
bash buildAll.bash
```

## Privacy Domain - Resource Test services

CD into `service-pd` directory
```bash
cd ~/webshield-dev/privacy-networks/testnet/service-pd
```

Run privacy domain for the PN enabled test service
```bash
docker-compose up
```

Open new `Terminal` shell and check status
```bash
curl -k https://localhost:41000/ra/v1/status
```
```bash
# Output expected
{"service_name":"Resource-Authority","status":200,"version":"2.2.0","ra_did":"","config_description":"","config_app_version":"","config_file_description":"Test Service PD services  - file paths assume Docker volumes","config_file_app_version":"1.0.0"}
```

## Privacy Domain - Client

CD into `client-pd` directory
```bash
cd ~/webshield-dev/privacy-networks/testnet/client-pd
```

Run privacy domain for client
```bash
docker-compose up
```

Open new Terminal shell and check status
```bash
curl -k https://localhost:41006/pa/v1/status
```
```bash
# Output expected
{"service_name":"Privacy-Agent","status":200,"version":"2.39.0","outbound_pipes":0,"pipe_endpoints":2,"pd_name":"test-service","pdid":"did:wsutm:localhost-resource-pd-test-service","config_description":"Test Service PD services  - file paths assume Docker volumes","adapter_2_agent_port":":444","agent_2_agent_port":":443"}
```

## Privacy Domain - Client - Test Outbound Pipes

- Ping the `client-pd` outbound pipes to verify everything was set up correctly

Ping `user` pipe
```bash
curl -k https://localhost:42006/pa/v1/privacy_pipe/outbound/user/ping
```
```bash
# Output expected
{"status":"200"}
```
Ping `query` pipe
```bash
curl -k https://localhost:42006/pa/v1/privacy_pipe/outbound/query/ping
```
```bash
# Output expected
{"status":"200"}
```

## Privacy Network - View the Awesome

```bash
docker container ls --format 'table {{.ID}}\t{{.Networks}}\t{{.Names}}\t{{.Status}}' 
```
```bash
# Output
CONTAINER ID        NETWORKS            NAMES               STATUS
8c319ce2578d        my-pn-network       client_pa           Up About an hour
aef0a6846e1f        my-pn-network       client_ta           Up About an hour
4d6752c74589        my-pn-network       test_service        Up About an hour
8a021fdf60f2        my-pn-network       test_service_ta     Up About an hour
f6e60755804a        my-pn-network       test_service_pa     Up About an hour
541832068cae        my-pn-network       test_service_rs     Up About an hour
```
