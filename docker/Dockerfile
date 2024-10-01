FROM ubuntu:jammy-20240808

ARG APTLY_DIR=/root/.aptly
ARG KEYRING_GPG=/etc/apt/keyrings/kubernetes-apt-keyring.gpg
ARG KUBERNETES_REPO=https://pkgs.k8s.io/core:/stable:/v1.28/deb/ 

RUN apt-get update && \
#Packages
    apt-get install -y \
      aptly \
      curl \
      nginx && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/* && \
#Download and save the kubernetes key
    echo "deb [signed-by=$KEYRING_GPG] $KUBERNETES_REPO/" | \
       tee /etc/apt/sources.list.d/kubernetes.list && \
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | \
       gpg --dearmor -o $KEYRING_GPG && \
#Clone the apt repo
    aptly \
       -architectures="amd64" \
       -keyring=$KEYRING_GPG \
       mirror create \
       kubernetes \
       $KUBERNETES_REPO\
       ./ && \
    aptly \
       -keyring=$KEYRING_GPG \
       mirror update \
       kubernetes && \
    aptly \
       snapshot create \
       kubernetes-snapshot from mirror kubernetes && \
#Create a new key for 'aptly publish'
    mkdir $APTLY_DIR/public && \
    status=$(gpg --quick-gen-key --batch --passphrase '' none@none.com) && \
    key_name=$(echo $status | sed -nr "s/^.*revocs\.d\/(.*)\.rev'$/\1/p") && \
    gpg --armor --output $APTLY_DIR/public/gpg --export $key_name && \
#Publish
    aptly \
       publish snapshot \
       kubernetes-snapshot && \
#Setup nginx
    chmod -R 444 $APTLY_DIR
RUN echo "\
events{} \
daemon off; \
user root; \
http { \
  server { \
    listen 80; \
    access_log /dev/stdout; \
    error_log /dev/stderr; \
\
    location / { \
      root $APTLY_DIR/public; \
      autoindex on; \
    } \
  } \
} \
" > /etc/nginx/nginx.conf

ENTRYPOINT ["nginx"]
