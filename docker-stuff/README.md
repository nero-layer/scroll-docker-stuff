## Scroll Deployment Plan

- Get common/docker/l1geth and common/docker/l2geth up and running
- Get the database up and running
- Run local blockchain
- Npx hardhat node

``` bash
# Install go
wget https://go.dev/dl/go1.20.2.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.20.2.linux-amd64.tar.gz
vim ~/.bashrc
export PATH=$PATH:/usr/local/go/bin
export PATH=$PATH:~/go/bin

# Install rust
curl https://sh.rustup.rs -sSf | sh

# Install foudnerty
curl -L https://foundry.paradigm.xyz | bash
export PATH=$PATH:~/.foundry/bin
foundryup --branch master

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo apt-get -y install docker-compose
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER

# Install tmux
sudo apt install -y tmux
sudo apt-get install -y postgresql-client

# Make sure you git ssh enabled
# ssh-keygen -t rsa -C "you@example.com"
git clone --recursive https://github.com/scroll-tech/scroll.git

# Build L1 and L2 Chain Networks
cd scroll
make dev_docker


# Deploy Postgres Database
curl  https://gist.githubusercontent.com/dentropy/e408f86de7261a516af9bb43234ae343/raw/5e764a89037921d5022f76963b516ba1fc133820/ -o postgres.docker-compose.yml
docker-compose -f postgres.docker-compose.yml up -d
# Build Database Migration Tool
# Migrate Database
cd  database 
make db_cli
vim config.json
db_cli migrate
psql postgresql://postgres:postgres@127.0.0.1:5432/postgres?sslmode=disable
# edit ./database/config.json to use postgres string above
\dt
exit
cd ..

# Build coordinator
cd coorinator
make libzkp
make coordinator
make test-verifier
make docker
cd ..

# Build Prover
cd prover
make clean
make prover
make docker

# Build rollup
go install -v github.com/scroll-tech/go-ethereum/cmd/abigen
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install solc
make clean
make mock_abi
make rollup_bins
make docker

# Run L1 and L2 Chain Networks
docker run -p 8545:8545 -p 8546:8546 scroll_l1geth
docker run -p 9999:8545 -p 10000:8546 scroll_l2geth

# Run Coordinator
cd coordinator
export LD_LIBRARY_PATH=$HOME/Projects/Blockhack/scroll/coordinator/internal/logic/verifier/lib:$LD_LIBRARY_PATH
ldd $HOME/scroll/coordinator/internal/logic/verifier/lib/libzktrie.so
./build/bin/coordinator \
	 --config ./config.json \
     --http --http.addr localhost --http.port 8390

# I GET ERROR HERE

# Run Prover
export CHAIN_ID=534353 # change to correct chain ID
export RUST_MIN_STACK=100000000
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:./prover/lib
./build/bin/prover

# Deploy Contracts

# Configure Bridge History API

```

## Docker Solutions

**Create Docker Network**


``` bash
docker network create --subnet=10.99.0.0/16 scrollnet
```

**L2 and L2 Nodes**

``` yaml
version: '3.1'

networks:
    traefik-homelab:
      external:
        name: scrollnet

services:
	scroll_l1geth:
		image: scroll_l1geth
		container_name: 
		restart: always
		ports:
		  - 8545:8545
		  - 8546:8546
	    networks:
	        scrollnet:
	scroll_l2geth:
		image: scroll_l2geth
		container_name:
		restart: always
		ports:
		  - 9999:8545
		  - 10000:8546
	    networks:
	      - scrollnet

```

**Postgres Database**
``` yaml
version: '3.1'

networks:
    scrollnet:
      external:
        name: scrollnet

services:
  postgres:
    image: postgres
    container_name: postgres
    hostname: postgres
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres 
      POSTGRES_DB: postgres
    ports:
      - 5432:5432
    networks:
      - scrollnet
    # volumes: 
    #  - ./postgres_db_1:/var/lib/postgresql/data

```

``` bash
psql postgresql://postgres:postgres@127.0.0.1:5432/postgres
```

``` bash
docker run -it --network scrollnet ubuntu bash
apt-get update
apt install -y iputils-ping
apt install -y postgresql-client
ping postgres
psql postgresql://postgres:postgres@postgres:5432/postgres?sslmode=disable
```

**Migrate Postgres Database

``` bash
make db_cli
# Update config.json
./build/bin/db_cli migrate

# Check that the migration is valid
psql postgresql://postgres:postgres@127.0.0.1:5432/postgres
\dt


# Docker Does not work nicely, just build and run using 
# postgres url in config.json
docker run -it --rm \
	-v $(pwd):/mount \
	scrolltech/db_cli
	"/bin/db_cli --config /mount/config.json migrate"
docker exec -it 6933a93e725e sh
cd mount
db_cli --config migrate
```

**Deploy Coordinator**

Update `./coordinator/conf/config.json` with postgres URL below

```
postgresql://postgres:postgres@postgres:5432/postgres?sslmode=disable
```

``` yaml

version: '3.1'

networks:
    scrollnet:
      external:
        name: scrollnet

services:
  coordinator:
    image: scrolltech/coordinator
    container_name: coordinator
    restart: always
    ports:
      - 8555:8555
    volumes:
      - ./coordinator/conf/config.json:/config.json
    # environment:
    networks:
      - scrollnet

```

**Deploy Prover**

``` bash
export LD_LIBRARY_PATH=/home/dentropy/Projects/Blockhack/scroll/coordinator


cd prober
./prover

```

**Deploy Contracts**

``` bash
cd ./contracts
npm install
npx hardhat test
cp .env.example .env
vim .env
# SCROLL_L1_RPC=https://localhost:8545
# SCROLL_L2_RPC=https://localhost:9999
cat ./scripts/README.md
```