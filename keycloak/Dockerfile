# ARG defined before FROM in multi-staged Dockerfile is shared among the stages
ARG KEYCLOAK_VERSION=26.8.2

# Build the provider
FROM maven:3.9.13-sapmachine-17 AS keycloak-pii-data-encryption
ARG KEYCLOAK_VERSION # Dockerfile peculiarity that requires ARG defined before FROM to be re-declared afterwards if we want to use it in the stage
WORKDIR /app
RUN apt-get update && apt-get install -y git && apt-get clean
RUN git clone https://github.com/MLukman/Keycloak-PII-Data-Encryption-Provider.git .
# Compatibility fix for newer Keycloak (LDAPIdentityStoreRegistry.getLdapStore signature change)
COPY ldap-getLdapStore.patch /tmp/
RUN git apply /tmp/ldap-getLdapStore.patch
RUN mvn clean package -Dkeycloak.version=$KEYCLOAK_VERSION

################################################################################

# Base image from official keycloak
FROM quay.io/keycloak/keycloak:$KEYCLOAK_VERSION

# Add provider JAR
COPY --from=keycloak-pii-data-encryption /app/target/*.jar /opt/keycloak/providers

# Need to build after adding providers
RUN /opt/keycloak/bin/kc.sh build --db=mysql --features="declarative-ui" --spi-user-provider=jpa-encrypted
