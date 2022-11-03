# Data Engineering API Example

Create a Data Engineering API around Flask and Pandas:

Data teams often need to build libraries and services to make it easier to work with data on the platform.  In this example there is a need to create a Proof of Concept aggregation of csv data.  A REST API that accepts a csv, a column to group on, and a column to aggregate and returns the result.

Note,this project is a Chapter in the book Pragmatic AI, the entire projects source can be found [here](https://github.com/noahgift/pragmaticai)

## Using the default web app.
The Swagger API has some pretty powerful tools built in.

* To list the plugins that are loaded:
![Plugins](images\testSwagger.png)

* To apply one of those functions:
![Swagger API](images\testSwagger2.png)

## Sample Input
```
first_name,last_name,count
chuck,norris,10
kristen,norris,17
john,lee,3
sam,mcgregor,15
john,mcgregor,19
```
## Sample Output
```
norris,27
lee,3
mcgregor,34
```

## How to run example and setup environment:
To create environment (tested on OS X 10.12.5), run `make setup`
Then source the virtualenv. I add an alias to my .zshrc:
```
alias ntop="cd ~/src/pai-aws && source ~/.pai-aws/bin/activate"
```
I can then type in: `ntop` 
Next, I then make sure I have the latest packages and that linting and tests pass by running make all: `make all`

I also like to verify that pylint and pytest and python are exactly the versions I expect: `make env`

## How to interact with Commandline tool (Click Framework):
Aggregate CSV
```
(.pai-aws) ➜  pai-aws git:(master) ✗ ./csvcli.py cvsagg --file ext/input.csv --column last_name
```
Testing a bigger file than the assignment:
```
(.pai-aws) ➜  pai-aws git:(master) ✗ ./csvcli.py cvsagg --file ext/large_input.csv --column last_name 
```

## How to run webapp (primary question) and use API
To run the flask api, run the make command:
```
(.pai-aws) ➜  pai-aws git:(master) ✗ make start-api
```

## Test Client with Swagger UI
Next, open a web browser to view Swagger API documentation: http://0.0.0.0:5001/apidocs/#/

For example to see swagger docs/UI for cvs aggregate endpoint go here: http://0.0.0.0:5001/apidocs/#!/default/put_api_aggregate

## Interactively Test application in IPython
Using the requests library you can query the api as follows in IPython:
```
In [1]: import requests, base64
In [2]: url = "http://0.0.0.0:5001/api/npsum"
In [3]: payload = {'column':'count', 'group_by':"last_name"}
In [3]: headers = {'Content-Type': 'application/json'}
In [3]: with open("ext/input.csv", "rb") as f:
    ...:     data = base64.b64encode(f.read())
In [4]: r = requests.put(url, data=data, params=payload, headers=headers)
In [5]: r.content
Out[5]: b'{"count":{"mcgregor":34,"lee":3,"norris":27}}'
```

## How to simulate Client:
```
(.pai-aws) ➜  tests git:(inperson-interview) ✗ python client_simulation.py
```

## How to interact with python library (nlib):
```
In [1]: from nlib import csvops
In [2]: df = csvops.ingest_csv("ext/input.csv")
In [3]: df.head()
```

## Benchmark web Service
Use the Makefile to start the web service and then benchmark it (which uploads base64 encoded csv): `make start-api`

Then run the apache benchmark via Makefile.  The output should look something like this:
```bash
(.pai-aws) ➜  pai-aws git:(inperson-interview) ✗ make benchmark-web
#very simple benchmark of api
ab -n 1000 -c 100 -T 'application/json' -u ext/input_base64.txt http://0.0.0.0:5001/api/aggregate\?column=count\&group_by=last_name
This is ApacheBench, Version 2.3 <$Revision: 1757674 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 0.0.0.0 (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests

Server Software:        Werkzeug/0.12.2
Server Hostname:        0.0.0.0
Server Port:            5001

Document Path:          /api/aggregate?column=count&group_by=last_name
Document Length:        154 bytes

Concurrency Level:      100
Time taken for tests:   7.657 seconds
Complete requests:      1000
Failed requests:        0
Total transferred:      309000 bytes
Total body sent:        308000
HTML transferred:       154000 bytes
Requests per second:    130.60 [#/sec] (mean)
Time per request:       765.716 [ms] (mean)
Time per request:       7.657 [ms] (mean, across all concurrent requests)
Transfer rate:          39.41 [Kbytes/sec] received
                        39.28 kb/s sent
                        78.69 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   1.1      0       6
Processing:    18  730 142.4    757     865
Waiting:       18  730 142.4    756     865
Total:         23  731 141.3    757     865

Percentage of the requests served within a certain time (ms)
  50%    757
  66%    777
  75%    787
  80%    794
  90%    830
  95%    850
  98%    860
  99%    862
 100%    865 (longest request)
```

## Circle CI Configuration
```
machine:
  python:
    version: 3.6.1

dependencies:
  pre:
    - make install

test:
  pre:
    - make lint-circleci
    - make test-circleci
```
Those make commands being called are below.  They write artifacts to the Circle CI Artifacts Directory:
```
lint-circleci:                                                              
  pylint --output-format=parseable --load-plugins pylint_flask --disable=R,C flask_app/*.py nlib csvcli > $$CIRCLE_ARTIFACTS/pylint.html  

test-circleci:
  @cd tests; pytest -vv --cov-report html:$$CIRCLE_ARTIFACTS --cov=web --cov=nlib test_*.py  
```














