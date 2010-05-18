Clojure-protobuf provides a clojure interface to Google's [protocol buffers](http://code.google.com/p/protobuf).
Protocol buffers can be used to communicate with other languages over the network, and
they are WAY faster to serialize and deserialize than standard Clojure objects.

## Usage

Write a `.proto` file:

    message Person {
      required int32  id    = 1;
      required string name  = 2;
      optional string email = 3;
      repeated string likes = 4;
    }

If you put it in the protos directory of your project, you can compile it with lein:

    lein proto example.proto

Now you can use the protocol buffer in clojure:

    (use 'protobuf)
    (defprotobuf Person Example$Person)

    (def p (protobuf Person :id 4 :name "Bob" :email "bob@example.com"))
    => {:id 4, :name "Bob", :email "bob@example.com"}

    (assoc p :name "Bill"))
    => {:id 4, :name "Bill", :email "bob@example.com"}

    (assoc p :likes ["climbing" "running" "jumping"])
    => {:id 4, name "Bob", :email "bob@example.com", :likes ["climbing" "running" "jumping"]}

    (def b (protobuf-dump p))
    => #<byte[] [B@7cbe41ec>

    (protobuf-load Person b)
    => {:id 4, :name "Bob", :email "bob@example.com"}

A protocol buffer map is immutable just like other clojure objects. It is similar to a
struct-map, except you cannot insert fields that aren't specified in the `.proto` file.

## Collection extensions

Clojure-protobuf supports extensions to protocol buffers which provide sets and maps using
repeated fields. To use these, you must import the extension file and include it when compiling. For example:

    import "collections.proto";
    message Photo {
      required int32  id     = 1;
      required string path   = 2;
      repeated string labels = 3 [(set)    = true];
      repeated Attr   attrs  = 4 [(map)    = true];
      repeated Tag    tags   = 5 [(map_by) = "person_id"];

      message Attr {
        required string key = 1;
        optional string val = 2;
      }

      message Tag {
        required int32 person_id = 1;
        optional int32 x_coord   = 2;
        optional int32 y_coord   = 3;
        optional int32 width     = 4;
        optional int32 height    = 5;
      }
    }

Compile the file:

     lein proto example.proto

Then you can access the maps in clojure:

    (use 'protobuf)
    (defprotobuf Photo Example$Photo)

    (def p (protobuf Photo :id 7 :path "/photos/h2k3j4h9h23" :labels #{"hawaii" "family" "surfing"}
                           :attrs {"dimensions" "1632x1224", "alpha" "no", "color space" "RGB"}
                           :tags  {4 {:person_id 4, :x_coord 607, :y_coord 813, :width 25, :height 27}}))
    => {:id 7 :path "/photos/h2k3j4h9h23" :labels #{"hawaii" "family" "surfing"}...}

    (def b (protobuf-dump p))
    => #<byte[] [B@7cbe41ec>

    (protobuf-load Photo b)
    => {:id 7 :path "/photos/h2k3j4h9h23" :labels #{"hawaii" "family" "surfing"}...}

## Installation

In your [Leiningen](http://github.com/technomancy/leiningen) project.clj:

    :dependencies [[clojure-protobuf "0.1.0-SNAPSHOT"]]

Build from source:

    lein deps
    lein jar

This code works with Clojure 1.2. Here's the [1.1 branch](http://github.com/ninjudd/clojure-protobuf/tree/clojure-1.1).