@Deprecated: stub-runner is now part of [micro-infra-spring](https://github.com/4finance/micro-infra-spring)

Stub-runner
===========

Runs stubs for service collaborators. Treating stubs as contracts of services allows to use stub-runner as an implementation of [Consumer Driven Contracts](http://martinfowler.com/articles/consumerDrivenContracts.html).

### Running stubs


#### Running specified stubs
To run stubs execute `gradle run<ProjectMetadataFileWithoutExtension>Stubs`.

For example: to run stubs for `healthCheck` project execute `gradle runHealthCheckStubs`.
This will:
* start Zookeeper
* run stubs for projects defined in healthCheck.json from both global nad realm specific context
* register stubs in Zookeeper

#### Running all stubs
To run all stubs execute `gradle runStubs`.

Having the following *project* files:

`fooBar.json`:

```json
{
    "pl": [
        "com/ofg/foo/bar"
    ]
}
```

`healthCheck.json`:

```json
{
    "pl": [
        "com/ofg/ping"
    ]
}
```

and the following *mappings* folder structure:


```
/com/ofg/foo/bar/foobar.json
/pl/com/ofg/foo/bar/foobar.json
/com/ofg/ping/ping.json
/com/ofg/another/mapping.json

```

will result in:

* Loading one Zookeeper instance
* Setting up Wiremock instance with stubs from `/com/ofg/ping/ping.json` since it's described in `healthCheck.json`
* Setting up Wiremock instance with stubs loading first `/com/ofg/foo/bar/foobar.json` and then `/pl/com/ofg/foo/bar/pl_foobar.json` since it's described in `fooBar.json`
and they share common root path whereas there are some context specific mappings

#### Running from fat jar

We deploy the stub-runner project as a JAR in jCenter. Since version 0.2.0 we provide easier way of providing options via Args4J library.
You can set the following options to the main class:

```
java -jar stub-runner.jar [options...] 
 -a (--runAllStubs) VAL        : Switch that signifies that you want to run all
                                 stubs (e.g. 'true')
 -c (--context) VAL            : Context for which the project should be run
                                 (e.g. 'pl', 'lt')
 -maxp (--maxPort) N           : Maximum port value to be assigned to the
                                 Wiremock instance (e.g. 12345)
 -minp (--minPort) N           : Minimal port value to be assigned to the
                                 Wiremock instance (e.g. 12345)
 -p (--projectPath) VAL        : Relative path to the project which you want to
                                 run (e.g. '/com/ofg/foo/barProject.json')
 -r (--repository) VAL         : Path to repository containing the 'repository'
                                 folder with 'project' and 'mapping' subfolders
                                 (e.g. '/home/4finance/stubs/')
 -z (--zookeeperPort) N        : Port of the zookeeper instance (e.g. 2181)
 -zl (--zookeeperLocation) VAL : Location of local Zookeeper you want to
                                 connect to (e.g. localhost:23456)
```

##### Running on an environment with already existing Zookeeper

When you want to register your stubs against an already existing Zookeeper instance it's enough to provide a switch

```
-zl localhost:1234
```

that will point to the place where the Zookeper instance can be found


### Stub runner configuration

Under `ext` block inside `build.gradle` you can configure stub runner with the following properties:
* stubRepositoryPath
* stubRegistryPort
* minStubPortNumber
* maxStubPortNumber
* context
    
Project metadata that will be loaded is derived from the executed task name, e.g.: `gradle runHealthCheckStubs` will register collaborators defined in `healthCheck.json`.

### Defining collaborators' stubs

#### Defining service collaborators

Service collaborators are defined in project metadata file (JSON document) with the following structure:
```
{
    "context": [ Fully qualified names of your collaborators ]
}
```

Example:
```json
{
    "pl": [
        "com/ofg/foo",
        "com/ofg/bar"
    ]
}
```

By default project metadata definitions are stored in `projects` directory inside stub repository.

#### Global vs Realm specific stubs

Assuming the following metadata configuration:

```json
{
    "pl": [
        "com/ofg/foo",
        "com/ofg/bar"
    ]
}
```

You can define global stubbing behaviour (Wiremock will be fed with those stubs for all contexts) under the folder: 

```
com/ofg/foo
```

To override existing or add new behaviour that is specific to this realm just place your mappings under:

```
pl/com/ofg/foo
```

We ensure such ordering:

* Global mappings will be registered in Wiremock
* Afterwards realm specific stubs will get registered

By default stub definitions are stored in `mappings` directory inside stub repository.

#### Stubbing collaborators

For each collaborator defined in project metadata all collaborator mappings (stubs) available in repository are loaded and registered in service registry (Zookeeper).
Stubs are defined in JSON documents, whose syntax is defined in [WireMock documentation](http://wiremock.org/stubbing.html)

Example:
```json
{
    "request": {
        "method": "GET",
        "url": "/ping"
    },
    "response": {
        "status": 200,
        "body": "pong",
        "headers": {
            "Content-Type": "text/plain"
        }
    }
}
```

Stub definitions are stored in stub repository under the same path as collaborator fully qualified name.
Paths (as long it's inside the directory mentioned above) and names of documents containing stub definitions not play any other role than describing stubs' role / purpose.

#### Viewing registered mappings

Every stubbed collaborator exposes list of defined mappings under `__/admin/` endpoint.

## Other projects basing on *stub-runner*

- [Stub Runner Spring](https://github.com/4finance/stub-runner-spring) - adds automatic downloading of the newest versions of stubs to your tests
