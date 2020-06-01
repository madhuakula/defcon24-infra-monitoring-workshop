# Backing up Elasticsearch using Curator (15 minutes)

### What is Curator ?

> Elasticsearch Curator helps you curate or manage your indices. 
> Supports a variety of actions from delete a snapshot to shard allocation routing.
> This is an additional software bundle that you install on top of Elasticsearch.

### Curator Installation

Curator is already installed in the virtual machine

* To install curator

```
sudo apt-get install python-pip -y
sudo pip install elasticsearch-curator
```

> Note: There is an alternate method using apt-get install as well.

```
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb http://packages.elastic.co/curator/3/debian stable main" | sudo tee -a /etc/apt/sources.list.d/curator.list
sudo apt-get update && sudo apt-get install python-elasticsearch-curator -y
```

## Hands On

* Create a directory to keep all the backups

```
sudo mkdir -p /var/backups/elasticsearch/
```

* Change the ownership of the directory to

```
sudo chown elasticsearch:elasticsearch -R /var/backups/elasticsearch/
```

* Add the backups path to the elasticsearch configuration `sudo vi /etc/elasticsearch/elasticsearch.yml`

```
path.repo: /var/backups/elasticsearch/
```

* Then restart the elasticsearch service

```
sudo service elasticsearch restart
```

* Stop the second Elasticsearch node that we created earlier

* Before we create backups/restores, we need to create a repo in elasticsearch
* Run the below command to create a `backup` repository in elasticsearch

```
curl -XPUT 'http://localhost:9200/_snapshot/backup' -d '{
"type": "fs",
"settings": {
"location": "/var/backups/elasticsearch/",
"compress": true
}
}'
```

* To snapshot an index called `apache-logs` into a repository `backup`

```
curator snapshot --name=apache_logs_snapshot --repository backup indices --prefix apache-logs
```

* To see all snapshots in the `backup` repository

```
curator show snapshots --repository backup
```

* To restore a snapshot from curator

```
curl -XPOST 'http://localhost:9200/_snapshot/backup/apache_logs_snapshot/_restore'
```

### Restore some sample logs for creating advanced dashboards (hands on)

* Unzip the log sample called `filebeat.tar.gz` and move the logs to backups directory

```
sudo tar -xvf /home/ninja/log-samples/filebeat.tar.gz -C /var/backups/elasticsearch/
```

* Check the snapshot name

```
curator show snapshots --repository backup
```

* Then use curator to restore the logs to the elasticsearch

```
curl -XPOST 'http://localhost:9200/_snapshot/backup/filebeat_logs_snapshot/_restore'
```

* Check the head plugin to see the updated logs from import
