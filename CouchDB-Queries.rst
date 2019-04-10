CouchDB Queries
===============

This lab instruction is built on top of Part 2 of the VSCode Lab from the `Blockchain Immersion Workshop <http://www.ibm.biz/bc-immersion>`_ Please go through that lab experience first before going through this lab.

Adding Query Files
##################

**1.** Within the editor perspective of VSCode, add in a few folders within the ``contract`` folder. You can do this by clicking on the `New Folder` button (second from the left) within your workplace. Add in a new folder called ``META-INF``

**PICTURE**

**2.** After you have added a new folder, create another folder called ``statedb``. Within ``statedb`` create another folder named ``couchdb``. Finally, within the ``couchdb`` folder, add another file called ``indexes``.

**PICTURE** 

**3.** Now, you should 4 new folders within your contract folder. The path should be
::

  contract -> META-INF -> statedb -> couchdb -> indexes
  
**4.** Within the ``indexes`` folder, create a new file called ``issuerindex.json``. You can create a new file by clicking on the ``New File`` button (first from the left)

**PICTURE**

**5.** Below, paste in the following code into our ``inssuerindex.json`` file
::

  {
    "index": {
        "fields": ["issuer"]
    },
    "ddoc": "issuerIndexDoc",
    "name": "issuerIndex",
    "type": "json"
  }

**6.** Make sure you save this file, by either doing ``control + s`` or ``File -> Save``

**PICTURE**

**7.** Within the ``indexes`` folder, create an ``ownerIndex.json`` file and paste in the code below
::

  {
    "index": {
        "fields": ["owner"]
    },
    "ddoc": "ownerIndexDoc",
    "name": "ownerIndex",
    "type": "json"
  }
  
Make sure you save before moving forward.

**8.** Once you have both ``issuerIndex.json`` and ``ownerIndex.json`` file made, go ahead and make ``currentStateIndex.json`` as our third file. You can paste in the code found below
::

  {
    "index": {
        "fields": ["currentState"]
    },
    "ddoc": "currentStateIndexDoc",
    "name": "currentStateIndex",
    "type": "json"
  }
  
Make sure you save this file as well before continuing on. 

Modifying Smart Contract
########################

**1.** Within our ``lib/papercontract.js``` file, add this code block below at ``line 171`` for the ability to actual query our couchdb
::

    /**
     * Query by Issuer
     *
     * @param {Context} ctx the transaction context
     * @param {String} issuer commercial paper issuer
    */
    async queryByIssuer(ctx, issuer) {

        let queryString = {
            "selector": {
                "issuer": issuer
            },
            "use_index": ["_design/issuerIndexDoc"]
        }

        let queryResults = await this.queryWithQueryString(ctx, JSON.stringify(queryString));
        return queryResults;

    }

    /**
     * Query by owner
     *
     * @param {Context} ctx the transaction context
     * @param {String} owner commercial paper owner
    */
    async queryByOwner(ctx, owner) {

        let queryString = {
            "selector": {
                "owner": owner
            },
            "use_index": ["_design/ownerIndexDoc"]
        }

        let queryResults = await this.queryWithQueryString(ctx, JSON.stringify(queryString));
        return queryResults;

    }
    
     /**
     * Query by current state
     *
     * @param {Context} ctx the transaction context
     * @param {String} currentState current state number of the commercial paper. Refer to paper.js for state values
    */    
    async queryByCurrentState(ctx, currentState) {

        let state = parseInt(currentState);

        let queryString = {
            "selector": {
                "currentState": state
            },
            "use_index": ["_design/currentStateIndexDoc"]
        }

        let queryResults = await this.queryWithQueryString(ctx, JSON.stringify(queryString));
        return queryResults;

    }

    /**
     * Query by Issuer
     *
     * @param {Context} ctx the transaction context
     * @param {String} issuer commercial paper issuer
    */
    async queryAll(ctx) {

        let queryString = {
            "selector": {}
        }

        let queryResults = await this.queryWithQueryString(ctx, JSON.stringify(queryString));
        return queryResults;

    }

    /**
     * Evaluate a queryString
     *
     * @param {Context} ctx the transaction context
     * @param {String} queryString the query string to be evaluated
    */    
    async queryWithQueryString(ctx, queryString) {

        console.log("query String");
        console.log(JSON.stringify(queryString));

        let resultsIterator = await ctx.stub.getQueryResult(queryString);
        
        let allResults = [];

        while (true) {
            let res = await resultsIterator.next();

            if (res.value && res.value.value.toString()) {
                let jsonRes = {};

                console.log(res.value.value.toString('utf8'));

                jsonRes.Key = res.value.key;

                try {
                    jsonRes.Record = JSON.parse(res.value.value.toString('utf8'));
                } catch (err) {
                    console.log(err);
                    jsonRes.Record = res.value.value.toString('utf8');
                }

                allResults.push(jsonRes);
            }
            if (res.done) {
                console.log('end of data');
                await resultsIterator.close();
                console.info(allResults);
                console.log(JSON.stringify(allResults));
                return JSON.stringify(allResults);
            }
        }

    }
    
Make sure you save this file before moving on.
 
**Explain Each Query**
 
**2.** Click on the ``package.json`` file within your ``contract`` folder. We need to modify the version to ``0.0.3`` so that we can upgrade our existing chaincode. Please notice was ``lines 2 & 3`` should look like in our ``package.json`` file.
 ::
 
  "name": "papercontract",
  "version": "0.0.3",
  
Make sure you save this file before continuing on.

**3.** Jump to our IBM Blockchain Platform Extension. Click on the ``gear`` button in the bottom left. Then, click on ``Command Palatte``. Once your get the option to enter a command type this in below
::

  >IBM Blockchain Platform: Package a Smart Contract Project
  
**4.** Select ``contract`` when it asks you what workplace folder to package up

**5.** You'll notice that there was a new ``papercontract@0.0.3`` in our ``Smart Contract Packages`` pane.

**PICTURE**

**6.** Once you have a new smart contract package, go ahead and install the chaincode on our peer by clicking on ``+ Install`` within our ``Local Fabric Ops`` pane. 

**7.** It'll ask you a series of questions before actually installing the smart contract
::

  Choose a peer to install the smart contract on: peer0.org1.example.com
  Choose which package to install on the peer: papercontract@0.0.3 Packaged
  
Hit enter once you have those selections correctly filled out. 

**8.** You will notice that the smart contract is installed once you see it in the installed section of our ``Local Fabric Ops`` pane.

**PICTURE**

**9.** Now that we have successfully installed our smart contract, let's actually upgrade the smart contract on the channel. You can do this by clicking on the ``Channels`` button in the ``Local Fabric Ops`` pane and then right clicking on ``mychannel``. Then select ``Upgrade Smart Contract``.  

**10.** It will ask you a series of questions before actually upgrading the smart contract
::

  Select the instantiated smart contract to upgrade: papercontract@0.0.2
  Select the smart contract version to perform and upgrade with: papercontract@0.0.3 Installed
  Hit enter
  
**11.** You will know your operation was successful if you see ``papercontract@0.0.3`` in the ``Instantiated`` section of the ``Local Fabric Ops`` pane.

**PICTURE**

Submitting Queries
##################


  
  


 
 
 
