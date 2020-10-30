
# CMRESHandler.py
This library provides an Elasticsearch logging appender compatible with the
python standard `logging <https://docs.python.org/2/library/logging.html>`_ library.

The code source is in github at [https://github.com/SHolzhauer/python-elasticsearch-logger](https://github.com/SHolzhauer/python-elasticsearch-logger)

This is a fork of the [original work](https://github.com/cmanaha/python-elasticsearch-logger) by cmanaha.

**Tested against**
* `Elasticsearch 7.3.1` with `Python3.6` 


## Installation
WIP

## Requirements
This library requires the following dependencies
#### Python 2
 - elasticsearch
 - requests
 - enum
#### Python 3
- elasticsearch
- requests
- packaging

## Using the handler
To initialise and create the handler, just add the handler to your logger as follow
```python
from cmreslogging.handlers import CMRESHandler
    handler = CMRESHandler(hosts=[{'host': 'localhost', 'port': 9200}],
                               auth_type=CMRESHandler.AuthType.NO_AUTH,
                               es_index_name="my_python_index")
    log = logging.getLogger("PythonTest")
    log.setLevel(logging.INFO)
    log.addHandler(handler)
```

You can add fields upon initialisation, providing more data of the execution context
```python
from cmreslogging.handlers import CMRESHandler
    handler = CMRESHandler(hosts=[{'host': 'localhost', 'port': 9200}],
                               auth_type=CMRESHandler.AuthType.NO_AUTH,
                               es_index_name="my_python_index",
                               es_additional_fields={'App': 'MyAppName', 'Environment': 'Dev'})
    log = logging.getLogger("PythonTest")
    log.setLevel(logging.INFO)
    log.addHandler(handler)
```

This additional fields will be applied to all logging fields and recorded in elasticsearch

To log, use the regular commands from the logging library

```python
log.info("This is an info statement that will be logged into elasticsearch")
```

Your code can also dump additional extra fields on a per log basis that can be used to instrument
operations. For example, when reading information from a database you could do something like
```python
start_time = time.time()
    database_operation()
    db_delta = time.time() - start_time
    log.debug("DB operation took %.3f seconds" % db_delta, extra={'db_execution_time': db_delta})
```
The code above executes the DB operation, measures the time it took and logs an entry that contains
in the message the time the operation took as string and for convenience, it creates another field
called db_execution_time with a float that can be used to plot the time this operations are taking using
Kibana on top of elasticsearch

## Initialisation parameters
The constructors takes the following parameters:
 - hosts:  The list of hosts that elasticsearch clients will connect, multiple hosts are allowed, for example
```python
    [{'host':'host1','port':9200}, {'host':'host2','port':9200}]
```
 - auth_type: The authentication currently support CMRESHandler.AuthType = NO_AUTH, BASIC_AUTH, KERBEROS_AUTH
 - auth_details: When CMRESHandler.AuthType.BASIC_AUTH is used this argument must contain a tuple of string with the user and password that will be used to authenticate against the Elasticsearch servers, for example ('User','Password')
 - aws_access_key: When ``CMRESHandler.AuthType.AWS_SIGNED_AUTH`` is used this argument must contain the AWS key id of the  the AWS IAM user
 - aws_secret_key: When ``CMRESHandler.AuthType.AWS_SIGNED_AUTH`` is used this argument must contain the AWS secret key of the  the AWS IAM user
 - aws_region: When ``CMRESHandler.AuthType.AWS_SIGNED_AUTH`` is used this argument must contain the AWS region of the  the AWS Elasticsearch servers, for example ``'us-east'``
 - use_ssl: A boolean that defines if the communications should use SSL encrypted communication
 - verify_ssl: A boolean that defines if the SSL certificates are validated or not
 - buffer_size: An int, Once this size is reached on the internal buffer results are flushed into ES
 - flush_frequency_in_sec: A float representing how often and when the buffer will be flushed
 - es_index_name: A string with the prefix of the elasticsearch index that will be created. Note a date with
   YYYY.MM.dd, ``python_logger`` used by default
 - index_name_frequency: The frequency to use as part of the index naming. Currently supports
   `CMRESHandler.IndexNameFrequency.DAILY`, `CMRESHandler.IndexNameFrequency.WEEKLY`,
   `CMRESHandler.IndexNameFrequency.MONTHLY`, `CMRESHandler.IndexNameFrequency.YEARLY`, `CMRESHandler.IndexNameFrequency.NONE` by default the daily rotation
   is used
 - es_doc_type: A string with the name of the document type that will be used ``python_log`` used by default
 - es_additional_fields: A dictionary with all the additional fields that you would like to add to the logs
