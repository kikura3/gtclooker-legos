

# HOW TO SETUP AND RUN THE LEGOS

## Environment Variables 
--------

| Name               | Value         | Description                          | Used by Lego |
| -------------------| ------------ | ---------------------------- | ----- |
| GRAPH_GITCOIN_GRANTS_URL | https://api.studio.thegraph.com &nbsp;/query/41140/gitcoin-grants/v0.0.7 | Subgraph Deployed at the Graph to retrieve grant data | Grant Data Extract      |
| WEB3_INFURA_PROJECT_ID | Get it from Infura(FREE) | Required by Ocean protocol to write/read transactions | All Legos      |
| IPFS_INFURA_* KEYS  | Get it from Infura(FREE until 5GB) | Required to write and pin IPFS files | All Legos      |
| ACCOUNT_PRIVATE_KEY | Your wallet account private key | This is required by ocean protocol to sign read/write transactions via script | All legos     |
| DATA_NFT_ADDRESS | See below | This acts as a datastore to store <job_id,file location> so that other jobs can access it  | All legos     |
| DATA_ASSET_DID | See below | This is required to update the dataset as an asset in ocean protocol for use by others | Grant Data Aggregate     |
| NETWORK | polygon-test for testnet, polygon-main for mainnet or any other network | The data store and assets are stored in polygon network | All Legos  |
| FLIPSIDE_API_KEY | Get it from flipside(FREE) | API Key to fetch onchain data | Metric Legos  |
| TWITTER_* KEYS  | Get it from Twitter(FREE) | API Key to fetch social presence data | Metric Legos  |
| SUPABASE_* KEYS | Get it from Supabase(FREE) | To update the insights to DB | Transform Insights to DB  |

**PS : Never make ACCOUNT_PRIVATE_KEY public**

&nbsp;
&nbsp;

## HOW TO CREATE DATA STORE IN OCEAN PROTOCOL (DATA_NFT_ADDRESS token)
----

&nbsp;

1. set env variable NETWORK as polygon-test to create in mumbai test. use polygon-main to create in mainnet

```
cd src
python3 -m venv venv
source venv/bin/activate
pip3 install -r ../requirements.txt
python ../scripts/create_data_nft.py
```

2. Record the Data NFT Address & update it to .env

&nbsp;

## HOW TO CREATE DATA ASSET IN OCEAN PROTOCOL (DATA_ASSET_DID token)
----

1. set env variable NETWORK as polygon-test to create in mumbai test. use polygon-main to create in mainnet

```
cd src
python3 -m venv venv
source venv/bin/activate
pip3 install -r ../requirements.txt
python ../scripts/create_url_asset.py
```

2. Record the DID & update it to .env

3. You can check if the asset is created here:
```
market.oceanprotocol.com/asset/<did>
```

&nbsp;

## HOW TO TEST IT LOCALLY
-----

Once all the required environment variables are updated in the .env file (use .env.template as reference)

1. The first step is to run the grant_data_extract job which
 extracts data from the graph & updates to ipfs

    ```
    python3 -m venv venv
    source venv/bin/activate
    pip3 install -r requirements.txt
    python run.py -n grant_data_extract
    ```

    PS: The subgraph is currently configured for gitcoin alpha round. It needs to be updated to capture new rounds.

2. (Optional) You may want to upload the files as an asset to Ocean protocol. If so, run this

    ```
    python run.py -n grant_data_aggregate
    ```

3. Run the below jobs in any order to create boolean legos (Farmer, MarketMixer, OnChainHistory, VoteTwitterImbalance)

    ```
    python run.py -n wallet_insights
    python run.py -n project_insights
    ```

4. Setup supabase (postgres sql service) to store the insights to a relational DB (to enable faster access)

5. Run the below scripts within supabase SQL editor to create necessary tables

    ```
    scripts/db_scripts.sql
    ```        

6. Finally, run the transformation job to load the data/insights to the above database

    ```
    python run.py -n transform_data_to_db
    ```

## Deployment
-----

These jobs are currently deployed in AWS Batch using [Dockerfile](https://github.com/kikura3/gtclooker-legos/blob/master/Dockerfile)

```
docker build -t gitcoin-legos .
docker run --env-file .env gitcoin-legos python run.py -n grant_data_extract
docker run --env-file .env gitcoin-legos  python run.py -n grant_data_aggregate
docker run --env-file .env gitcoin-legos python run.py -n wallet_insights
docker run --env-file .env gitcoin-legos  python run.py -n project_insights
docker run --env-file .env gitcoin-legos  python run.py -n transform_data_to_db
```