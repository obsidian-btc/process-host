# ProcessHost

Lightweight process host for single threaded, input bound processes. Ideal for TCP services (including HTTP), message buses, log readers, or pub/sub subscribers -- basically, any process that waits around for a TCP socket to supply work.

## Why?

Currently, if you want to run, say, an http server *and* a background job processor in the same ruby program, you have to use a full blown asynchronous I/O framework such as Celluloid or Event-Machine.  This changes your architecture fairly significantly.  If you just want to operate more than one process in the same ruby program, but still want to use blocking I/O for each of those processes, ProcessHost can help.

## Usage

Here is a rough sketch of what a client process looks like:

```ruby
class SomeClient
  def start
    loop do
      # iterate, using `connection` for IO access
    end
  end

  # This will be invoked by ProcessHost in order to enable cooperative
  # multitasking.
  def change_connection_scheduler(scheduler)
    connection.scheduler = scheduler
  end

  def connection
    @connection ||= Connection.client "127.0.0.1", 2113
  end
end
```

A server:

```ruby
class SomeServer
  def start
    loop do
      server_connection.accept do |client_connection|
        # Fulfill the request using client_connection for IO access
      end
    end
  end

  # This will be invoked by ProcessHost in order to enable cooperative
  # multitasking.
  def change_connection_scheduler(scheduler)
    server_connection.scheduler = scheduler
  end

  def server_connection
    @server_connection ||= Connection.server 2113
  end
end
```

## License

The `process-host` library is released under the [MIT License](https://github.com/obsidian-btc/process-host/blob/master/MIT-License.txt).
