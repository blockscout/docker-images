# ARG defined before FROM in multi-staged Dockerfile is shared among the stages
ARG KEYCLOAK_VERSION=26.8.2

# Build the provider
FROM maven:3.9.13-sapmachine-17 AS keycloak-pii-data-encryption
ARG KEYCLOAK_VERSION # Dockerfile peculiarity that requires ARG defined before FROM to be re-declared afterwards if we want to use it in the stage
# Pinned, so that upstream changes get into the image only after a review
ARG PII_PROVIDER_COMMIT=b27910429104d3faeffe108dc5440f51891f0ab3
WORKDIR /app
RUN apt-get update && apt-get install -y git && apt-get clean
RUN git clone https://github.com/MLukman/Keycloak-PII-Data-Encryption-Provider.git . && git checkout "$PII_PROVIDER_COMMIT"
# Compatibility fix for newer Keycloak (LDAPIdentityStoreRegistry.getLdapStore signature change)
COPY ldap-getLdapStore.patch /tmp/
RUN git apply /tmp/ldap-getLdapStore.patch
RUN mvn clean package -Dkeycloak.version=$KEYCLOAK_VERSION

################################################################################

# Build Blockscout endpoints for atomic updates of user data (https://github.com/blockscout/keycloak-user-api-extension)
FROM maven:3.9.13-sapmachine-17 AS keycloak-user-api-extension
ARG KEYCLOAK_VERSION
ARG KEYCLOAK_USER_API_EXTENSION_VERSION=v0.4.0
WORKDIR /app
RUN apt-get update && apt-get install -y git && apt-get clean
# The repository is private. The token is passed as a BuildKit secret and only to this git command,
# so it is not stored in any layer. The source is cloned, built and removed in one step, so that only the jar,
# which is in the image anyway, gets into a layer: intermediate layers are exported to the public build cache.
RUN --mount=type=secret,id=github_token,required=true \
    git -c "http.extraHeader=Authorization: Basic $(printf 'x-access-token:%s' "$(cat /run/secrets/github_token)" | base64 -w0)" \
        clone --depth 1 --branch "$KEYCLOAK_USER_API_EXTENSION_VERSION" \
        https://github.com/blockscout/keycloak-user-api-extension.git /tmp/src \
    && mvn -f /tmp/src/pom.xml clean package -Dkeycloak.version=$KEYCLOAK_VERSION \
    && mkdir -p target && cp /tmp/src/target/keycloak-user-api-extension.jar target/ \
    && rm -rf /tmp/src

################################################################################

# Base image from official keycloak
FROM quay.io/keycloak/keycloak:$KEYCLOAK_VERSION

# Add provider JARs
COPY --from=keycloak-pii-data-encryption /app/target/*.jar /opt/keycloak/providers
COPY --from=keycloak-user-api-extension /app/target/keycloak-user-api-extension.jar /opt/keycloak/providers

# Need to build after adding providers
RUN /opt/keycloak/bin/kc.sh build --db=mysql --features="declarative-ui" --spi-user-provider=jpa-encrypted
