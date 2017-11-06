Sample setup for HBase Solr NRT Indexer

Setup the hbase table

```
hbase shell << EOF
create 'sample_table', {NAME => 'data'}
disable 'sample_table'
alter 'sample_table', {NAME => 'data', REPLICATION_SCOPE => 1}
enable 'sample_table'
put 'sample_table', 'row1', 'data','value'
put 'sample_Table', 'row2', 'data','value2'
```

Generate a solr instance directory that we can modify

```
solrctl instancedir --generate $HOME/hbase_collection_config
```

The assumption is we're not using Sentry, so we can just copy the schema.xml over

```
cp schema.xml $HOME/hbase_collection_config/conf
```

Create the solr index

```
solrctl instancedir --create hbase_collection_config $HOME/hbase_collection_config
solrctl collection --create hbase_collection -s 2 -c hbase_collection_config
```

Copy the contents of the morphlines.conf file into Cloudera Manager

```
Key-Value Store Indexer service > Configuration > Category > Morphlines > Morphlines File
```

On the server running the lily indexer, switch to the root user

```
su -
```

Change directory in to process directory for the lily indexer
```
cd /var/run/cloudera-scm-agent/process/`ls -1 /var/run/cloudera-scm-agent/process/ | grep 'HBASE_INDEXER$' | sort -n | tail -1`
```

Register the lily indexer using the hbase keytab and jaas.conf file

```
HBASE_INDEXER_OPTS="$HBASE_INDEXER_OPTS -Djava.security.auth.login.config=jaas.conf" \
hbase-indexer add-indexer \
--name myIndexer \
--indexer-conf /home/cloudera/hbase-solr-nrt/morphline-hbase-mapper.xml \
--connection-param solr.zk=mstr1.feetytoes.com,mstr2.feetytoes.com,mstr3.feetytoes.com/solr \
--connection-param solr.collection=hbase_collection \
--zookeeper mstr1.feetytoes.com:2181,mstr2.feetytoes.com:2181,mstr3.feetytoes.com:2181
```
