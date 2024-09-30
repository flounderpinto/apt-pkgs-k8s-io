FROM ubuntu:jammy-20240808

#TODO
#ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
#Packages
    apt-get install -y \
      aptly \
      curl \
      nginx && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/* && \
#Download and save the kubernetes key
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | \
       tee /etc/apt/sources.list.d/kubernetes.list && \
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | \
       gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg && \
#Create a new key for 'aptly publish'
    status=$(gpg --quick-gen-key --batch --passphrase '' none@none.com) && \
    key_name=$(echo $status | sed -nr "s/^.*revocs\.d\/(.*)\.rev'$/\1/p") && \
    gpg --armor --output ./output.key --export $key_name && \
#Clone the apt repo
    aptly \
       -architectures="amd64" \
       -keyring=/etc/apt/keyrings/kubernetes-apt-keyring.gpg \
       mirror create \
       kubernetes \
       https://pkgs.k8s.io/core:/stable:/v1.28/deb/ \
       ./ && \
    aptly \
       -keyring=/etc/apt/keyrings/kubernetes-apt-keyring.gpg \
       mirror update \
       kubernetes && \
    aptly \
       snapshot create \
       kubernetes-snapshot from mirror kubernetes && \
    aptly \
       publish snapshot \
       kubernetes-snapshot && \
    chmod -R 444 /root/.aptly && \
    echo '\
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
      root /root/.aptly/public; \
      autoindex on; \
    } \
  } \
} \
' > /etc/nginx/nginx.conf

ENTRYPOINT ["nginx"]
