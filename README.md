# fluent-plugin-azurefunctions
[Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview) (HTTP Trigger) output plugin for Fluentd. The plugin aggregates semi-structured data in real-time and writes the buffered data via HTTPS request to HTTP Trigger Function.

## Installation
```
$ gem install fluent-plugin-azurefunctions
```

## Configuration

### Azure Functions
You need to create a function (HTTP Trigger) if you don't already have. But you need to have an function app first of all that hosts the execution of your functions in Azure. Once you have an function app, you can create a function. Here are great instructions:

 * [Create your first Azure Function](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-azure-function)
 * [Azure Functions developer reference](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference)

A quick-start HTTP trigger function sample is included under examples/function-csharp:
```
examples/function-csharp
  - function.json    (Configurations for Azure Functions)
  - project.json     (Configurations for package dependencies)
  - run.csx          (Function code)
```

### Fluentd - fluent.conf

```
<match azurefunctions.**>
    @type azurefunctions
    endpoint  AZURE_FUNCTION_ENDPOINT   # ex. https://<accountname>.azurewebsites.net/api/<functionname>
    function_key AZURE_FUNCTION_KEY     # ex. aRVQ7Lj0vzDhY0JBYF8gpxYyEBxLwhO51JSC7X5dZFbTvROs7uNg==
    key_names key1,key2,key3,key4,key5
    add_time_field true
    time_field_name mytime
    time_format %s
    localtime true
    add_tag_field true
    tag_field_name mytag
</match>
```

 * **endpoint (required)** - Azure Functions Endpoint URL
 * **function\_key (required)** - Azure Functions API Key
 * **key\_names (optional)** - Key names in in-comming records to post to Azure functions HTTP Trigger functions. Each key needs to be separated by a comma. If key_names not specified, all in-comming records are posted. If in-comming records contain the same key names as the ones specified in time_field_name and tag_field_name, their values are replaced by the values of time_field_name and tag_field_name
 * **add\_time\_field (optional)** - Default:true. This option allows to insert a time field to record
 * **time\_field\_name (optional)** - Default:time. This is required only when add_time_field is true
 * **localtime (optional)** - Default:false. Time record is inserted with UTC (Coordinated Universal Time) by default. This option allows to use local time if you set localtime true. This is valid only when add_time_field is true
 * **time\_format (optional)** -  Default:%s. Time format for a time field to be inserted. Default format is %s, that is unix epoch time. If you want it to be more human readable, set this %Y%m%d-%H:%M:%S, for example. This is valid only when add_time_field is true.
 * **add\_tag\_field (optional)** - Default:true. This option allows to insert a tag field to record
 * **tag\_field\_name (optional)** - Default:tag. This is required only when add_time_field is true


## Configuration examples

fluent-plugin-azurefunctions adds **.rid** attribute which is UUID format and any other attributes of record automatically. In addition, it adds **time** and **tag** attributes if **add_time_field** and **add_tag_field** are true respectively. Below are two types of the plugin configurations - Default and All options configuration.

### (1) Default Configuration (No options)
<u>fluent.conf</u>
```
<source>
    @type forward
    @id forward_input
</source>

<match azurefunctions.**>
    @type azurefunctions
    endpoint https://yoichikademo.azurewebsites.net/api/HttpTriggerFunction
    function_key aRVQ7Lj0vzDhY0JBYF8gpxYyEBxLwhO51JSC7X5dZFbTvROs7uNg==(dummy)
</match>
```

The plugin write all records in in-comming event stream out to Azure Functions:
```
# Generating test event using fluent-cat
echo ' { "key1":"value1", "key2":"value2", "key3":"value3", "key4":"value4"}' | fluent-cat azurefunctions.msg

# HTTP POST request body to Azure Functions
{
    "payload": '{"key1":"value1", "key2":"value2", "key3":"value3", "key4":"value4", "time":"1479741633", "tag":"azurefunctions.msg"}'
}

mytime=1479741633


```


### (2) Configuration with All Options
<u>fluent.conf</u>
```
<source>
    @type forward
    @id forward_input
</source>

<match azurefunctions.**>
    @type azurefunctions
    endpoint https://yoichikademo.azurewebsites.net/api/HttpTriggerFunction
    function_key aRVQ7Lj0vzDhY0JBYF8gpxYyEBxLwhO51JSC7X5dZFbTvROs7uNg==(dummy)
    key_names key1,key2
    add_time_field true
    time_field_name mytime
    time_format %s
    localtime true
    add_tag_field true
    tag_field_name mytag
</match>
```

The plugin write only records that are specified by key_names in in-comming event stream out to Azure Functions:
```
# Generating test event using fluent-cat
echo ' { "key1":"value1", "key2":"value2", "key3":"value3", "key4":"value4"}' | fluent-cat azurefunctions.msg

# HTTP POST request body to Azure Functions
{
    "payload": '{"key1":"value1", "key2":"value2", "time":"1479741633", "tag":"azurefunctions.msg"}'
}
```

## Tests
### Running test code
```
$ git clone https://github.com/yokawasa/fluent-plugin-azurefunctions.git
$ cd fluent-plugin-azurefunctions

# edit CONFIG params of test/plugin/test_azurefunctions.rb
$ vi test/plugin/test_azurefunctions.rb

# run test
$ rake test
```

### Creating package, running and testing locally
$ rake build
$ rake install:local

# running fluentd with your fluent.conf
$ fluentd -c fluent.conf -vv &

# generate test event
$ echo ' { "key1":"value1", "key2":"value2", "key3":"value3", "key4":"value4"}' | fluent-cat azurefunctions.msg


## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/yokawasa/fluent-plugin-azurefunctions.

## Copyright

<table>
  <tr>
    <td>Copyright</td><td>Copyright (c) 2016- Yoichi Kawasaki</td>
  </tr>
  <tr>
    <td>License</td><td>Apache License, Version 2.0</td>
  </tr>
</table>

