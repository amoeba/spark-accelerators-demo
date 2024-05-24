# spark-accelerators-demo

> [!NOTE]
> This repository is intended for demonstration purposes only. Its content is [APL-2.0](./LICENSE).

This repository contains a basic demonstration of using two open-source native accelerators for [Apache Spark](https://spark.apache.org/):

- [Apache Gluten](https://gluten.apache.org/) w/ [Velox](https://github.com/facebookincubator/velox) as a backend.
- [Apache DataFusion Comet](https://datafusion.apache.org/comet/)

The intended audience here is anyone looking to start experimenting with these accelerators who may already be familiar with Spark itself but not with the accelerators.

## Introduction

What are these accelerators, what are we accelerating, and how does this fit into Spark?

TODO

## Setup

Beyond performance (query execution time, memory usage during query execution), a key concern anyone experimenting with Spark accelerators must be aware of is _compatibility_.
Specifically,

1. Which version(s) of Spark are supported and tested?
2. Which Spark expressions and operators are supported?

Generally speaking, you need to be aware of which Spark version or versions your accelerator has been tested with.
For example, a single Gluten release is only compatible with a single Spark release.

For the purposes of this demo, we're using a version of Spark that is compatible with the version of Gluten we'll be building and also Comet: Spark 3.4.2.

To run through this demo, you'll need:

- A Debian or RHEL-based Linux system with superuser privileges
- git
- Java 8
- Docker w/ Docker Compose for running our demo Spark cluster
- A Spark 3.4.2 binary release

## Gluten w/ Velox

Gluten supports either [Velox](https://github.com/facebookincubator/velox) or [Clickhouse](https://clickhouse.com/) as an execution engine.
We'll be using the Velox backend here as it's fairly well-documented.

### Building

The Gluten project tries to provide a small set of binaries in their [Releases on GitHub](https://github.com/apache/incubator-gluten/releases) but I didn't have any luck running them.
Therefore I recommend building Gluten yourself which is challenging but doable.

Please note:

1. The [official Gluten build documentation](https://gluten.apache.org/docs/velox/getting-started) has you run commands which use `sudo`, are not self-contained, and modify your fileystem so I recommended running these commands on a test system, inside a container, or in a virutal machine.
2. I wasn't able to run their all-in-one build script, `./dev/buildbundle-veloxbe.sh`, so my instructions here differ from Gluten's a bit.


Run the following commands:

```sh
git clone https://github.com/apache/incubator-gluten
cd ep/build-velox/src
./get-velox.sh # must run this first, build-velox doesn't sync for me
./build-velox.sh
cd ../../../cpp
./compile.sh --build_velox_backend=ON
cd ..
mvn clean package -Pbackends-velox -Prss -Pspark-3.4 -DskipTests
```

You should now have a binary JAR in `./package/target` named something like `gluten-velox-bundle-spark3.4_2.12-debian_12_x86_64-1.2.0-SNAPSHOT.jar`.

### Using Gluten

Now we can launch a standalone Spark client/server to test that our built JAR works:

```sh
export GLUTEN_JAR="/path/to/your/built/gluten.jar"
./bin/spark-shell \
  --conf spark.plugins=org.apache.gluten.GlutenPlugin \
  --conf spark.memory.offHeap.enabled=true \
  --conf spark.memory.offHeap.size=20g
  --conf spark.shuffle.manager=org.apache.spark.shuffle.sort.ColumnarShuffleManager \
  --jars $GLUTEN_JAR
```

Once we're in the Spark Shell, we can run a basic query and ask for an EXPLAIN to check that our query will be executed with Velox:

```scala
scala> spark.sql("select 1;").explain
== Physical Plan ==
VeloxColumnarToRowExec
+- ^(3) ProjectExecTransformer [1 AS 1#58]
   +- ^(3) InputIteratorTransformer[fake_column#60]
      +- ^(3) InputAdapter
         +- ^(3) RowToVeloxColumnar
            +- *(1) Scan OneRowRelation[fake_column#60]
```

## Apache DataFusion Comet

## More

TODO

### Data

TODO: Figure out a good data source to test against.

### Queries

TODO: Figure out a reasonable query to run.
