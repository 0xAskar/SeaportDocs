---
sidebar_position: 2
---

# Listing NFTs

This page is going to discuss how to buy List NFTs with Seaport either through the Opensea API or through using Seaport's validate function

## A) Listing with Seaport and Openea's API

If you don't want to pay a fee, then listing with their API is better. The obvious downside is requiring an api key from Opensea.

Listing on Opensea is a bit tricky using Seaport. The reason being is because you need to create the order through Seaport first which has a slightly different syntax then what is required to send to Opensea's API to list. The details will be explained below.

## Overview

### 1) Create listing order structure through Seaport
You need to create the Seaport order that is correct for the protocol. That structure in itself is slightly different than what you will need to send to the api.

### 2) Generate seller's signature
 After creating the order through Seaport, prompt the user or generate your own signature through your private key. Without a signature, you can't authorize a listing, else, anyone could do it for you?

### 3) Send to Opensea's API 
This structure will inherit most of the information from the Seaport structure plus the signature, with a few changes as you will see below. With the API, you will be able to send it.

:::tip
You need an Opensea API key. Without it, you can't interact with the Opensea API
:::

Okay, but on the overview page, you said you don't need an API key? Why are you lying to me Askar!? I didn't. The Seaport Protocol doesn't require an API key, but using the API can help send requests to list on Opensea. However, you can still list without using an api key.


## Code

### 1) Pull listing order through **Opensea's API**

```jsx title="Retrieve Listing Order Through Opensea's API" 
    // there some extra functions, please scroll to the bottom for them
    const provider = new ethers.providers.Web3Provider(window.ethereum);
    const seaport = new Seaport(provider);
    // get seller and prices
    let seller = await getBuyer()
    let royalty_info = await getRoyaltyInfo(contract_address)
    let prices = calculatePrices(listing_price, royalty_info.royalty_fee)
    if (prices === -1) {return}

```

### Create buy order and generate signature from buyer, and submit

```jsx title="Create and buy"
    async function getBuyer() {
        let accounts = await ethereum.request({
            method: "eth_accounts"
        });
        let address = accounts[0].toLowerCase()
        return address
    }
```




## Other Functions

### Get Buyer

```jsx title = "Get the buyer from the provider"

```