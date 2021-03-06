<a name = "adding-new-org-to-existing-network-in-indy"></a>
# Adding a new node in Indy

  - [Prerequisites](#prerequisites)
  - [Create Configuration File](#create-configuration-file)
  - [Run playbook](#run-playbook)

<a name = "prerequisites"></a>
## Prerequisites
To add a new organization in Indy, an existing Indy network should be running, pool and domain genesis files should be available. 

---
**NOTE**: Addition of a new organization has been tested on an existing network which is created by BAF. Networks created using other methods may be suitable but this has not been tested by BAF team.

---

**NOTE**: The guide is only for the addition of VALIDATOR Node in existing Indy network.

---

<a name = "create_config_file"></a>
## Create Configuration File

Refer [this guide](./indy_networkyaml.md) for details on editing the configuration file.

The `network.yaml` file should contain the specific `network.organization` patch along with the genesis information and `type` as `add-org`.

---
**NOTE**: If you are adding node to the same cluster as of another node, make sure that you add the ambassador ports of the existing node present in the cluster to the network.yaml

---
For reference, sample `network.yaml` file looks like below (but always check the latest network-indy-newnode.yaml at `platforms/hyperledger-indy/configuration/samples`):

```
---
  # This is a sample configuration file for hyperledger indy which can be reused for adding of new org with 2 validator nodes.
  # It has 1 organizations:
  # 2. organization "bank" with 1 trustee, 2 stewards and 1 endorser

  network:
    # Network level configuration specifies the attributes required for each organization
    # to join an existing network.
    type: indy
    version: 1.11.0
  
    #Environment section for Kubernetes setup
    env:
      type: indy             # tag for the environment. Important to run multiple flux on single cluster
      proxy: ambassador               # value has to be 'ambassador' as 'haproxy' has not been implemented for Indy
      # Any additional Ambassador ports can be given below, must be comma-separated without spaces. 
      # Must be different from all other ports specified in the rest of this network yaml
      ambassadorPorts: 15110,15123,15124,15133,15134,15143,15144 
      retry_count: 40                 # Retry count for the checks
      external_dns: enabled           # Should be enabled if using external-dns for automatic route configuration
  
    # Docker registry details where images are stored. This will be used to create k8s secrets
    # Please ensure all required images are built and stored in this registry.
    # Do not check-in docker_password.
    docker:
      url: "index.docker.io/hyperledgerlabs"
      username: "docker_username"
      password: "docker_password"
  
    # It's used as the Indy network name (has impact e.g. on paths where the Indy nodes look for crypto files on their local filesystem)
    name: baf 
  
    # Information about pool transaction genesis and domain transactions genesis
    # All the fields below in the genesis section are MANDATORY
    # add_org: Flag to denote if the below listed organization needs to be added to an existing organization Indy network or if the organizations are a part of a new Indy network yet to be deployed
    # pool: Pool Genesis Txn block of the existing Indy network to which the new org needs to be added
    # domain: Domain Genesis Txn block of the existing Indy network to which the new org needs to be added
    genesis:
      add_org: true
      pool: |-
        {"reqSignature":{},"txn":{"data":{"data":{"alias":"university-steward-1","blskey":"3NB4s2W7G1VKro7FqJc1EYuWwjcVKksppKebdtqLdG9dTzQWu1ZuRKVo9hydZosq83W4ruBkVa8zR1Dq1JwnjSgyTK7u3qsBCUDNMcPkZQCt1mBwawvcd23LnLPVtPtiJSj2PDFZMjwuXsvsJQM4PhQcmJkkEaixPMxsZ9ZmUzgZQ7s","blskey_pop":"QrJfHxYxhyNLUTQLVxEFy8KmFiLTKFFAEP1M5c3nRLPkaQkgbeScYCNK4CH9nCjkBii3JbMLHB9yKLTHLpJaGzGYS1XmWhv73MZrKdfaScLexeZL7uWefQ8MWHUWFpKNHop9NYA75oxrFPNE96xhmBrQmXXGM22HJ8KqS44m691gsH","client_ip":"127.0.0.1","client_port":15112,"node_ip":"127.0.0.1","node_port":15111,"services":["VALIDATOR"]},"dest":"2ZHHvp42Dhpxsa8oi8nbihcHrWsqbRzxKEDqfHZVDyRL"},"metadata":{"from":"3reGzoRtudTcfTPeV6e9NK"},"type":"0"},"txnMetadata":{"seqNo":1,"txnId":"16bcef3d14020eac552e3f893b83f00847420a02cbfdc80517425023b75f124e"},"ver":"1"}
        {"reqSignature":{},"txn":{"data":{"data":{"alias":"university-steward-2","blskey":"2Ba4viKCTKkPxeLkmEW5fzNABgf9MFR6GASkyBQXZckgWe2J9XP1foypdVWC6u1w5U1kAmtmMsXsJQoM8npzCUsgMorocWgMg4EvdRGrqXqUBMJG7z4nuSVCzokZvSBH96Rni4KDgsm4EZKE169EPs16NTGH8fECJ3i56m7tFdVqv52","blskey_pop":"R19an5nzvijHQhrN4E1enB4AfVtHiUD7cUcNAxiPeaBiv9h95qLw715rBwXH6wAJqTX1FQd8qwFwKQaA6by4N31xLXnQ8LQD5i7bt84eSzLcibZ6NAYaGuED8d8apKvegh8cJWsCGzyBKSHNX2RBrXeth7hbcta2b5EHdJNEjkWVyh","client_ip":"127.0.0.1","client_port":15122,"node_ip":"127.0.0.1","node_port":15121,"services":["VALIDATOR"]},"dest":"91XVJhNpuoq8VMiUHXx2bW16g1UQ4wRxnVWZjCQha43k"},"metadata":{"from":"FhHg9RpfSjBUfW6h7Shn8F"},"type":"0"},"txnMetadata":{"seqNo":2,"txnId":"ab3146fcbe19c6525fc9c325771d6d6474f8ddec0f2da425774a1687a4afe949"},"ver":"1"}
        {"reqSignature":{},"txn":{"data":{"data":{"alias":"employer-steward-1","blskey":"ZGrrL1ZXTP4RR5n9UjN7s31bWLc5sNGTPUEecp6XRwA75BqzpgdM6EQdJYeEFtbch2dxpj1cyitsUc4chtNGe2XErJ6ieLrrb2uuS2ksE29gnGVUxMD14apzAoetbxpEncR16v7Ye7VAFFTNU9iXNF4EGv7SndeqCR5upzVuAAWAbh","blskey_pop":"QmLcUhnhMVPEgtSqMznKiyvepmyg2xfYKLNarbDmgc64HCzZU9Ne47HQYRBs46rwc1TZNxhgxvA8HBNQoFyU5rvyn9KWPQd9sK9TudXQMNkZDHyGGQW1VsPiNshnuTHNKxxbsMFqcciBChKGGd1nJEPgrHkMCE6A7nEDW9J7XKtCtF","client_ip":"127.0.0.1","client_port":15132,"node_ip":"127.0.0.1","node_port":15131,"services":["VALIDATOR"]},"dest":"43pXez6KTwVGrDt6VvjRgeXvyXy6fidctGKZVPMEMSXZ"},"metadata":{"from":"6bRJz1UqmxnjmhYkwut8ci"},"type":"0"},"txnMetadata":{"seqNo":3,"txnId":"d85334ed1fb537b2ff8627b8cc4bcf2596d5da62c6d85244b80675ebae91fd07"},"ver":"1"}
        {"reqSignature":{},"txn":{"data":{"data":{"alias":"employer-steward-2","blskey":"3DwLwSp4kvpXLexqey8tdeVbBGdkEuMf8skCSYz1Xfu5QiShBeUMdqJer7oJHteVeaRmSGQ2VG6Mmx9qf5hnrVRzAw3YkYVn2GpsAVRAjm2TFVEkCrCzWU7JtEtMyhjc4eCxJX7Nkuu7dbvuLx8hUzF5Ev7qX5cab4KuTb428sYVimj","blskey_pop":"RS81jzNE4RtwUjXnd7D6wRVocwsqRdBcu3Z421481bp2s3sgAVpeXuMRDSdxs3n66T4oXFPrb1HvxFPNEUL3HvAmdSGq2FpNaEfhfpnHdnTCNod3xfMc4wMcoYzc4Jr23LpPxJdCYPtBHuko78RRUGKnsTLGvcy6Gx5wVVnXszZgfR","client_ip":"127.0.0.1","client_port":15142,"node_ip":"127.0.0.1","node_port":15141,"services":["VALIDATOR"]},"dest":"2ydXcHkmWVp9K3tWajvonAkcJCyGFnKN4Hcr7f7s5Vhb"},"metadata":{"from":"4dKNcjECQsoq5JaumirCTH"},"type":"0"},"txnMetadata":{"seqNo":4,"txnId":"1b0dca5cd6ffe526ab65f1704b34ec24096b75f79d4c0468a625229ed686f42a"},"ver":"1"}
      domain: |-
        {"reqSignature":{},"txn":{"data":{"alias":"authority-trustee","dest":"72LiZq46oM9emBVCniosiP","role":"0","verkey":"4HQKopFsnMSbWyfysAjAsTTBjjwdfyPQ8b3iPX8vzLZL"},"metadata":{},"type":"1"},"txnMetadata":{"seqNo":1},"ver":"1"}
        {"reqSignature":{},"txn":{"data":{"alias":"university-trustee","dest":"2wsaJxpE3527P4j9AMZNFG","role":"0","verkey":"24X77wNrAU3ekgjPjkczkU3ZrwXtwBu27Z3Fz5i9WDzK"},"metadata":{"from":"72LiZq46oM9emBVCniosiP"},"type":"1"},"txnMetadata":{"seqNo":2},"ver":"1"}
        {"reqSignature":{},"txn":{"data":{"alias":"university-steward-1","dest":"3reGzoRtudTcfTPeV6e9NK","role":"2","verkey":"2ZHHvp42Dhpxsa8oi8nbihcHrWsqbRzxKEDqfHZVDyRL"},"metadata":{"from":"2wsaJxpE3527P4j9AMZNFG"},"type":"1"},"txnMetadata":{"seqNo":3},"ver":"1"}
        {"reqSignature":{},"txn":{"data":{"alias":"university-steward-2","dest":"FhHg9RpfSjBUfW6h7Shn8F","role":"2","verkey":"91XVJhNpuoq8VMiUHXx2bW16g1UQ4wRxnVWZjCQha43k"},"metadata":{"from":"2wsaJxpE3527P4j9AMZNFG"},"type":"1"},"txnMetadata":{"seqNo":4},"ver":"1"}
        {"reqSignature":{},"txn":{"data":{"alias":"employer-trustee","dest":"86w9vTqsju1RwvtbrE11kj","role":"0","verkey":"4sX35pLcjeDiKYFT6gckPupFm4w68kPZ75mFqRihfH9B"},"metadata":{"from":"72LiZq46oM9emBVCniosiP"},"type":"1"},"txnMetadata":{"seqNo":5},"ver":"1"}
        {"reqSignature":{},"txn":{"data":{"alias":"employer-steward-1","dest":"6bRJz1UqmxnjmhYkwut8ci","role":"2","verkey":"43pXez6KTwVGrDt6VvjRgeXvyXy6fidctGKZVPMEMSXZ"},"metadata":{"from":"86w9vTqsju1RwvtbrE11kj"},"type":"1"},"txnMetadata":{"seqNo":6},"ver":"1"}
        {"reqSignature":{},"txn":{"data":{"alias":"employer-steward-2","dest":"4dKNcjECQsoq5JaumirCTH","role":"2","verkey":"2ydXcHkmWVp9K3tWajvonAkcJCyGFnKN4Hcr7f7s5Vhb"},"metadata":{"from":"86w9vTqsju1RwvtbrE11kj"},"type":"1"},"txnMetadata":{"seqNo":7},"ver":"1"}
  
    # Allows specification of one or many organizations that will be connecting to a network.
    organizations:
    - organization:
      name: provider
      type: peer
      cloud_provider: aws-baremetal             # Currently eks is not supported due to aws_authenticator
      external_url_suffix: indy.blockchaincloudpoc.com  # Provide the external dns suffix. Only used when Indy webserver/Clients are deployed.

      aws:
        access_key: "aws_access_key"            # AWS Access key
        secret_key: "aws_secret_key"            # AWS Secret key
        encryption_key: "encryption_key_id"     # AWS encryption key. If present, it's used as the KMS key id for K8S storage class encryption.
        zone: "availability_zone"               # AWS availability zone
        region: "region"                        # AWS region

      publicIps: ["3.221.78.194"]               # List of all public IP addresses of each availability zone from all organizations in the same k8s cluster                        # List of all public IP addresses of each availability zone
  
      # Kubernetes cluster deployment variables. The config file path has to be provided in case
      # the cluster has already been created.
      k8s:
        config_file: "cluster_config"
        context: "kubernetes-admin@kubernetes"

      # Hashicorp Vault server address and root-token. Vault should be unsealed.
      # Do not check-in root_token
      vault:
        url: "vault_addr"
        root_token: "vault_root_token"

      # Git Repo details which will be used by GitOps/Flux.
      # Do not check-in git_access_token
      gitops:
        git_protocol: "https" # Option for git over https or ssh
        git_url: "https://github.com/<username>/blockchain-automation-framework.git"                   # Gitops https or ssh url for flux value files 
        branch: "develop"                     # Git branch where release is being made
        release_dir: "platforms/hyperledger-indy/releases/dev"           # Relative Path in the Git repo for flux sync per environment. 
        chart_source: "platforms/hyperledger-indy/charts"               # Relative Path where the Helm charts are stored in Git repo
        git_repo: "github.com/<username>/blockchain-automation-framework.git"             # Gitops git repository URL for git push 
        username: "git_username"                    # Git Service user who has rights to check-in in all branches
        password: "git_access_token"                    # Git Server user password
        email: "git_email"                          # Email to use in git config
        private_key: "path_to_private_key"          # Optional (required when protocol is ssh) : Path to private key file which has write-access to the git repo

      # Services maps to the pods that will be deployed on the k8s cluster
      # This sample has trustee, 2 stewards and endoorser
      services:
        trustees:
        - trustee:
          name: provider-trustee
          genesis: true
        stewards:
        - steward:
          name: provider-steward-1
          type: VALIDATOR
          genesis: true
          publicIp: 3.221.78.194    # IP address of current organization in current availability zone
          node:
            port: 9711
            targetPort: 9711
            ambassador: 9711        # Port for ambassador service
          client:
            port: 9712
            targetPort: 9712
            ambassador: 9712        # Port for ambassador service
        - steward:
          name: provider-steward-2
          type: VALIDATOR
          genesis: true
          publicIp: 3.221.78.194    # IP address of current organization in current availability zone
          node:
            port: 9721
            targetPort: 9721
            ambassador: 9721        # Port for ambassador service
          client:
            port: 9722
            targetPort: 9722
            ambassador: 9722        # Port for ambassador service
        endorsers:
        - endorser:
          name: provider-endorser
          full_name: Some Decentralized Identity Mobile Services Provider
          avatar: http://provider.com/avatar.png

```
Three new sections are added/updated to the network.yaml used to add new organisations
| Field       | Description                                              |
|-------------|----------------------------------------------------------|
| genesis.add_org | Flag to denote if the below listed organization needs to be added to an existing organization Indy network or if the organizations are a part of a new Indy network yet to be deployed |
| genesis.pool | Pool Genesis of the existing Indy network.|
| genesis.domain | Domain Genesis of the existing Indy network.|


<a name = "run_network"></a>
## Run playbook

The [site.yaml](https://github.com/hyperledger-labs/blockchain-automation-framework/tree/master/platforms/shared/configuration/site.yaml) playbook is used to add a new organization to the existing network. This can be done using the following command

```
ansible-playbook platforms/shared/configuration/site.yaml --extra-vars "@path-to-network.yaml" 
```
