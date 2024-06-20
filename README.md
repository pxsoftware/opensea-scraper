OpenSea asset scraper written in Python

opensea.exe is for mysterious cloudflare bypassing


```python
from multiprocessing import cpu_count
from multiprocessing.pool import ThreadPool
import subprocess
import requests
import time
import json
import os

collection = "https://opensea.io/collection/remilio-babies"
cursor = "xx"

collection = collection.split("/collection/")[1]
arguments = []
page = 1

if not os.path.exists(collection):
    os.mkdir(collection)

def download_url(args):
    t0 = time.time()
    url = args[0]
    file_name = args[1]
    with open(file_name, 'wb') as f:
        req = requests.get(url)
        f.write(req.content)
    return (url, time.time() - t0)

def download_parallel(args):
  cpus = cpu_count()
  results = ThreadPool(cpus - 1).imap_unordered(download_url, args)
  for result in results:
    print('url:', result[0], 'time (s):', result[1])

while True:
    j_obj = json.loads(subprocess.check_output(['opensea.exe', collection, cursor]).decode())
    try:
        for item in j_obj["data"]["collectionItems"]["edges"]:
            item = item["node"]
            tokenId = item["tokenId"]
            image = item["displayImageUrl"].replace("?w=500", "?w=1000")
            file_name = "{}/{}.png".format(collection, tokenId)
            arguments.append([image, file_name])
        print("Scraping page {}... [count: {}]".format(page, len(arguments)))
        page += 1
        if j_obj["data"]["collectionItems"]["pageInfo"]["hasNextPage"]:
            cursor = j_obj["data"]["collectionItems"]["pageInfo"]["endCursor"]
        else:
            break
    except Exception as e:
        print(str(e))

download_parallel(arguments)
```
