# Diskover FileSystem Crawler

Welcome to Diskover FS Crawler

What if you are running low on disk space. You need to free some up, by finding files that are a waste of space and deleting them (or moving them to archive). How do you find the right files to delete?

Diskover helps to index files from your local file system or nfs mounts.
It crawls your file system and indexes files and adds to [Elasticsearch](https://www.elastic.co) or [Amazon Elasticsearch Service](https://aws.amazon.com/elasticsearch-service/). It is written in Python and uses multi-threading to speed up indexing crawl times. The indexed files are bulk added and streamed into Elasticsearch while the crawl is running allowing you to visualize the data in [Kibana](https://www.elastic.co/products/kibana) without having to wait until the crawl is finished.

## Screenshots

### Dashboards
![kibana-screenshot](docs/kibana-dashboard-screenshot.png)
![kibana-screenshot](docs/kibana-dashboard-dupes-screenshot.png)
### Dupes Graph
![kibana-screenshot](docs/kibana-graph-dupes-screenshot.png)
### Hardlinks Graph
![kibana-screenshot](docs/kibana-graph-hardlinks-screenshot.png)


# Installation Guide

## Download diskover

```sh
git clone https://github.com/shirosaidev/diskover.git
cd diskover
```

## Update diskover

```sh
cd diskover
git pull
```


# User Guide

## Requirements

* `Linux or Mac OS X` (tested on Mac OS X 10.11.6 and Ubuntu 16.04, have not tested on Windows)
* `Python 2.7.` (tested on Python 2.7.10 and 2.7.12, have not tested on Python 3)
* `Python client for Elasticsearch`
* `GNU find command` (most likely you already have this)
* `Elasticsearch` (local or AWS ES service, tested on Elasticsearch 5.3.0)
* `Kibana` (tested on Kibana 5.3.0)

### Optional Install

* [X-Pack](https://www.elastic.co/downloads/x-pack) (for graphs, reports, monitoring and http auth)


## Getting Started

You need to have at least **Python 2.7.** and have installed Python client for Elasticsearch using `pip`:

```sh
python --version
```

```sh
pip install elasticsearch
```

If you don't have pip, you can install it with:

```sh
sudo easy_install pip
```

You also need GNU `find` command which is used for building the directory queue list. It needs to be in your PATH, which is usually in `/usr/bin`:

```sh
which find
```

Start Diskcover FS crawler as root user with:

```sh
cd /path/you/want/to/crawl
sudo python /path/to/diskover.py
```

A successfull crawl should look like this:

```

   ___       ___       ___       ___       ___       ___       ___       ___
  /\  \     /\  \     /\  \     /\__\     /\  \     /\__\     /\  \     /\  \
 /::\  \   _\:\  \   /::\  \   /:/ _/_   /::\  \   /:/ _/_   /::\  \   /::\  \
/:/\:\__\ /\/::\__\ /\:\:\__\ /::-"\__\ /:/\:\__\ |::L/\__\ /::\:\__\ /::\:\__\
\:\/:/  / \::/\/__/ \:\:\/__/ \;:;-",-" \:\/:/  / |::::/  / \:\:\/  / \;:::/  /
 \::/  /   \:\__\    \::/  /   |:|  |    \::/  /   L;;/__/   \:\/  /   |:\/__/
  \/__/     \/__/     \/__/     \|__|     \/__/    v1.0.7     \/__/     \|__|
                                      https://github.com/shirosaidev/diskover


[2017-05-03 17:58:31] [status] Connecting to Elasticsearch
[2017-05-03 17:58:31] [info] Checking for ES index: diskover-2017.04.22
[2017-05-03 17:58:31] [warning] ES index exists, deleting
[2017-05-03 17:58:31] [info] Creating ES index
Crawling: [100%] |████████████████████████████████████████| 7960/7960
[2017-05-03 17:58:40] [status] Finished crawling
[2017-05-03 17:58:40] [info] Directories Crawled: 7960
[2017-05-03 17:58:40] [info] Files Indexed: 344
[2017-05-03 17:58:40] [info] Elapsed time: 9.41380310059
```


## Diskover CLI arguments

```
usage: diskover.py [-h] [-d TOPDIR] [-m MTIME] [-s MINSIZE] [-t THREADS]
                   [-i INDEX] [-n] [--dupesindex] [-v]

optional arguments:
  -h, --help            show this help message and exit
  -d TOPDIR, --topdir TOPDIR
                        Directory to start crawling from (default: .)
  -m MTIME, --mtime MTIME
                        Minimum days ago for modified time (default: 30)
  -s MINSIZE, --minsize MINSIZE
                        Minimum file size in MB (default: 5)
  -t THREADS, --threads THREADS
                        Number of threads to use (default: 2)
  -i INDEX, --index INDEX
                        Elasticsearch index name (default: from config)
  -n, --nodelete        Do not delete existing index (default: delete index)
  --dupesindex          Create duplicate files index (default: don't create)
  -v, --verbose         Run more verbose (default: not verbose)
```


## Config file

Diskcover will read a local config file `diskover.cfg`. **It needs to be in the same directory as `diskover.py`**.

Here you can exclude any directories and files you don't want to index separated by `,` (**spaces after comma are treated as part of file/directory name**).

Elasticsearch hostname (endpoint), port and index name are also set here. If you are using AWS ES, set `aws = True` and `port = 443` and set the host to the endpoint in your AWS ES console. If you are running Elasticsearch on your localhost or lan, comment out or set `aws = False` and the default port is `9200`. **If you installed Elasticsearch X-Pack, uncomment and set user/password for http auth**.

Lines beginning with `;` are comments and ignored by Diskover.

```
[excluded_dirs]
dirs = .snapshot,DO_NOT_DELETE,Databases

[excluded_files]
files = Thumbs.db,.DS_Store,._.DS_Store,.localized,desktop.ini

[elasticsearch]
;aws = False
;host = search-diskover-es-cluster-eg3yztrvzb6qucroyyjk2vokza.ap-northeast-1.es.amazonaws.com
;port = 443
host = localhost
port = 9200
; http auth for X-Pack, uncomment the below two lines if you installed
;user = elastic
;password = changeme
indexname = diskover-2017.04.22
```

## Speeding up crawl times

Diskover skips empty directories and will only index files that are older than `modified time` and larger than `minimum file size` from the command line options. Excluding certain files/directories will help speed up crawl times as well. **Running with verbose logging will increase crawl times**.

For example, if you wanted to find all the old files that are larger than 10MB that haven't been modified in more than 6 months, you could run Diskover with:

```sh
cd /path/you/want/to/crawl
sudo python /path/to/diskover.py -m 180 -s 10
```

You could also speed up the crawl by running multiple Diskover `diskover.py` processes and bulk loading into the same `diskover-<name>` index in Elasticsearch. Below is a diagram showing this example.

![diskover-diagram](docs/diskover-diagram.png)

## Benchmarks

Here are some benchmarks running on my macbook pro, this includes time to crawl my local filesystem and index in Elasticsearch (in seconds) using the default of 2 threads and single `diskover.py` running. The files were all 1KB.

```
[2017-04-23 10:02:58] [info] Directories Crawled: 10001
[2017-04-23 10:02:58] [info] Files Indexed: 10000
[2017-04-23 10:02:58] [info] Elapsed time: 16.2423961163

[2017-04-23 10:46:24] [info] Directories Crawled: 100001
[2017-04-23 10:46:24] [info] Files Indexed: 100000
[2017-04-23 10:46:24] [info] Elapsed time: 167.5496099
```


# Elasticsearch

## Indices

Diskover creates an index with the name from the config file or from the cli option `-i`. **If an existing index exists with the same name, it will be deleted and a new index created, unless `-n or --nodelete` cli argument is used**. 

If you are doing crawls every week for example, you could name the indices diskover-2017.04.09, diskover-2017.04.16, diskover-2017.04.23, etc. **Index names are required to be `diskover-<string>`**.

### Append data to existing index

If you are running concurrent `diskover.py` processes you will need to use the `-n or --nodelete` cli argument to append  data to an existing index. See above diagram for example.

### Duplicate files index

An index for duplicate files can also be created using the `--dupesindex` cli argument **after all crawls are finished**. If you named your index `diskover-2017.05.03`, the index for duplicate files will be named `diskover_dupes-2017.05.03`. See above diagram for example.

## Generated fields

Diskover creates the following fields :

|         Field        |                Description                  |                    Example                  |
|----------------------|---------------------------------------------|---------------------------------------------|
| `last_modified`      | Last modification date                      | `1389220808`                                |
| `last_access`        | Last access date                            | `1482435694`                                |
| `last_change`        | Last change date                            | `1389220808`                                |
| `indexing_date`      | Indexing date                               | `1492612283`                                |
| `filesize`           | File size in bytes                          | `50502536`                                  |
| `filename`           | Original file name                          | `"mypic.png"`                               |
| `extension`          | Original file name extension                | `"png"`                                     |
| `path_parent`        | Parent path name of file                    | `"/tmp/dir1/dir2"`                          |
| `path_full`          | Actual real path name                       | `"/tmp/dir1/dir2/mypic.png"`                |
| `owner`              | Owner name                                  | `"cpark"`                                   |
| `group`              | Group name                                  | `"staff"`                                   |
| `hardlinks`          | Hardlink count                              | `1`                                         |
| `inode`              | Inode number                                | `652490`                                    |
| `filehash`           | MD5 hash of file                            | `3a6949b4b74846a482016d0779560327`          |

## Filehash

In order to speed up crawl times, a MD5 hash for each file is made from combining the strings `filename+filesize+last_modified` and hashing that string, rather than the contents of the file. This seems to be a fast high-level way to create a hash of the file. **You should run the md5 command on the files to compare their hashes before you delete the dupes**.

## Kibana

For the index pattern use `diskover-*`. For the duplicate files use `diskover_dupes-*`. **Make sure the `Index contains time-based events` box is `unchecked`** when you create index patterns.

### Diskover Dashboard

To use the Diskover dashboard (screenshot), import the saved objects file `export.json` into Kibana for the dashboard visualizations. In Kibana go to `Management > Saved Objects > Import`.

If nothing is showing in the dashboard, go to `Management > Index Patterns > diskover-*` and then hit the `refresh icon`.

### Kibana Field Formatting

This will help make the dashboard easier to read like in the screenshot for filesize and dates. In Kibana go to `Management > Index Patterns > diskover-*`. In the `Fields tab` click the `edit icon` under controls column for `filesize` field. Change the format to `bytes` and click `Update Field`. For `access_time, modified_time, and change_time`, edit the fields and change the format to `date` and set the format pattern to `MM-DD-YYYY, HH:mm` and click `Update Field`.

### Kibana Search Filters

Once you have imported the `export.json` into Kibana, in Kibana's `Discover` page click `open`. There will be a few saved searches in there to help you filter for old files and find duplicate files.

Here are some filter examples:

* `last_modified:[now-5y TO now-3M]` filters files that haven't been modified in over 3 months and less than 5 years
* `last_modified:[now-5y TO now-6M] AND last_access:[now-5y TO now-6M]` filters files that haven't been modified or accessed in over 6 months and less than 5 years
* `last_modified:[now-5y TO now-1y] AND last_access:[now-5y TO now-1y]` filters files that haven't been modified or access in over 1 year and less than 5 years
* `extension:(jpg OR gif OR png OR tif OR tiff OR dpx OR exr OR psd OR bmp OR tga)` filters for image files
* `extension:(aif OR iff OR m3u OR m4a OR mid OR mp3 OR mpa OR wav OR wma)` filters for audio files
* `extension:(asf OR avi OR flv OR m4v OR mov OR mp4 OR mpg OR rm OR vob OR wmv)` filters for video files
* `extension:(cache OR tmp OR temp OR bak OR old)` filters for temp files
* `extension:(7z OR deb OR gz OR pkg OR rar OR rpm OR tar OR zip OR zipx)` filters for compressed files

## X-Pack

### Graphs

To create the graphs in the screenshots above you need to install [X-Pack](https://www.elastic.co/downloads/x-pack). After X-Pack is installed, edit `diskover.cfg` for http auth credentials since X-Pack adds http auth to Elasticsearch and Kibana.

#### Dupes graph

index pattern: `diskover_dupes-*`
verticies field source #1: `filehash` set to `50 max terms per hop`
verticies field source #2: `inode` set to `50 max terms per hop`
verticies field source #3: `path_parent` set to `50 max terms per hop`
settings: uncheck `Significant links` and set `Certainty to 1`
search filter: `hardlinks: 1`

#### Hardlinks graph

index pattern: `diskover-*`
verticies field source #1: `path_parent` set to `50 max terms per hop`
verticies field source #2: `inode` set to `50 max terms per hop`
settings: uncheck `Significant links` and set `Certainty to 1`
search filter: `hardlinks: >1`


# License

```
This software is licensed under the Apache 2 license, quoted below.

Copyright 2017 Chris Park

Licensed under the Apache License, Version 2.0 (the "License"); you may not
use this file except in compliance with the License. You may obtain a copy of
the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations under
the License.
```
