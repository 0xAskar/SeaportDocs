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


### 2) Generate seller's signature
 

### 3) Send to Opensea's API 


**You need an Opensea API key. Without, you can't interact with the Opensea API**


## Code

### 1) Pull listing order through **Opensea's API**

```jsx title="Retrieve Listing Order Through Opensea's API" 

```

### Create buy order and generate signature from buyer, and submit

```jsx title="Create and buy"

```
