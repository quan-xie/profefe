# profefe - Collect profiling data for long-term analysis

[![Build Status](https://travis-ci.org/profefe/profefe.svg?branch=master)](https://travis-ci.org/profefe/profefe)
[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/profefe/profefe/master/LICENSE)

*profefe* continuously collects profiling data from a running Go application and provides an API for querying
the profiling samples base on metadata associated with the application.

---

**The project is still in the early prototyping stage. Things will change. Opinions and contributions are welcome.**

---

## Why Continuous Profiling?

"[Continuous Profiling and Go](https://medium.com/@tvii/continuous-profiling-and-go-6c0ab4d2504b)" describes
the motivation behind the project.

## How does it work?

See [Design Docs](DESIGN.md) documentation.

## Quickstart

**TODO add quickstart**

To build and start profefe collector, run:

```
> make
> ./BUILD/profefe -addr :10100 -log.level debug -badger.dir /tmp/profefe

2019-06-06T00:07:58.499+0200    info    profefe/main.go:86    server is running    {"addr": ":10100"}
```

The project includes a fork of [Google Stackdriver Profiler's example application][5], modified to use profefe agent,
that sends profiles to the local collector.

To start the example, in a separate terminal window run:

```
> go run ./examples/hotapp/main.go
```

After a brief period, the application will start sending CPU profiles to the collector:

```
send profile: http://localhost:10100/api/0/profiles?instance_id=87cdc549c84507f24944793b1ddbdc34&labels=version%3D1.0.0&service=hotapp-service&type=cpu
send profile: http://localhost:10100/api/0/profiles?instance_id=87cdc549c84507f24944793b1ddbdc34&labels=version%3D1.0.0&service=hotapp-service&type=cpu
send profile: http://localhost:10100/api/0/profiles?instance_id=87cdc549c84507f24944793b1ddbdc34&labels=version%3D1.0.0&service=hotapp-service&type=cpu
```

### Querying Profiles

Querying profiling data is an HTTP call to profefe collector API endpoint:

```
> go tool pprof 'http://localhost:10100/api/0/profiles/merge?service=hotapp-service&type=cpu&from=2019-05-30T11:49:00&to=2019-05-30T12:49:00&labels=version=1.0.0'

Fetching profile over HTTP from http://localhost:10100/api/0/profiles...
Saved profile in /Users/varankinv/pprof/pprof.samples.cpu.001.pb.gz
Type: cpu
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 43080ms, 99.15% of 43450ms total
Dropped 53 nodes (cum <= 217.25ms)
Showing top 10 nodes out of 12
      flat  flat%   sum%        cum   cum%
   42220ms 97.17% 97.17%    42220ms 97.17%  main.load
     860ms  1.98% 99.15%      860ms  1.98%  runtime.nanotime
         0     0% 99.15%    21050ms 48.45%  main.bar
         0     0% 99.15%    21170ms 48.72%  main.baz
         0     0% 99.15%    42250ms 97.24%  main.busyloop
         0     0% 99.15%    21010ms 48.35%  main.foo1
         0     0% 99.15%    21240ms 48.88%  main.foo2
         0     0% 99.15%    42250ms 97.24%  main.main
         0     0% 99.15%    42250ms 97.24%  runtime.main
         0     0% 99.15%     1020ms  2.35%  runtime.mstart
```

Note, above we requested all profiling data associated with the given meta data (service and time period),
as a single *merged* profile.

profefe includes an experimental tool, that allows importing existing pprof data into the collector.
While the collector running, run the tool as following:

```
> ./scripts/pprof_import.sh --service service1 --label region=europe-west3 --label host=backend1 --type cpu -- path/to/cpu.prof

uploading service1-cpu-backend1-20190313-0948Z.prof...OK
```

## HTTP API

### Save pprof data

```
POST /api/0/profiles?service=<service>&instance_id=<iid>&type=[cpu|heap]&labels=<key=value,key=value>
body pprof.pb.gz
```

- `service` — service name (string)
- `instance_id` — an identifier of running instance (string) (*TODO: why do we need instance_id?*)
- `type` — profile type (cpu, heap, block, mutex, or goroutine)
- `labels` — a set of key-value pairs, e.g. "region=europe-west3,dc=fra,ip=1.2.3.4,version=1.0" (Optional)

### Query saved meta information

```
GET /api/0/profiles?service=<service>&type=[cpu|heap]&from=<created_from>&to=<created_to>&labels=<key=value,key=value>
```

- `service` — service name
- `type` — profile type
- `from`, `to` — a time window between which pprof data was collected
- `labels` — a set of key-value pairs

### Query saved pprof data returning it as a single merged profile

```
GET /api/0/profiles/merge?service=<service>&type=[cpu|heap]&from=<created_from>&to=<created_to>&labels=<key=value,key=value>
```

Request parameters are the same as for querying meta information.

### Return individual pprof data

```
GET /api/0/profiles/<id>
```

- `id` - id of stored pprof file; returned with the request for meta information query

## Feedback

The feedback and contribution are very welcome.

## Further reading

While the topic of continuous profiling in the production is a bit unrepresented in the public internet, some
research or commercial projects are already exist

- [Stackdriver profiler][3]
- [Google-Wide Profiling: A Continuous Profiling Infrastructure for Data Centers](https://ai.google/research/pubs/pub36575) (paper)
- [StackImpact](https://stackimpact.com/docs/go-profiling/)
- [conprof](https://github.com/conprof/conprof)
- [Opsian - Continuous Profiling for JVM](https://opsian.com) (provides on-premises plan for enterprise customers)
- [Liveprof - Continuous Profiling for PHP](https://habr.com/ru/company/badoo/blog/436364/) (RUS)
- [FlameScope](https://github.com/Netflix/flamescope)

## License

MIT

[3]: https://stackimpact.com/
[5]: https://github.com/GoogleCloudPlatform/golang-samples/tree/master/profiler/hotapp
[pprof]: https://github.com/google/pprof/
