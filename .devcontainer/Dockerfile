FROM mcr.microsoft.com/devcontainers/java:1-17-bullseye

# Sedona 1.9 targets Java 11 bytecode; Flink 2.x requires Java 11/17/21 at runtime.
# JDK 17 as the build/toolchain JDK satisfies both.
RUN apt-get update && apt-get install -y --no-install-recommends \
        python3 \
        python3-pip \
        unzip \
        maven \
        ca-certificates \
        libxext6 \
        libxrender1 \
        libxtst6 \
    && rm -rf /var/lib/apt/lists/*

# Optional local corporate/network TLS-inspection root CAs (not committed — see
# certs/.gitignore). On a network with a MITM inspection proxy, Maven's HTTPS
# dependency downloads fail cert verification inside the container unless the
# proxy's root CA is trusted here too. Drop .crt files in certs/ locally to fix.
COPY certs/ /usr/local/share/ca-certificates/extra/
RUN update-ca-certificates

# The JDK keeps its own trust store separate from the OS bundle above — Maven
# (running on the JVM) needs the same corporate/proxy roots imported here too,
# or HTTPS dependency downloads fail PKIX validation even though curl/wget work.
RUN for f in /usr/local/share/ca-certificates/extra/*.crt; do \
        [ -f "$f" ] || continue; \
        alias=$(basename "$f" .crt); \
        keytool -importcert -noprompt -trustcacerts \
            -alias "$alias" \
            -file "$f" \
            -keystore "$JAVA_HOME/lib/security/cacerts" \
            -storepass changeit; \
    done

# Node.js (VS Code Server, extension marketplace calls, any npm-based tooling) bundles its
# own CA list and ignores the OS/JDK trust stores above unless told otherwise — same corporate
# root CAs need to be handed to it explicitly via NODE_EXTRA_CA_CERTS as one combined PEM file.
RUN cat /usr/local/share/ca-certificates/extra/*.crt > /usr/local/share/ca-certificates/extra-bundle.pem 2>/dev/null || true
ENV NODE_EXTRA_CA_CERTS=/usr/local/share/ca-certificates/extra-bundle.pem
