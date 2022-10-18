---
sidebar_position: 1
---

# Purchasing NFTs

This page is going to discuss how to buy NFTs either that already exist or through creating your own orders. 

## A) Purchasing Existing Opensea Listings

In order to purchase existing NFT listings through Seaport, there are true important steps.

## Overview

### 1) Pull listing order through **Opensea's API**
You need to pull the exact structure from the Opensea API for ease and because you need the signature. You can't just input the basic information about the listing because without the seller's signature (which already exists when they created the listing), then you can't submit a Opensea transfer request for that NFT 

**You need an Opensea API key. Without, you can't interact with the Opensea API**


### 2) Prompt user to sign to get their signature
This is a simple request prompting through Metamask or any other wallet provider (my favorite is Tally Ho! They are a decentralized wallet provider and will prompt over Metamask without any changes in your code). Without receiving the buyer's signature, then you can submit an order on the behalf of that address. If you are just using one of your wallets, you can use your own private key to generate the signature instead. 

### 3) Create Buy order
You're going to use the buyer's signature that you just asked for through their permission and the Opensea API order to construct the buy order and then submit through the Seaport Documentation.


## Code

### 1) Pull listing order through **Opensea's API**

```jsx title="Retrieve Listing Order Through Opensea's API" 
async function getListingOrder(contract_address, token_id) {
    var contract = req.query.contract;
    var token_id = req.query.tokenID;
    const options = {
        method: 'GET',
        headers: { Accept: 'application/json', 'X-API-KEY': process.env.API_KEY }
    };

    fetch('https://api.opensea.io/api/v1/assets?token_ids=' + token_id + '&order_direction=desc&asset_contract_address=' + contract_address + '&limit=20&include_orders=true', options)
        .then(response => response.json())
        .then(response => {
            return
                {
                    wyvern_orders: response.assets[0].sell_orders,
                    seaport_sell_orders: response.assets[0].seaport_sell_orders
                }
        })
        .catch(err => {
            return{ msg: "error", err: err }
        });
}
```

### Create buy order and generate signature from buyer, and submit

```jsx title="Create and buy"
export async function buyNFT(contract_address, token_id) {

    const buy = async() => {
        try {
            // create the seaport instance with the provider
            const provider = new ethers.providers.Web3Provider(window.ethereum);
            const seaport = new Seaport(provider);
            // get the buyer and order
            let buyer = await getBuyer()
            let res = await getListingOrder(contract_address, token_id)
            if (res["seaport_sell_orders"] === null) {
                console.log("Error getting orders, may not exist for NFT")
                return
            }
            // get the order for that listing
            let full_order = res["seaport_sell_orders"][0]["protocol_data"]
            // calculate the tip
            // MAKE SURE TO CHANGE TIP ADDRESS IN THIS FUNCTION
            let new_tip = await getTip(full_order["parameters"])
            if (new_tip === "E") { 
                console.log("error with generating tip)
                return 
            } // return if can't get price
            // now execute buy order
            if (typeof seaport !== "undefined" && full_order) {
                const { executeAllActions: executeAllFullfillActions} = await seaport.fulfillOrder({
                    order: full_order,
                    tips: new_tip,
                    accountAddress: buyer
                })
                const transaction = await executeAllFullfillActions()
                if (transaction[code] === 4001) {
                    console.log("They denied transaction")
                    return
                }
            }
        } catch(error) {
            console.log("Error in purchasing NFT")
            console.log(error)
        }
    }

    async function doAll() {
        await buy()
    }
    doAll()
}
```

### Add Tip (Optional)
 You can add a tip or extra fee in case you wanted to. This is an optional parameter and not neccessary when submitting an order. If you want to add a tip, it is already included in the code above. 

```jsx title="Add Tip to Buy Order"
async function getTip(full_order) {
        // calculate the tip pricing 
        const user_address = "0x1bacd1d3cee7ca249f9197782b9097b75707bcfb"
        //get the full real price without fees
        let real_price = getRealPrice(full_order["consideration"])
        if (real_price === "E") {return "E"}
        // if you have a price, calculate tip
        let tip_percent = ethers.BigNumber.from("100")
        let price = ethers.BigNumber.from(real_price)
        let tipping_price = price.div(tip_percent)
        // create the tip
        let new_tip = [{
            itemType: 0, //ETH
            token: "0x0000000000000000000000000000000000000000", 
            identifier: "0",
            amount: tipping_price,
            recipient: user_address
        }]
        return new_tip
    }
```