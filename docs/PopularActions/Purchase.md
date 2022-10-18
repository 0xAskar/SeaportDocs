---
sidebar_position: 1
---

# Purchasing NFTs

This page is going to discuss how to buy NFTs either that already exist or through creating your own orders. 

## A) Purchasing Existing Opensea Listings

In order to purchase existing NFT listings through Seaport, there are true important steps.

### 1) Pull listing order through **Opensea's API**
You need to pull the exact structure from the Opensea API for ease and because you need the signature. You can't just input the basic information about the listing because without the seller's signature (which already exists when they created the listing), then you can't submit a Opensea transfer request for that NFT 

**You need an Opensea API key. Without, you can't interact with the Opensea API**


### 2) Prompt user to sign to get their signature
This is a simple request prompting through Metamask or any other wallet provider (my favorite is Tally Ho! They are a decentralized wallet provider and will prompt over Metamask without any changes in your code). Without receiving the buyer's signature, then you can submit an order on the behalf of that address. If you are just using one of your wallets, you can use your own private key to generate the signature instead. 

### 3) Create Buy order
You're going to use the buyer's signature that you just asked for through their permission and the Opensea API order to construct the buy order and then submit through the Seaport Documentation.
