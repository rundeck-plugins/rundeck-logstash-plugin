# Rundeck Logstash Plugin

This is a simple Rundeck streaming Log Writer plugin that will pipe all log output to a Logstash server by writing Json to a TCP port. *For Rundeck version 1.6.0 or later.*

# Installation

Copy the `LogstashPlugin.groovy` to your `$RDECK_BASE/libext/` directory for Rundeck.

Enable the plugin in your `rundeck-config.properties` file:

    rundeck.execution.logs.streamingWriterPlugins=LogstashPlugin

# Configure Rundeck

The plugin supports these configuration properties:

* `host` - hostname of the logstash server
* `port` - TCP port to send JSON data to

You can update the your framework/project.properties file to set these configuration values:

in `framework.properties`:

    framework.plugin.StreamingLogWriter.LogstashPlugin.port=9700
    framework.plugin.StreamingLogWriter.LogstashPlugin.host=localhost

or in `project.properties`:

    project.plugin.StreamingLogWriter.LogstashPlugin.port=9700
    project.plugin.StreamingLogWriter.LogstashPlugin.host=localhost

# Configure Logstash

Add a TCP input that uses Json format data.  Here is an example `rundeck-logstash.conf`:

    input {
      
      tcp {
        debug => true 
        format => "json"
        host => "localhost"
        message_format => "%{message}"
        mode => server
        port => 9700
        #ssl_cacert => ... # a valid filesystem path (optional)
        #ssl_cert => ... # a valid filesystem path (optional)
        #ssl_enable => ... # boolean (optional), default: false
        #ssl_key => ... # a valid filesystem path (optional)
        #ssl_key_passphrase => ... # password (optional), default: nil
        #ssl_verify => ... # boolean (optional), default: false
        tags => ["rundeck"]
        type => "rundeck"
      }

    }

    output { 
      stdout { }

      elasticsearch { embedded => true }
    }


# Start Logstash

Use the config file when starting logstash.

    java -jar ../logstash-1.1.13-flatjar.jar agent -f rundeck-logstash.conf -- web --backend elasticsearch://localhost/
