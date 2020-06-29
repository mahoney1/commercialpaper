## Commercial Paper tutorials (2020 - series of 3 tutorials)


In this repo there are tutorial docs and supporting files for the  **IBM Blockchain Platform VS Code Extension Commercial Paper tutorial series**  - the current editions located on IBM Developer at https://developer.ibm.com/series/blockchain-running-enhancing-commercial-paper-smart-contract/ are out of date and will be replaced on IBM Developer itself, in due course (probably during July 2020).

The old IBM Developer based series have been revised (new Github tutorials below) and those old tutorials no longer work.

## Tutorials summary

1. Running and executing a commercial paper locally in development, using the IBM Blockchain Platform VS Code extension
2. Developing and enhancing the smart contract, adding a range of query transaction functions, both simple and advanced. Then testing these work locally.
3. Building a pilot IBM Blockchain Platform 2.5  Commercial Paper ('commerce') network in IBM Cloud, consisting of three organisations: MagnetoCorp, DigiBank and Hedgematic. You will use the [IBM Blockchain Platform ansible collection](https://ibm-blockchain.github.io/ansible-collection/). In the github repo you cloned earlier, you will see a subdirectory called `ansible` - this contains the ansible playbooks to build the network in IBM Blockchain Platform in IBM Cloud. Once built, you will then:

    - connect up to your remote blockchain network "('Commerce') in IBM Cloud using the VS Code extension
    - test the new functionality added to the smart contract all 3 organisations will use.  
    - connect up your client applications to be able to interact with the network and finally
    - use client browser apps to render HTML 5 reports: asset history and the deltas history.

## Diagrams

![Overview](/img/main/reduced-overview.png)


A summary of the end-to-end cycle is shown here:

![End-to-End Flow; Dev -> IBM Blockchain Platform in IBM Cloud](/img/main/dev-overview.png)
    
    
A typical local dev lifecycle looks something like this:

![Local Dev Cycle](/img/main/typical-dev.png)





## Out of date series links.

Running the Commercial Paper sample, using different blockchain IDs - https://developer.ibm.com/tutorials/run-commercial-paper-smart-contract-with-ibm-blockchain-vscode-extension/

Adding Query capability - https://developer.ibm.com/tutorials/queries-commercial-paper-smart-contract-ibm-blockchain-vscode-extension/

Adding Deltas capability - https://developer.ibm.com/tutorials/add-further-query-functionality-using-the-ibp-vscode-extension/

Please msg me DIRECT (not on main channel) on Rocketchat (mahoney1) - chat.hyperledger.org if you have any issues.

Paul O'Mahoney (Rocketchat: mahoney1).
Artifacts for Commercial Paper tutorials (2020)
