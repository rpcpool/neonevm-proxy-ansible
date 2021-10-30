# NeonEVM Proxy 

The workbookes here sets up a dedicated Neon EVM proxy host. They are intended as a starting point for a new NeonEVM node operator. 

## Basic requirements

The NeonEVM proxy should best be run with a dedicated Solana RPC node. You will need to have minimal possible latency between your Solana RPC and the Neon EVM. Our recommended approach is to run the NeonEVM proxy on the same host as the Solana RPC node.

If you run the NeonEVM proxy on a separate node from your Solana RPC node, then you should ensure the following:

 - Minimal latency between NeonEVM and the Solana node, ideally they should be on the same private or local network or at least within the same DC.
 - No public access to the Solana RPC node and (ideally) no other Solana RPC traffic to the Neon EVM node: Since it is very easy to make a Solana RPC node fall behind the tip of the network, which would cause issues for the Neon EVM proxy we **strongly** recommend that you have a dedicated RPC node for Neon EVM.

Using a shared or public RPC node for NeonEVM is accordingly not recommended as there is too strong possibility that the node will have variable performance and therefore cause your Neon EVM proxy to fail to submit or confirm some of its transactions, which would mean that you risk loosing the benefits from processing those transactions.

If you don't want to run your own Solana RPC node but just want to be a Neon EVM operator, then [Triton](https://triton.one) is one of several RPC providers in the [Solana ecosystem](https://solana.com/ecosystem) that can provide you with a dedicated node to point your Neon EVM proxy to (see below for how to specify the Solana RPC url).

## Hardware requirements

An RPC server requires at least the same specs as a Solana validator, but typically has higher requirements. For more information about hardware requirements, please see [the Solana docs](https://docs.solana.com/running-validator/validator-reqs). In particular they recommend:

 - 16 cores
 - 256 gb memory

For the EVM proxy, the [hardware requirements](https://docs.neon-labs.org/docs/proxy/operator_guide#hardware-recommendations). NeonEVM recommends:

 - 4 cores
 - 16 gb memory

In sum, this suggest that a combined node running NeonEVM and the Solana RPC could have the following recommended specs:

 - 16-24 cores of a recent processor Zen 2, Zen 3 or latest Cascade Lake Refresh Xeons
 - 256 gb memory 

A suitable processor might be EPYC 7443P or 7413P.

Considering the bandwidth requirements of Solana we recommend using a baremetal host rather than running on the cloud, as you will otherwise pay high egress fees as well as (potentially) see low performance due to the virtualisation layer of cloud providers.

## Software requirements

These deploys are built for [Ansible 2.8](https://docs.ansible.com/ansible/2.8/user_guide/index.html) on Ubuntu 20.04 LTS. Any Debian-like Linux distribution you should work as well.

## Sample playbook

Before running the playbook, make sure you install the required roles:

```
ansible-galaxy install rpcpool.solana_rpc
ansible-galaxy install rpcpool.neonevm_proxy
```


To deploy the Neon EVM proxy with a Solana RPC node on the local machine, running on devnet:

```
- name: deploy rpc hosts
  remote_user: root
  hosts:
    - 127.0.0.1
  roles:
    - role: rpcpool.solana_rpc
      vars:
  	solana_network: devnet
    - role: rpcpool.neonevm_proxy
      vars:
        neonevm_network: devnet
``` 

More details about the configuration of the [Solana RPC node can be found here](https://github.com/rpcpool/solana-rpc-ansible).


## Neon EVM setup

### Choosing an network

The main configuration variable is `neonevm_network`. Set this to `mainnet`, `testnet` or `devnet`. The default is `devnet`. This will set the neonevm `CONFIG` environment variable which will set the correct values for `EVM_LOADER`, `COLLATERAL_POOL_BASE` and `ETH_TOKEN_MINT`.

### Setting solana RPC server

You also need to set `neonevm_solana_rpc`. This will default to our recommended version of running Solana on localhost - i.e. `http://localhost:8899`. If you want to run Neon EVM separate from the Solana node you can specify a different RPC URL in this variable.


### Creating a custom config

You can create a custom config by specifying `neonevm_env`. For example to run Neon EVM on a local cluster specify the following:
```
neonevm_env:
	SOLANA_URL: http://localhost:8899
	CONFIG: local
	EVM_LOADER: deploy # not strictly required as this is the default value for 'CONFIG: local'
	COLLATERAL_POOL_BASE: deploy  # not strictly required as this is the default value for 'CONFIG: local'
	ETH_TOKEN_MINT: deploy # not strictly required as this is the default value for 'CONFIG: local'
```

### Neon EVM keypair

The playbook creates a new Neon EVM keypair by default located in `/home/neonevm/neonevm-keypair.json`. You need to fund this keypair for your EVM to work on the Solana chain. The keypair hash can be found in `neonevm.pub` in the deployment directory after the deploy has successfully completed. You can transfer funds to this key from any wallet. 


