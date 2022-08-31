---
sidebar_position: 2
slug: /testnet-manual
title: Setup - Manual
---
# Join testnet - Manual setup
## Prerequisites

1. Verify [hardware requirements](reqs) are met
2. Install package dependencies
    - Note: You may need to run as `sudo`
    - Required packages installation
        
        ```bash
        ### Packages installations
        sudo apt update # in case of permissions error, try running with sudo
        sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
        # Create the temp dir for the installation
        temp_folder=$(mktemp -d) && cd $temp_folder
        ```
        
    - Go installation
        
        ```bash
        ### Configurations
        go_package_url="https://go.dev/dl/go1.18.linux-amd64.tar.gz"
        go_package_file_name=${go_package_url##*\/}
        # Download GO
        wget -q $go_package_url
        # Unpack the GO installation file
        sudo tar -C /usr/local -xzf $go_package_file_name
        # Environment adjustments
        echo "export PATH=\$PATH:/usr/local/go/bin" >>~/.profile
        echo "export PATH=\$PATH:\$(go env GOPATH)/bin" >>~/.profile
        source ~/.profile
        ```
        
    - Installation verifications
        
        ```
        1. You can verify the installed go version by running: "go version"
        
        2. The command "go env GOPATH" should include $HOME/go
        If not, then, export GOPATH=$HOME/go
        
        3. PATH should include $HOME/go/bin
        ```
        

## 1. Set up a local node

### A. Download app configurations

- Download setup configuration
    
    Download the configuration files needed for the installation
    
    ```bash
    # Download the installation setup configuration
    git clone https://github.com/K433QLtr6RA9ExEq/GHFkqmTzpdNLDd6T.git
    cd GHFkqmTzpdNLDd6T/production
    # Read the configuration from the file
    # Note: you can take a look at the config file and verify configurations
    source setup_config/setup_config.sh
    ```
    
- Set app configurations
        
    Copy lavad default config files to config Lava config folder
    
    ```bash
    echo "Lava config file path: $lava_config_folder"
    mkdir -p $lavad_home_folder
    mkdir -p $lava_config_folder
    cp default_lavad_config_files/* $lava_config_folder
    ```
    

### B. Set the genesis file

- Set the genesis file in the configuration folder
    
    ```bash
    # Copy the genesis.json file to the Lava config folder
    cp genesis_json/genesis.json $lava_config_folder/genesis.json
    ```
        

## 2. Join the Lava Testnet

:::info
The following sections will describe how to install Cosmovisor for automating the upgrades process. If you choose to not take this path, please [refer to the manual upgrade page](testnet-upgrade-manual) for completing the sync to testnet to its latest block. 
:::

### A. Set up Cosmovisor {#cosmovisor}

- Set up cosmovisor to ensure any future upgrades happen flawlessly. To install Cosmovisor:
    
    ```bash
    go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0
    # Create the Cosmovisor folder and copy config files to it
    mkdir -p $lavad_home_folder/cosmovisor
    cp -r cosmovisor-upgrades/* $lavad_home_folder/cosmovisor
    ```

    ```bash
    # Set the environment variables
    echo "# Setup Cosmovisor" >> ~/.profile
    echo "export DAEMON_NAME=lavad" >> ~/.profile
    echo "export CHAIN_ID=lava" >> ~/.profile
    echo "export DAEMON_HOME=$HOME/.lava" >> ~/.profile
    echo "export DAEMON_ALLOW_DOWNLOAD_BINARIES=true" >> ~/.profile
    echo "export DAEMON_LOG_BUFFER_SIZE=512" >> ~/.profile
    echo "export DAEMON_RESTART_AFTER_UPGRADE=true" >> ~/.profile
    echo "export UNSAFE_SKIP_BACKUP=true" >> ~/.profile
    source ~/.profile
    ```

    ```bash
    # Initialize the chain
    $lavad_home_folder/cosmovisor/genesis/bin/lavad init \
    my-node \
    --chain-id lava \
    --home $lavad_home_folder \
    --overwrite
    cp genesis_json/genesis.json $lava_config_folder/genesis.json
    ```

    ```bash
    # Please note that cosmovisor will throw an error, this is ok:
    # lstat /home/ubuntu/.lava/cosmovisor/current/upgrade-info.json: no such file or directory
    cosmovisor version
    ```
    
    Create the systemd unit file
    
    ```bash
    # Create Cosmovisor unit file

    echo "[Unit]
    Description=Cosmovisor daemon
    After=network-online.target
    [Service]
    Environment="DAEMON_NAME=lavad"
    Environment="DAEMON_HOME=${HOME}/.lava"
    Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
    Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
    Environment="DAEMON_LOG_BUFFER_SIZE=512"
    Environment="UNSAFE_SKIP_BACKUP=true"
    User=$USER
    ExecStart=${HOME}/go/bin/cosmovisor start --home=$lavad_home_folder --p2p.seeds $seed_node
    Restart=always
    RestartSec=3
    LimitNOFILE=infinity
    LimitNPROC=infinity
    [Install]
    WantedBy=multi-user.target
    " >cosmovisor.service
    sudo mv cosmovisor.service /lib/systemd/system/cosmovisor.service
    ```
    
- Enable and start the Cosmovisor service
    
    ```bash
    # Enable the cosmovisor service so that it will start automatically when the system boots
    sudo systemctl daemon-reload
    sudo systemctl enable cosmovisor.service
    sudo systemctl restart systemd-journald
    sudo systemctl start cosmovisor
     ```
    

## 3. Verify setup

### A. Make sure the `cosmovisor` service is running

- Check the state of the cosmovisor service
    
    ```bash
    sudo systemctl status cosmovisor
    # To view the service logs
    sudo journalctl -u cosmovisor -f
    ```
    

## Done 🌋

:::tip Joined Testnet? Be a validator!
You are now running a Node in the Lava network 🎉🥳! 

Congrats, happy to have you here 😻 Celebrate it with us on Discord.

When you're ready, start putting the node to use **as a validator**:
[<RoadmapItem icon="🧑‍⚖️" title="Power as a Validator" description="Validate blocks, secure the network, earn rewards"/>](validator#account)

:::