---
sidebar_position: 2
---

# Listing NFTs

This page is going to discuss how to buy List NFTs with Seaport either through 
- the Opensea API 
- through using Seaport's validate function

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

### 2) Create the order with the parameters to generate signature from user
In this case, I'm assuming you are building this application to have other users sign,
so I will show that in this case. If you are just doing this on your own, you can
just have your private key / seed phrase in your .env variables and sign it through
your code without any prompt from a wallet provider.

:::info
Important Definitions:
- **Offer**: The offer is the token that you are trying to *purchase*. It is **not** what you are 
offering to buy the item with. It is the offer from the perspective of the seller.
- **Considerations**: These are the items to exchange for the seller's item. Basically, it is what the
buyer is going to give up.
- **Start_time**: Just the start time of the listing. It has to be at least 15 minutes ahead of when 
you're trying to invoke the function, so I usually try to do at least 20 minutes.
- **End_time**: Time at which listing will end.
- salt: it is a random number generated for entropy. Basically, adds another level of uniqueness
to your transaction.
- **ConduitKey**: This is the specific permissions you're allowing your site or whatever this is being used on to transfer the tokens from that wallet. This is what you see when you first list or sell an item on Openseas. **NOTE:** You should not use the conduitKey I have below. The one I have below is for giving permissions for ANY token, which your users will not like nor should you have in case of exploit.
- **Zone & Zonehash**: These confuse me too; I will update this when I understand them better.
:::

Why do I have three considerations? Well, first, you can have multiple considerations. What does this mean? That means, if i wanted to buy Deadfellaz#7201 from you, then I can give you a list of currencies/NFTs in exchange. That is why this is great! I don't only have to pay you ETH. I could pay you in whichever cryptocurrency and/or NFTs. 

That said, what are my three? Well, my example below is a bit vanilla. In every Seaport listing, **that you want to appear on Opensea**, you have to include the Opensea fee and the Royalty fee. If you don't, the you can still create the Seaport Listing, **but** it will not show up on Opensea. What does this mean? It means, someone can still fulfill that listing/transaction, but they wouldn't find it on Opensea. This is why you can still create your own marketplaces and transactions without Opensea, by using Seaport. Pretty cool eh? Now, I don't think Opensea will list transactions that have multiple considerations, but if you wanted your own listings, then you could do that.

Anyways, the first consideration below is the listing_profit, which is the amount that will go to the seller: Price - OpenseaFee - RoyaltyFee. Then the second consideration is the Opensea Fee (2.5%) and the third is the Royalty fee (x%). If you don't get these mathematically correct, Opensea will not show the transaction. 

:::caution
You have to include the Opensea fee and Royalty Fee of the collection in order to have the listing listed on Opensea.
:::

```jsx title="Create order for signature"
let new_parameters = {
            offerer: seller,
            offer: [
                {
                    itemType: 2,
                    token: contract_address,
                    identifierOrCriteria: token_id,
                    startAmount: "1",
                    endAmount: "1"
                }],
            consideration: [
                {
                    itemType: 0,
                    token: "0x0000000000000000000000000000000000000000",
                    identifierOrCriteria: "0",
                    startAmount: prices.listing_profit,
                    endAmount: prices.listing_profit,
                    recipient: seller 
                },
                {
                    itemType: 0,
                    token: "0x0000000000000000000000000000000000000000",
                    identifierOrCriteria: "0",
                    startAmount: prices.opensea_fee,
                    endAmount: prices.opensea_fee,
                    recipient: "0x8De9C5A032463C561423387a9648c5C7BCC5BC90" 
                },
                {
                    itemType: 0,
                    token: "0x0000000000000000000000000000000000000000",
                    identifierOrCriteria: "0",
                    startAmount: prices.royalty_fee,
                    endAmount: prices.royalty_fee,
                    recipient: royalty_info.royalty_address 
                }],
            startTime: ethers.BigNumber.from((Math.floor(new Date().getTime() / 1000)).toString()),
            endTime: ethers.BigNumber.from((Math.floor((new Date().getTime() / 1000) + 60*60*endTimeSeconds)).toString()),
            orderType: 2,
            zone: "0x004C00500000aD104D7DBd00e3ae0A5C00560C00",
            zoneHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
            salt: "30158852458975423",
            conduitKey: "0x0000007b02230091a7ed01230072f7006a004d60a8d4e71d599b8104250f0000",
            totalOriginalConsiderationItems: ethers.BigNumber.from("3"),
        }
```

### 3) Get the signature from the lister/seller for the above parameters
:::info
**Important**: You need to add a counter parameter to your structure after signing an order on Seaport. They need to be separate before adding it to your order.
:::
```jsx title = "Generate signature from seller/lister
let counter = 0
let signature = await seaport.signOrder(parameters, counter)
parameters["counter"] = counter
return {
    parameters: parameters, 
    signature: signature
}
```

### 4) FINAL STEP: Send the listing to Opensea's API
:::info
**Important**: You have to make sure that you tweak the structure of the parameters before sending it to Opensea. First, you need to convert the `startTime` and `endTime` to Strings. Then, you have to convert the `totalOriginalConsiderationItems` to a Number.
:::
```js title = "Send the listing to Opensea's API"
let api_params = order["parameters"]
api_params.startTime = api_params.startTime.toString()
api_params.endTime = api_params.endTime.toString()
api_params.totalOriginalConsiderationItems = api_params.totalOriginalConsiderationItems.toNumber()
order["parameters"] = api_params
const options = {
    method: 'POST',
    headers: {
    Accept: 'application/json',
    'X-API-KEY': process.env.NEXT_PUBLIC_OPENSEAS_API_KEY,
    'Content-Type': 'application/json'
    },
    body: JSON.stringify(order)
};
fetch('https://api.opensea.io/v2/orders/ethereum/seaport/listings', options)
.then(response => response.json())
.then(response => console.log(response))
.catch(err => console.error(err));
```



## Other Functions

### Get Buyer

```jsx title = "Get the buyer from the provider"
    async function getBuyer() {
        let accounts = await ethereum.request({
            method: "eth_accounts"
        });
        let address = accounts[0].toLowerCase()
        return address
    }

```

### Calculate Fees

```jsx title = "Calculate the fees and total profit"
    function calculatePrices(listing_price, royalty) {
        try {
            let opensea_fee = (listing_price * 0.025).toFixed(18)
            let royalty_fee = (listing_price / (1/(royalty / 100 / 100))).toFixed(18) // 100 cause percent comes in *100
            let listing_profit = ethers.utils.parseEther(String((listing_price - opensea_fee - royalty_fee).toFixed(18)))
            return {
                listing_profit: listing_profit.toString(),
                royalty_fee: ethers.utils.parseEther(String(royalty_fee)).toString(),
                opensea_fee: ethers.utils.parseEther(String(opensea_fee)).toString()
            }
        }
        catch (error) {
            console.log(error)
            return -1
        }

```

## B) List using Seaport's Validation
