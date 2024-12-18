# Overview
During the time contributing to the liberdus project, creating a local network is a great way to test your changes and see how they affect the network. This guide will walk you through the process of setting up a local network and launching it.
Liberdus network is built on top of shardus protocol which is framework for building sharded and decentralized apps. The primary library @shardus/core is powered by rust binded networking protocol. 

# Prerequisites
Make sure you system has the following installed:
- [Rust](https://www.rust-lang.org/tools/install)
- [Node.js](https://nodejs.org/en/download/)
- [Python3](https://www.python.org/downloads/)

The exact version of nodejs version required can be oberseved in liberdus/server's package.json file.

# Cloning the repositories
First, clone the liberdus repository to your local machine. You can do this by running the following command:
```bash
git clone https://github.com/Liberdus/server.git
```
Please use dev branch for the latest changes.
```bash
git checkout dev
```
Once the repository is cloned, navigate to the repo directory and install the dependencies by running:
```bash
cd server
npm install
```
To launch a local network, shardus provide a process manager that allow you to launch multiple network along with an archiver and monitoring system to form a working network. Install the shardus cli tool by running:
```bash
npm install -g shardus
```

# Launching the network
To launch a local network, you can use the shardus cli tool. The following command will launch a network with 10 validator node, archiver node and a monitor server.
```bash
shardus create-net 10
```
After creating the network, you can observe the network status by visiting http://localhost:3000.


# Launching the rpc server
Clone the rpc server repository to your local machine. You can do this by running the following command:
```bash
git clone git@github.com:Liberdus/liberdus-rpc.git
```
Once the repository is cloned, navigate to the repo directory and install the dependencies by running:

```bash
cd liberdus-rpc
rustup install 1.81
rustup default 1.81
```
To launch the server, run the following command:
```bash
cargo run
```
RPC server will be running on http://localhost:8545.

# Launching the liberdus demo client
In order to interact with the network, you can use the liberdus demo client. Clone the repository to your local machine by running the following command:
```bash
git clone https://github.com/Liberdus/liberdus-web-client.git
```
To run the vue project
```bash
cd liberdus-web-client
npm install
npm run serve
```









