
docker-compose.yml文件：
```yaml
version: '3.9'

x-user: &user admin
x-pwd: &pwd superadmin123

x-nifi-base: &nifi-base
  image: apache/nifi:1.26.0
  networks:
    - nifi

x-nifi-deploy: &nifi-deploy
  deploy:
    resources:
      limits:
        cpus: '2'
        memory: 2048M
      reservations:
        cpus: '1'
        memory: 2048M

x-nifi-toolkit-base: &nifi-toolkit-base
  image: apache/nifi-toolkit:1.26.0
  networks:
    - nifi

x-nifi-environment: &nifi-environment
  NIFI_WEB_HTTPS_PORT: 8443
  NIFI_CLUSTER_IS_NODE: "true"
  NIFI_JVM_HEAP_INIT: 1024m
  NIFI_JVM_HEAP_MAX: 1024m
  NIFI_ZK_CONNECT_STRING: "zookeeper:2181"
  NIFI_ELECTION_MAX_WAIT: "30 sec"
  NIFI_ELECTION_MAX_CANDIDATES: 1
  NIFI_SENSITIVE_PROPS_KEY: "my-random-string"
  NIFI_CLUSTER_NODE_PROTOCOL_PORT: 8082
  NIFI_WEB_PROXY_HOST: "nifi0:8443,nifi0,nifi1:8443,nifi1,localhost:8080"
  SINGLE_USER_CREDENTIALS_USERNAME: *user
  SINGLE_USER_CREDENTIALS_PASSWORD: *pwd
  NIFI_SECURITY_USER_AUTHORIZER: "single-user-authorizer"
  NIFI_SECURITY_USER_LOGIN_IDENTITY_PROVIDER: "single-user-provider"
  INITIAL_ADMIN_IDENTITY: *user
  AUTH: "tls"
  KEYSTORE_TYPE: "JKS"
  KEYSTORE_PASSWORD: supersecretkeystore
  TRUSTSTORE_TYPE: "JKS"
  TRUSTSTORE_PASSWORD: supersecrettruststore

x-nifi0-environment: &nifi0-environment
  NIFI_CLUSTER_ADDRESS: "nifi0"
  NIFI_WEB_HTTPS_HOST: "nifi0"
  KEYSTORE_PATH: "/opt/certs/nifi0/keystore.jks"
  TRUSTSTORE_PATH: "/opt/certs/nifi0/truststore.jks"

x-nifi1-environment: &nifi1-environment
  NIFI_CLUSTER_ADDRESS: "nifi1"
  NIFI_WEB_HTTPS_HOST: "nifi1"
  KEYSTORE_PATH: "/opt/certs/nifi1/keystore.jks"
  TRUSTSTORE_PATH: "/opt/certs/nifi1/truststore.jks"

services:
  redis:
    hostname: redis
    container_name: redis
    image: arm64v8/redis:latest
    ports:
      - 6379
    networks:
      - nifi
    environment:
      - REDIS_PASSWORD=nifi_redis
  kafka:
    image: 'bitnami/kafka:latest'
    deploy:
      mode: replicated
      replicas: 3

    depends_on:
      - zookeeper
    ports:
      - 9092
    networks:
      - nifi
    environment:
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      # 开启自动创建 topic 功能便于测试
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      # 全局消息过期时间 6 小时(测试时可以设置短一点)
      - KAFKA_CFG_LOG_RETENTION_HOURS=6
  zookeeper:
    container_name: zookeeper
    image: bitnami/zookeeper:3.8.0
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
    networks:
      - nifi

  nifi-toolkit:
    <<: *nifi-toolkit-base
    container_name: nifi-toolkit
    volumes:
      - nifi_certs:/opt/certs
    user: root
    entrypoint: ["bash", "-c", "/opt/nifi-toolkit/*/bin/tls-toolkit.sh standalone -o /opt/certs -n nifi[0-1] -P supersecrettruststore -K supersecretkeystore -S supersecretkeystore; chown -R nifi:nifi /opt/certs"]

  proxy:
    image: arm64v8/nginx:latest
    container_name: proxy
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "8443:8443"
    networks:
      - nifi
    depends_on:
      - nifi0
      - nifi1

  nifi0:
    <<: [*nifi-base,*nifi-deploy]
    container_name: nifi0
    depends_on:
      nifi-toolkit:
        condition: service_completed_successfully
    volumes:
      - nifi_certs:/opt/certs
      - nifi0_conf:/opt/nifi/nifi-current/conf
      - nifi0_extensions:/opt/nifi/nifi-current/extensions
      - nifi0_database_repository:/opt/nifi/nifi-current/database_repository
      - nifi0_flowfile_repository:/opt/nifi/nifi-current/flowfile_repository
      - nifi0_content_repository:/opt/nifi/nifi-current/content_repository
      - nifi0_provenance_repository:/opt/nifi/nifi-current/provenance_repository
      - nifi0_state:/opt/nifi/nifi-current/state
      - nifi0_logs:/opt/nifi/nifi-current/logs
    environment:
      <<: [*nifi-environment,*nifi0-environment]
    networks:
      - nifi
    entrypoint:
      - "/bin/bash"
      - "-c"
      - "sed -i 's/nifi.ui.banner.text=.*/nifi.ui.banner.text=nifi0 (v1.26.0)/' conf/nifi.properties; ../scripts/start.sh"

  nifi1:
    <<: [*nifi-base,*nifi-deploy]
    container_name: nifi1
    depends_on:
      nifi-toolkit:
        condition: service_completed_successfully
    volumes:
      - nifi_certs:/opt/certs
      - nifi1_conf:/opt/nifi/nifi-current/conf
      - nifi1_extensions:/opt/nifi/nifi-current/extensions
      - nifi1_database_repository:/opt/nifi/nifi-current/database_repository
      - nifi1_flowfile_repository:/opt/nifi/nifi-current/flowfile_repository
      - nifi1_content_repository:/opt/nifi/nifi-current/content_repository
      - nifi1_provenance_repository:/opt/nifi/nifi-current/provenance_repository
      - nifi1_state:/opt/nifi/nifi-current/state
      - nifi1_logs:/opt/nifi/nifi-current/logs
    environment:
      <<:  [*nifi-environment,*nifi1-environment]
    networks:
      - nifi
    entrypoint:
      - "/bin/bash"
      - "-c"
      - "sed -i 's/nifi.ui.banner.text=.*/nifi.ui.banner.text=nifi1 (v1.26.0)/' conf/nifi.properties; ../scripts/start.sh"

networks:
  nifi:
    driver: bridge

volumes:
  nifi_certs:
  # nifi 0
  nifi0_conf:
  nifi0_extensions:
  nifi0_database_repository:
  nifi0_flowfile_repository:
  nifi0_content_repository:
  nifi0_provenance_repository:
  nifi0_state:
  nifi0_logs:
  # nifi 1
  nifi1_conf:
  nifi1_extensions:
  nifi1_database_repository:
  nifi1_flowfile_repository:
  nifi1_content_repository:
  nifi1_provenance_repository:
  nifi1_state:
  nifi1_logs:

```