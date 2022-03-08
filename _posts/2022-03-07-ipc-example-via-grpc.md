---
title: "How to build IPC communication using Protobuf and gRPC"
last_modified_at: 2022-03-07T12:29
categories:
  - Blog
tags:
  - Protobuf
  - IPC
  - gRPC
---

Some projects aren't enough using a single programming language. When in comes to the project that uses multiple languages, communicating between different languages become a problem.
There are (of course) many ways to implement IPC communication but Google's [protobuf](https://developers.google.com/protocol-buffers) seems to be a reasonable choice unleess I have a special purpose in mind. 

# Basic workflow (How does it work?)

Protobuf is a language-neutral serialization solution. Language-neutral means that you need to define messages first no matter which language you use. It looks like we are doing another work but it has a certain advantages. Be patient.

Basically, you define message `.proto` file.

```proto
message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;
}
```

And turn this `.proto` file into your language API using `protoc`

```java
Person john = Person.newBuilder()
    .setId(1234)
    .setName("John Doe")
    .setEmail("jdoe@example.com")
    .build();
output = new FileOutputStream(args[0]);
john.writeTo(output);
```

TA-DA. Serialized `Person` instance has been written to output stream!
It looks simple enough. Let's take a closer look how we can apply Protobuf into the project.

# ~~Install protobuf~~

~~Protobuf requires to be installed prior to use it. You can go to [https://developers.google.com/protocol-buffers/docs/downloads](https://developers.google.com/protocol-buffers/docs/downloads) to find an installation package. If you are planning to use C++, you will need to build from the source. Check [C++ installation page in protobuf GitHub page](https://github.com/protocolbuffers/protobuf/blob/v3.19.4/src/README.md).~~

It turns out I didn't need to install protobuf separately. gRPC installs protobuf since it's based on protobuf serialization.

# Install gRPC

Wait, what is gRPC? My original plan was to serialize messages via protobuf and send the serialized data into socket communication which will work as IPC communication. But why reinvent wheels? gRPC was designed to call a method on a server from the client directly which its purpose exactly matches with what IPC does.

You can follow instructions on how to install gRPC in [https://grpc.io/docs/languages/cpp/quickstart/](https://grpc.io/docs/languages/cpp/quickstart/)

# Write .proto for gRPC ready.

I have introduced simple `Person` message definition above. Let's expand it to IPC communication ready. Here is `.proto` message definition.

```proto
syntax = "proto3";

package protobuf_example;

service WhoIsIt {
    rpc SendMe (PersonRequest) returns (Person) {}
}

message Person {
  string name = 1;
  optional int32 id = 2;
  optional string email = 3;
}

message PersonRequest {
    string name = 1;
}
```

In this definition, I have defined `WhoIsIt` service which takes control of server and client side communication via RPC(Remote Procedure Call). `SendMe` method receieves `PersonRequest` message and responses with `Person` message. The structures of messages are defined as we have written in `.proto` file.

# Generate `.cc` and `.h` file (And `.py` file)

Once we have written the `.proto` file, we are ready to write language-dependent files. We assume that our message definition `.proto` file is `msg.proto`.

```bash
protoc --cpp_out ./cxx --grpc_out ./cxx --plugin=protoc-gen-grpc="/home/user/.local/bin/grpc_cpp_plugin" msg.proto
```

You will need to change `proto-gen-grpc` path along with your installation path. Once you run this command, you will have 4 files. `msg.grpc.pb.cc`, `msg.grpc.pb.h`, `msg.pb.cc`, `msg.pb.h`
As you already might have noticed that protoc generates `msg.pb.cc` and `msg.pb.h` which defines how protobuf generates message serialization. And the `protoc-gen-grpc` plugin generates `msg.grpc.pb.cc` and `msg.grpc.pb.h` which contains communication services.

What if I want to use `python` language for IPC between `C++` and `python`? You will need to generate python file as well.

```bash
protoc --python_out ./python --grpc_out ./python --plugin=protoc-gen-grpc="/home/user/.local/bin/grpc_python_plugin" msg.proto
```

And you will see `msg_pb2_grpc.py`, `msg_pb2.py` files has been generated.
Now we are ready to write our own IPC communication program!

# Write main.cpp for IPC server

I want to write an application that receieves name of the person request and send reply with email information in `C++` and send request message from `python` language. Let start with `C++` first.

```c++
#include <grpcpp/ext/proto_server_reflection_plugin.h>
#include <grpcpp/grpcpp.h>
#include <grpcpp/health_check_service_interface.h>

#include <iostream>
#include <memory>
#include <string>

#include "msg.grpc.pb.h"

using grpc::Server;
using grpc::ServerBuilder;
using grpc::ServerContext;
using grpc::Status;

using protobuf_example::Person;
using protobuf_example::PersonRequest;
using protobuf_example::WhoIsIt;

class PersonServiceImpl final : public WhoIsIt::Service {
  Status SendMe(ServerContext* context, const PersonRequest* request,
                Person* person) override {
    person->set_name(request->name());
    person->set_email("lim.jeikei@gmail.com");
    return Status::OK;
  }
};

int main(int argc, char** argv) {
  std::string server_address("0.0.0.0:50051");
  grpc::EnableDefaultHealthCheckService(true);
  grpc::reflection::InitProtoReflectionServerBuilderPlugin();

  PersonServiceImpl service;

  ServerBuilder builder;
  builder.AddListeningPort(server_address, grpc::InsecureServerCredentials());
  builder.RegisterService(&service);
  std::unique_ptr<Server> server(builder.BuildAndStart());
  std::cout << "Server listening on " << server_address << std::endl;

  server->Wait();

  return 0;
}
```

I have made a `PersonServiceImpl` class that inherits `WhoIsIt::Service` which was generated by `protoc`. Let take a look in `python` code now.

```python
import grpc

from protobuf.python.msg_pb2 import Person
from protobuf.python.msg_pb2_grpc import WhoIsItStub

if __name__ == "__main__":
    with grpc.insecure_channel('localhost:50051') as channel:
        stub = WhoIsItStub(channel)
        response = stub.SendMe(Person(name="Jongkuk Lim"))

    print(response)
```

You will find that `python` prints message like below. Once you run binary file built from C++ and run python script.

```shell
name: "Jongkuk Lim"
email: "lim.jeikei@gmail.com"
```

You can find demo repository in [https://github.com/JeiKeiLim/protobuf-ipc-example](https://github.com/JeiKeiLim/protobuf-ipc-example){:target='_blank'}

