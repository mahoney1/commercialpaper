# Tutorial 3/3: "Putting it altogether: deploy your contract to a real Commercial Paper IBM Blockchain Platform network built using Ansible"


## Introduction

Having successfully upgraded the Commercial Paper smart contract ( on the'Commerce' network) in [tutorial 2](https://github.com/mahoney1/commercialpaper/blob/master/tutorial2-queries-commercial-paper-smart-contract-ibm-blockchain-vscode-extension.md), MagnetoCorp, DigiBank and Hedgematic - all part of a blockchain consortium - wish to deploy the new contract on their network in IBM Blockchain Platform in IBM Cloud. In this tutorial, you'll automatically provision this network, then deploy the smart contract. The last part will show you how client applications from any organisation, can interact with the ledger data - ie 'business as usual'.

One aim of this tutorial, is to show how the [IBM Blockchain Platform Ansible collection](https://github.com/IBM-Blockchain/ansible-collection/blob/master/README.md) can be used to provision an IBM Blockchain Platform 2.5 network - your `papercontract` smart contract is also installed on each peer in this network. For the purposes of this tutorial, all 3 organisation's nodes will be deployed to the same 30-day trial cluster. You can get [information on how to get this cluster](https://cloud.ibm.com/docs/blockchain?topic=blockchain-ibp-saas-pricing#ibp-saas-pricing-free) and use it free for 30 days. For full details on getting the trial SaaS environment, see the [outlined steps here](https://cloud.ibm.com/docs/blockchain?topic=blockchain-ibp-saas-pricing#ibp-saas-pricing-free-howto)

The ansible collection is fully-scripted for you; all you have to do is: get your free cluster, set up an IBM Blockchain Platform service, then: 'press the button'. If you want to read more on IBM Blockchain Ansible collections, including a tutorial - check it out [here](https://ibm-blockchain.github.io/ansible-collection/) 

<img src="/img/tutorial3/ibp-cons-summary.png" title="IBP Nodes and instantiated contract" alt="IBP Nodes and instantiated contract" />

Once provisioned, you will interact with the contract remotely using: 

   + the IBM Blockchain Platform VS Code extension and 
   + application clients (provided for you). 

The last part of the tutorial will see you using a HTML 5 client app to render asset reports showing the full lifecycle/history of an asset. 


## Pre-requisites

You will need an IBM Kubernetes cluster in IBM Cloud and you'll need to create an IBM Blockchain Platform service instance. You can avail of the 30-day free cluster trial (see Step 1 below), or use an existing IBM Kubernetes cluster in IBM Cloud that is a supported level as described [here](https://cloud.ibm.com/docs/blockchain?topic=blockchain-ibp-console-overview#ibp-console-overview-supported-cfg). 


## Steps

### Step 1: Create your cluster, IBM Blockchain Service instance and Service Credentials

1. Log into IBM Cloud and create your free cluster `mycluster` in IBM Cloud -you can [preview the IBM Blockchain Platform in IBM Cloud at no charge for 30 days](https://cloud.ibm.com/registration?target=%2Fcatalog%2Fservices%2Fblockchain).

    - Create the cluster with the name `mycluster` - it will take a couple of hours to have the IBM Kubernetes cluster provisioned and available (`Status: Green` in your Cloud environment). 

2. Once the cluster is available,  create your IBM Blockchain Platform service instance via the IBM Cloud Catalog](https://cloud.ibm.com/catalog/services/blockchain-platform)  eg name: `Blockchain-Platform-ibp`

    <img src="/img/tutorial3/create-service.png" title="Create the IBP service instance" alt="Create IBP service" />

3. When prompted, link your new IBM Blockchain Platform service instance that you created to the cluster `mycluster` when prompted. 

    <img src="/img/tutorial3/link-cluster.png" title="Link the IBP service instance" alt="Link to cluster" />

    On the top left you'll also see a `Service Credentials` option - this will enable us to create credentials, so our ansible collection script can use these to connect to your IBM Blockchain Platform instance. We'll tell you how to get your credentials from IBM Blockchain Platform instance there so you can provide these to the Ansible collection you will use so Ansible can authenticate and provision the 'Commerce' 3-org network).

4. Click 'Next' to proceed. It will take a little while to provision the IBM Blockchain Platform service, but eventually it will come back with a button to 'Launch the IBM Blockchain Platform Console' - click on this.

    <img src="/img/tutorial3/launch-ibp.png" title="Launch the IBP console" alt="Launch IBP" />

5. Finally, you will see the IBM Blockchain Platform console 'Getting Started' page - click on this button and you will get IBM Blockchain Platform console - no nodes as of yet.

    <img src="/img/tutorial3/ibp-console.png" title="Getting started - IBP console" alt="Console Get Started" />

    OK. At this point, we have our IBM Blockchain Platform service with no nodes/components in the network. The next step is to provision our 'three-organisation' network in this environment. This is the job of the IBM Blockchain Platform ansible collection.

6. Return to your IBM Blockchain Service instance under the IBM Kubernetes resource list at https://cloud.ibm.com/resources and click on the IBM Blockchain Platform service you created.

    You will now create some service credentials, which are crucial for the Ansible collection to be able to connect to your IBM Cloud based blockchain service instance.

7. Click on `Service Credentials` menu item on the left and click on `New Credentials` button

    <img src="/img/tutorial3/create-credentials.png" title="Create Service Credentials" alt="Create Credentials" />

8. When prompted - create credentials with the name `ibp-creds1` and click `Add` - your credentials have been created. Now you can return to the browser tab with the IBM Blockchain Platform Console open.

    <img src="/img/tutorial3/credentials-added.png" title="Credentials added" alt="Added Credentials" />


9. Open up a terminal and change directory to the `commercialpaper` repo (eg in your $HOME directory) - then change to subdirectory `ansible`

    ```
    cd $HOME/commercialpaper
    cd ansible/
    ```

10. Copy the credentials using the copy icon on the right - then open a terminal window and create a file called `creds.txt` and paste in the Cloud credentials you copied into this file (approx 6 lines in JSON format) to this file - save the file. You'll use these credentials in the next section to build your blockchain network in IBM Blockchain Platform.


### Step 2. Locate the Ansible collection and launch the ansible builder for the 'Commerce' network.

In this section, you will launch the ansible builder to provision a network for our three organisations - each will have one peer and one CA. There is also an Ordering organisation and of course an ordering service (RAFT).  Lastly, you will provision your `papercontract@0.0.2` smart contract this network.

1. You'll need to clone the IBM Blockchain Platform Ansible collection repo  and change directory to you home directory, clone it and change directory to that:

    ```
    cd
    git clone https://github.com/IBM-Blockchain/ansible-collection.git
    cd ansible-collection
    ```

2. You will need pre-requisite images for the IBM Blockchain Platform Ansible collection itself (as outlined in the [Installation pages](https://ibm-blockchain.github.io/ansible-collection/installation.html) . For this - we conveniently built a Docker image that contains **all** of the pre-requisites from a `Dockerfile` in the `docker` subdirectory of the cloned collection. Use the sequence below to build the image (see also, comment below about new IBP ansible image). Please don't forget the '.' at the end of the command:

    ```
    cd docker
    docker build --tag myansible:latest .
    ```
    
    ** As of September 2020 - there is now a docker image `ibp-ansible` on Docker Hub, that does the above for you ^^ - it is an Ansible collection for building Hyperledger Fabric networks using the IBM Blockchain Platform: more info at  https://registry.hub.docker.com/r/ibmcom/ibp-ansible  
    
    <img src="/img/tutorial3/docker-build.png" title="Docker build" alt="Ansible docker build" />

    The above build process, will take approx 15mins or so to complete, please note. Once complete, you will have a docker image (checked using `docker images`) that's tagged  `myansible:latest`.  If you're using/downloading the `ibp-ansible` image, then replace the tagged image name in step 6 below.

    We're now ready to build our IBM Blockchain Platform network.

    <img src="/img/tutorial3/tagged-build.png" title="Docker build complete" alt="Ansible build tagged" />


3. Change directory to the `$HOME/commercialpaper/ansible/tutorial` directory

    ```
    cd $HOME/commercialpaper/ansible/tutorial
    ```

4. Next - you need to edit 3 files `org1-vars.yml`, `org2-vars.yml` and `org3-vars.yml` - these 3 organisation variable files need the `api_endpoint` and `api_key` lines edited - to use your service instance credentials that you created earlier. In particular - lines 5 and 7 in the files - see image (lines greyed for security purposes). Note that you can leave the `api_secret` variable on line 8 as it is : `xxxxxx` - as this is not used. All of the other variables in the file **can also remain exactly as they are** - but take note of the values anyway.

    <img src="/img/tutorial3/edit-orgvariables.png" title="Edit organisation variables" alt="Edit the organisation variables" />

    So, edit files  `org1-vars.yml`, `org2-vars.yml` and `org3-vars.yml` in turn, so that the first 4 lines (beginning with `api_endpoint`) are identical in each file. 

5. The last file to edit is `ordering-org-vars.yml` - the ordering service variables YAML file. This also needs the same api information - carry out **exactly** the same changes to this fourth file,  as you did in the previous step. The first 4 lines of this file are identical to those added in the previous step.

    You've now completed your edits. Next, launch the ansible to build the 3 org network. This will run a set of ansible playbooks.

6.  Ensure you are still in the same `tutorial` directory to run this command - as this is the mount point (-v) into the container instance launched below:. Also ensure the file `tutorial.sh` has execute permissions (it should have,  after cloning earlier but your system may have new file permission defaults).  

    ```
    docker run --rm -v "$PWD:/tutorial" myansible:latest /tutorial/tutorial.sh
    ```

    You will shortly see it build the IBM Blockchain Platform. 

    <img src="/img/tutorial3/run-ansible-script.png" title="Run the ansible build" alt="Run the IBP Ansible build" />

    Now is a good time to go to your 'empty' IBM Blockchain Platform Console you launched earlier - best to do this using a browser inside your virtual machine (for Step 2 below), as you can see the nodes being added in the console (Note:  you will need to toggle between 'Nodes' and 'Channels' (ignoring messages about unable to find wallets) and back to 'Nodes' to see the very latest status of a node being provisioned.

    <img src="/img/tutorial3/provisioned-ibp-env.png" title="IBP nodes being provisioned" alt="IBP nodes being provisioned" />

    Your IBP Blockchain Platform environment should now be provisioned - you will be at the point where the smart contract is installed on all peers - you will instantiate it manually. Note also that there are a list of exported node and identity JSON files. These include identities that you can use to import in the IBM Blockchain Platform console or import as appropriate into the IBM Blockchain Platform VS Code extension wallets,  to connect to your blockchain network. One very useful feature of the IBP VS Code extension is that you can also import your Cloud 'Commerce' network nodes under 'Fabric Environments' by connecting to it using your IBM Cloud id and password  (add an IBM Blockchain Platform Fabric environment for an IBM Cloud service instance) - you'll be prompted for the login id and password to authenticate to the cloud. You'll then be asked to choose the IBM Blockchain Platform service instance and prompted to import all of the components from your SaaS environment.  For the purposes of this tutorial, we won't need to do this: we've provisioned application identities using Ansible, and we'll simply enroll these from the VS Code extension.

    <img src="/img/tutorial3/list-of-jsons.png" title="List of JSON files" alt="List of JSON files" />


### Step 3. Instantiate the smart contract on the Commerce network in IBM Cloud

1.  In your IBM Blockchain Platform console, click on the `Wallets` icon on the left then click 'Add Identity'. Upload each of `MagnetoCorp Admin`, `DigiBank Admin` and `Hedgematic Admin`  in turn, using the `Upload json` option. Ensure you DON'T choose the `CA Admin` json file :-) when doing this step.

    <img src="/img/tutorial3/add-identities.png" title="Add identities" alt="Add identities" />

2. Next, click on the `Nodes` icon, then click on each peer (in turn),  and select each of their respective Admin identities,  to associate with that Organisation's peer.  Eg `DigiBank Peer` should be associated with `DigiBank Admin`  and so on.

    <img src="/img/tutorial3/associate-identity.png" title="Associate identity" alt="Associate identity" />

3. Click on the  `Smart Contracts` icon on the left. It will show that under 'Smart Contracts' the `papercontract@0.0.2` is already installed on the peers. Click on the `ellipsis` icon on the right and select `Instantiate Contract`

    <img src="/img/tutorial3/instantiate-contract.png" title="Instantiate contract" alt="Instantiate contract" />

4. When prompted, select the channel `mychannel` for `papercontract@0.0.2`, accept the defaults by clicking 'Next' but for the function name enter the name `instantiate` as the function to call. It will take a minute or two to instantiate. In this step, we choose to show the operational concepts of instantiation - if you prefer, you can do this instantiation from the Ansible script (given that it installs the smart contract on all peers - FYI).


### Step 4. Connect to the Commerce Network on IBM Blockchain Platform in IBM Cloud

1. Return to the IBM Blockchain Platform VS Code extension in your development by clicking on the extension's icon.

2. Next, import the organisational Gateway definitions (JSON)) that were created by Ansible. Click on the '+' under `Fabric Gateways` and select to 'Create a gateway from a connection profile'

    <img src="/img/tutorial3/create-ext-gateway.png" title="Create Gateway from profile" alt="Create Gateway from profile" />

3. Give it a name of `MagnetoCorp_GW` when prompted - and browse to your `tutorial` subdirectory to add the file `MagnetoCorp Gateway.json` - you'll see a 'successfully added' notification bottom right.

4. Repeat steps 2 and 3 above for `DigiBank_GW` and `Hedgematic_GW` and select the respective files to import. The end result is three gateways.

    <img src="/img/tutorial3/three-gateways-added.png" title="Three Gateways added" alt="Three Gateways added" />

5. Next, add some wallets and enrole some application identities. Click on the '+' button under the `Fabric Wallets` view and choose to 'Create a new wallet and add identity'.

    <img src="/img/tutorial3/create-ext-wallet.png" title="Create new wallet" alt="Create new wallet" />

6. Give the wallet a name of 'MagnetoCorp' when prompted and give the identity a name of 'magnus' 

7. Enter an MSP id of 'MagnetoCorpMSP' for this identity

8. Choose to  'select a gateway' and provide 'an enrollment id and secret'

    <img src="/img/tutorial3/enrol-identities.png" title="Enrol application identity" alt="Enrol application identity" />

9. Select the `MagnetoCorp_GW` gateway and provide the enrol id / secret of `magnus` and `demopw`  to enrol your identity - you'll get a message the wallet was created  and the identity was successfully added to it.

10. Repeat steps 5 through 9 for `DigiBank_GW` and `Hedgematic_GW` . The MSP ids to provide are `DigiBankMSP` and `HedgematicMSP` and the identities are `david` and `helen` respectively. Note that the enrol secret for each identity is also `demopw` .

    After adding, if you expand these 3 wallets you should see the following structure, showing the enrolled application identities in their respective organisational wallets.

    <img src="/img/tutorial3/wallet-summary.png" title="Create new wallet" alt="Wallet Structure" />

    Well done - you're now ready to test your connection to your 'Commerce network' and the first thing to try is to see if you can see the smart contract transaction list. At this point, FYI - there is no ledger data on the blockchain.

11. Click on the `MagnetoCorp_GW` gateway under `Fabric Gateways` and choose the wallet `MagnetoCorp` - there is just one identity in it, so it will connect automatically with the identity `magnus` added earlier. You will see the channel `mychannel` . Expand the channel, then the smart contract 'twisty' to reveal the list of transaction functions - these include all of the query functions you added in tutorial 2. The list may take a little while to appear (approx 20 secs).

    OK - we're now ready to execute transactions as each of the organisations and after creating the commercial paper lifecycle of transactions, then we can do some simple queries. 


### Step 5. Perform the issue, buy, and redeem transaction lifecycle to create data on the ledger

Let’s create some transactions, invoked as the different identities, to create a history of transactions on the ledger. The sequence is:

   + Issue a paper as "MagnetoCorp." 
   + Buy the paper as "DigiBank," the new owner.
   + Buy the paper as "Hedgematic," the changed owner.
   + Redeem the paper at face value, as existing owner "Hedgematic," with MagnetoCorp as the original issuer.

#### Transaction 1: Execute an `issue` transaction as MagnetoCorp

1. From the IBM Blockchain Platform VS Code sidebar panel, you should already be connected as application id `magnus` to the `MagnetoCorp_GW`. Expand the `mychannel` twisty, then  expand `papercontract@0.0.2` and highlight the `issue` transaction.

    <img src="/img/tutorial3/issue-txn-saas.png" title="Issue trxn as MagnetoCorp" alt="Issue trxn as MagnetoCorp" />

2. Right-click  on `issue and choose `Submit Transaction`. A pop-up window should appear at the top.
  
3. When prompted, copy and paste the following parameters (incl. double-quotes) **inside** the existing square brackets "[]" and hit ENTER. Whem prompted, just hit ENTER to accept defaults for the next two prompts:  `Transient data` entry, and  `default  peer targeting policy`. **Make sure there are no trailing ' ' spaces** in your input to transactions:

    ```
   "MagnetoCorp","0001","2020-05-31","2020-11-30","5000000"
    ```
  
4. Check the message (in the **Output** pane) indicating that this transaction was successfully submitted. Note: the FIRST transaction only, may take about 5-10 seconds to complete - please be patient.

    <img src="/img/tutorial3/issue-success.png" title="Confirm issue success" alt="Confirm issue success" />
  
5. Lastly, disconnect from the `MagnetoCorp_GW` Gateway by clicking the 'Fabric Gateways' pane title, then click on the "disconnect" icon.

#### Transaction 2. Execute a `buy` transaction as DigiBank

1. Click once on the `DigiBank_GW` Gateway - it will connect with identity 'david' - the only one in this wallet.
  
2. Expand the `mychannel` twisty and then the `papercontract@0.0.2` twisty, in turn to see the transaction list.

3. Highlight, then right-click, the "buy" transaction and right-click "Submit Transaction." A pop-up window will appear.

4. When prompted, copy and paste the following parameters (incl. the double-quotes) **inside** the square brackets, `[]`. Hit ENTER to accept defaults for the next two prompts:  `Transient data` entry, and  `DEFAULT peer targeting policy`:
  
    ```
    "MagnetoCorp","0001","MagnetoCorp","DigiBank","4900000","2020-05-31"
    ```

    <img src="/img/tutorial3/buy-success.png" title="Confirm buy success" alt="Confirm buy success" />
  
5. Once again, check the message (in the output pane) indicating that this transaction was successfully submitted.

6. Disconnect from the `DigiBank_GW` Gateway from the `Fabric Gateways` view.

#### Transaction 3. Execute another `buy` transaction - this time, as Hedgematic

1. Connect to the `Hedgematic_GW` and wallet `Hedgematic` - you will connect as `helen` to execute another `buy` transaction as Hedgematic. 
  
2. Expand the `mychannel` twisty and then the `papercontract@0.0.2` twisty, in turn.

3. Highlight, then right-click, the "buy" transaction and right-click "Submit Transaction." A pop-up window will appear.

4. When prompted, copy and paste the following parameters (incl. the double-quotes) **inside** the square brackets, `[]`. Hit ENTER to accept defaults for the next two prompts:  `Transient data` entry, and  `DEFAULT peer targeting policy`:

    ```
    "MagnetoCorp","0001","DigiBank","Hedgematic","4930000","2020-06-15"
    ```

    <img src="/img/tutorial3/2nd-buy-success.png" title="Confirm Hedgematic buy success" alt="Confirm Hedgematic buy success" />
 
5. Again, check the results for a successful transaction in the `Output` pane.

#### Transaction 4. Execute a `redeem` transaction as Hedgematic -- c. six months later

Months later in this commercial paper's lifecycle, the current owner (Hedgematic) wishes to **redeem** the commercial paper "0001" at face value and get a return on the investment outlay. Typically, a client application would perform this task with a valid identity. For testing purposes, we can use the IBM Blockchain VS Code extension to do this.


1. Still connected to the Hedgematic gateway,  highlight the `redeem` transaction and right-click ... "Submit Transaction." A pop-up window will appear.
  
2. When prompted, copy/paste the following parameters (incl. the double-quotes) **inside** the square brackets, `[]`, and hit ENTER, then hit ENTER again (to skip Transient Data and Peer Targeting):
  
    ```
    "MagnetoCorp","0001","Hedgematic","2020-11-30"
    ```

3. Check the message (in the output pane) indicating that this `redeem` transaction was successfully submitted.

4. Disconnect from the `Hedgematic` Gateway using the disconnect icon (click the Fabric Gateways title to see the icon).

Optional: create another set of 4 transactions (issue, buy, buy, redeem) for another commercial paper: replace the paper id `0001` with `0002` in the parameter list above - change some of the values for offer and buy price too. This will mean you have more ledger data (2 asset lifecycles) to perform different queries in the section below.


Well done! You've completed a full commercial paper transaction lifecycle; now its time to do some queries on the data


### Step 6. Perform queries of ledger data in the VS Code extension

In this section, you'll test your queries pull data from the real Commerce network on IBM Cloud. This is a primer, before you go to connect up a sample javascript query application in the next section. You've tested these queries in development, so it should take a couple of minutes to try these out.

1. Still connected as `Hedgematic` identity `helen` - right click on the `queryHist` function and choose `Evaluate Transaction`. Provide the following parameters inside the `[ ]` brackets:

    ```
    "MagnetoCorp","0001"
    ```

    You should see a JSON array with the history of transactions for commercial paper asset `0001`.

    <img src="/img/tutorial3/asset-history-query.png" title="Query asset history" alt="Query asset history" />

2. Next, right-click on `queryOwner` transaction and `Evaluate Transaction`. Provide just one parameter to this (ie. 'owner name'). 

    ```
    "MagnetoCorp"
    ```

    <img src="/img/tutorial3/ownership-query.png" title="Query ownership" alt="Query ownership" />

    In our screenshot, we added a 2nd optional asset below: if you added a 2nd asset, you will see that "MagnetoCorp" has ownership of both papers (as both were redeemed by Hedgematic).

    OK cool - well done. The last part of this tutorial is to connect up our query application to do some asset history reporting. We have provided a query client application (Javascript) in the github repo, that will query the ledger and export it to JSON file. The goal is to render this asset history in a HTML 5 browser app for local reporting.

### Step 7. Connect up the query client application (Node.JS) to render asset history in a HTML browser app

1. Locate the client script `clientqueryapp.js` and peruse it to see how it connects to the IBM Cloud 'Commerce' network using the following parameters:

    - Gateway (via the `connection.json` from one of the three organisation's in our environment)
    - Identity (one of the three application identities we created)
    - Organisation (Use the name, to locate the corresponding wallet path).

    The asset history query in particular will also write results to a local file called `results.json` - used by the browser app later.

2. Install the Node.JS dependencies using the following command 

    ```
    npm install
    ```

    (Ignore any warning message for now)

3. Launch the Node.js application as follows: you need to provide two parameters (Organisation name and identity name):

    ```
    node clientqueryapp.js DigiBank  david
    ```

    <img src="/img/tutorial3/clientapp-query.png" title="Query History from client" alt="Query History from client" />

    You should see the asset history in a JSON array as output from your application query as identity david. Great. Note it has also written a file `results.json`

4. Now lets use our browser client to render the asset history (above) in HTML 5. The supplied file `index.html` uses the free tool [Tabulator](http://tabulator.info/) to render report in very powerful, extensible and interactive ways. You don't have to install anything, you just launch the HTML file as a browser app in Firefox (currently only supported in FF please note) as follows:

    ```
    firefox index.html
    ```

    You should see a report of your asset data (its dependent on `results.json` existing above) and here you'll see a history of asset `0001`


    <img src="/img/tutorial3/html-asset-report.png" title="Render Asset History report" alt="Render Asset History report" />


### Step 8. Edit the client application to render asset history deltas report in HTML 5 browser app.

In this section, edit the `clientqueryapp.js`  and add a piece of code to get deltas only from the history records - this represents a smaller payloads when considering voluminous asset histories.

1. Edit `clientqueryapp.js` and at line 105 (just before the comment line  `// queryOwner the OWNER of a commercial paper`

2. Paste in the following code segment and ensure you save your javascript client file.

    ```
    console.log('Calling queryDeltas to get the deltas of Commercial Paper instance 0001');
    console.log('========================================================================');

    // QUERY the deltas of a commercial paper providing it the Issuer/paper number combo below
    let deltaResponse = await contract.evaluateTransaction('queryDeltas', 'MagnetoCorp', '0001');
    deltaResponse = "data = '" + deltaResponse.toString().replace(/\\"/g,'') + "'";

    // parse the response sent back from contract -> client app
    let file2 = await fs.writeFileSync('deltas.json', deltaResponse, 'utf8');
    console.log('the query DELTAS response is ' + deltaResponse);
    console.log(' ');

    console.log('Transaction complete.');
    console.log(' ');
    ```

    This code calls the `queryDeltas` transaction function you added when enhancing the smart contract earlier, and will write the results out to a file called `deltas.json` - you'll use another HTML client to get this Deltas report.

3. Now invoke the upgraded client app just as you had done before:

    ```
    node clientqueryapp.js DigiBank  david
    ```

4. Now using the provided `deltas.html` provided in the Github repo, render the results in `deltas.json` in your browser app - you'll provide a parameter to this file:

    ```
    firefox deltas.html?myParam="MagnetoCorp:0001"
    ```

    <img src="/img/tutorial3/html-deltas-report.png" title="Render Deltas History report" alt="Render Asset History report" />

    You should see a report of your asset data (its dependent on `deltas.json` existing above) and here you'll see a history of deltas for asset `0001`. The third and fourth column are provided as padding for the report (they're not necessarily 'changed' between asset updates FYI).

#### Optional Exercise: Gain insights into your blockchain transactions in the IBM Blockchain Platform console.

As a further optional exercise, you can return to your IBM Blockchain Platform Console and browse your transaction history via the IBM Blockchain Platform console. 

5. Simply click on the `Channels` icon - then select `mychannel` and review the transaction history by clicking on a block and then clicking on the transaction id listed, to see the contents of the transaction. You'll be able to easily reconcile this data,  with the transactions you performed earlier. Feel free to browse the transaction data in your block list.

Well done! You've completed the full set of 3 tutorials.


## Summary


Lets recall what you've completed during the course of this tutorial series:

  + developed your Papercontract smart contract, then enhanced it by adding functionality (different types of queries)
  + provisioned a three organisation network* (MagnetoCorp, DigiBank and Hedgmatic) instantiated a smart contract and provisioned identities
  + connected up to your network, using the IBM Blockchain Platform extension
  + transacted as different organisations, to complete a 'typical' commercial paper asset lifecycle
  + connected up your local client application, to be able to query as an identity from any one of the three organizations
  + finally, as DigiBank, pulled the history of a commercial paper asset on the ledger, that is shared across the three organizations. You'll see the complete lifecycle of transactions that you performed, rendered in HTML 5. The same applies with the 2nd report, reporting deltas only.


*Normally of course, each organisation would operate their own nodes/components as part of the network, and join the consortium. They would also have their own client application needs etc. etc.

For further study, I firstly recommend you to look at the new [IBM Blockchain Foundation Developer course and badge](https://cognitiveclass.ai/courses/ibm-blockchain-foundation-dev) that allows developers to demonstrate their understanding of Hyperledger Fabric development. The course makes full use of the new tutorials embedded in the IBM Blockchain Platform VS Code extension itself, and the resulting badge rewards developers for completing these. Note that the badge is fully accredited by IBM, and is part of the Acclaim open badge platform that recognizes professional success.

Also - take a look at the IBM Developer [code patterns](https://developer.ibm.com/patterns/category/blockchain/) to try out further sample use cases.
