# Monitoring Cryptocurrencies with Bash


I bought into the crypto scene but was concerned about the price of crypto dropping dramatically as it does, so I made an incredibly basic bash script to monitor the prices of crypto using the CoinGecko API. This could be used for instance to show the current price on Bitcoin in PolyBar or DWM

```sh
#!/bin/sh

COINS=$1
CURRENCIES=$2

curl -X GET "https://api.coingecko.com/api/v3/simple/price?ids=${COINS}&vs_currencies=${CURRENCIES}" -H  "accept: application/json"
echo ""
```

Save this to a file `fetch-crypto.sh` and make it executable with `chmod +x fetch-crypto.sh`. Then we can call it as

```sh
$ ./fetch-crypto.sh bitcoin aud
{"bitcoin":{"aud":72630}}
```

If you want more currencies or coins, you can use a comma separated list

```sh
$ ./fetch-crypto.sh bitcoin,ethereum aud
{"bitcoin":{"aud":72773},"ethereum":{"aud":2246.25}}

$ ./fetch-crypto.sh bitcoin,ethereum aud,usd
{"ethereum":{"aud":2246.25,"usd":1742.93},"bitcoin":{"aud":72773,"usd":56467}}
```

Then if you want to actually extract data from the JSON, you could use the [jq](https://stedolan.github.io/jq/) tool. 
For instance, if I wanted to make a script that printed out the price of Bitcoin in AUD, I would write

```
$ fetch_crypto bitcoin aud | jq '.bitcoin.aud'
72773
```

I've also turned this into a rebinding and made it output the price in a notification 

```
$ notify-send "$(./fetch-crypto.sh bitcoin aud | jq '.bitcoin.aud')"
```

