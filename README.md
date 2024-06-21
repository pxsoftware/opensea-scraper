OpenSea image scraper written in Python

opensea.exe for mysterious cloudflare bypassing


```python
from multiprocessing import cpu_count
from multiprocessing.pool import ThreadPool
import subprocess
import requests
import time
import json
import os

collection = "https://opensea.io/collection/cryptopunks"

root = "G:\\nft-collections"
collection = collection.split("/collection/")[1]
arguments = []
page = 1

def download_url(args):
    t0 = time.time()

    while True:
        if os.path.exists(args[1]):
            return "already exists...", time.time() - t0

        try:
            base = json.loads(subprocess.check_output(['opensea.exe', "lookup", args[0][0], args[0][1], args[0][2]]).decode())
            try:
                if "404" in base["errors"][0]["message"]:
                        return "404...", time.time() - t0
            except:
                pass
            url = base["data"]["nft"]["imageUrl"].replace("?w=500", "?w=1000")
        except Exception as e:
            time.sleep(1)
            continue

        with open(args[1], 'wb') as f:
            req = requests.get(url)
            f.write(req.content)
        return url, time.time() - t0

def download_parallel(args):
  cpus = cpu_count()
  results = ThreadPool(cpus - 1).imap_unordered(download_url, args)
  for result in results:
    print('url:', result[0], 'time (s):', result[1])

total_count = json.loads(subprocess.check_output(['opensea.exe', "info", collection, "None", "None"]).decode())["data"]["collectionItems"]["totalCount"]
collection_name = json.loads(subprocess.check_output(['opensea.exe', "info", collection, "None", "None"]).decode())["data"]["collectionItems"]["edges"][0]["node"]["collection"]["name"]
address = json.loads(subprocess.check_output(['opensea.exe', "address", collection, "None", "None"]).decode())["data"]["collection"]["representativeAsset"]["assetContract"]["address"]
identifier = json.loads(subprocess.check_output(['opensea.exe', "address", collection, "None", "None"]).decode())["data"]["collection"]["defaultChain"]["identifier"]
if not os.path.exists("{}/{}".format(root, collection_name)):
    os.mkdir("{}/{}".format(root, collection_name))

for i in range(0, total_count+1):
    arguments.append([(str(i), address, identifier), "{}/{}/{}.png".format(root, collection_name, i)])

download_parallel(arguments)
```
