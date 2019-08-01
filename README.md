# loadtest - load testing from R <img src="man/figures/README-ai-logo.png" align="right" height="40px" />

[![License](https://img.shields.io/badge/License-Apache%202.0-yellowgreen.svg)](LICENSE)

<img src="man/figures/README-loadtest-hex.png" align="right"  height="120px"/>

This package is to make load testing of APIs, such as those created with the R package plumber, easy to do. It uses Apache JMeter on the backend--a modern platform for load testing. The loadtest package is open source and maintained by the AI @ T-Mobile team.

Load testing, the process of ensuring that a website or API works by simulating a massive load a traffic and seeing how the website or API performs, is a critical step in the development of production systems. R is becoming more popular for production system environments through tools such as [plumber](https://www.rplumber.io/), [Rocker](https://hub.docker.com/u/rocker/), and the [T-Mobile R TensorFlow API platform](https://github.com/tmobile/r-tensorflow-api), so having a way to do load testing while staying within the R environment is useful.

This package has a single primary function `loadtest` used to run a load test against a URL. For example, suppose we wanted to test having 5 threads hit google.com, and each thread hits Google 10 times sequentially. To run that test we would do a single `loadtest` call, which creates a data frame of results:

```r
library(loadtest)

results <- loadtest(url = "https://www.google.com", method = "GET", threads = 5, loops = 10)

head(results)
```

<div style="width: 100%; overflow: auto;">

| request_id|start time                | thread| num threads|response code |response message |request status | sent bytes| received bytes| time since start| elapsed| latency| connect| idle|
|----------:|:-------------------|------:|-----------:|:-------------|:----------------|:--------------|----------:|--------------:|----------------:|-------:|-------:|-------:|----:|
|          1|12:22:23 |      1|           5|200           |OK               |Success        |        115|          12263|                0|     696|     668|     604|    0|
|          2|12:22:23 |      5|           5|200           |OK               |Success        |        115|          13190|                0|     701|     668|     604|    0|
|          3|12:22:23 |      2|           5|200           |OK               |Success        |        115|          12219|                0|     701|     668|     604|    0|
|          4|12:22:23 |      4|           5|200           |OK               |Success        |        115|          12268|                0|     705|     668|     604|    0|
|          5|12:22:23 |      3|           5|200           |OK               |Success        |        115|          12246|                0|     707|     673|     604|    0|
|          6|12:22:23 |      1|           5|200           |OK               |Success        |        115|          12298|              700|     152|     128|      78|    0|

</div>

This table has 50 rows, one for each of the ten requests that the five threads made. There are lots of useful columns, like:

* __request_status__ - if the request was successful
* __start_time__ - the time the request started
* __elapsed__ - how many milliseconds passed between the start of the request and the end of the response

If you're creating your own API, by using the loadtest package you can test how quickly the responses will be returned as you increase the concurrent requests. This can help you avoid situations where the API is released and is unable to handle the production load.

In addition to creating a table of test results, the package has plotting capabilities to help you quickly understand the values of the test. using the `loadtest::plot_` commands you can plot the data in multiple ways. Here we show a new request with 8 threads and 32 requests each.

```r
plot_elapsed_times(results)
plot_elapsed_times_histogram(results)
plot_requests_by_thread(results)
plot_requests_per_second(results)
```

![Example plots](man/figures/README-example-plots.png)


Starting from the upper-left:

1. The elapsed time of each request of the course of the test. Here we can see that the completion time was rather consistent, but had some variance for the first request of the threads.
2. A histogram of the elapsed times. We can see the vast majority of the requests too around 1750ms to complete.
3. The start time of the requests per each thread. It shows consistent behavior across the threads.
4. The distribution of the number of requests per second. Here we see the API achieved a rate of resolving around 4-5 requests per second. If we expect the production load to be higher than that then there would be a problem.

Finally, you can easily create an Rmarkdown report which includes these plots as well as other summary statistics. This can be useful for automating automatically running load tests as part of a build process. The function takes the results of a load test and a path to save to and generates the report.

```{r}
loadtest_report(results,"C:/[location_to_save_to].html")
```

<img src="man/figures/README-example-report.png" height="400px" style="margin-top:20px">


## Installation

Since loadtest is powered by Apache JMeter, which requires Java, the installation steps are:

1. Install Java*
2. Install JMeter
3. Install loadtest R package

_*While Java is required for loadtest, the package rJava is not needed._

### Installing Java

To install Java, go to the [Oracle website](https://java.com/en/download/help/download_options.xml) and follow their instructions. 

_Be sure that the Java path is directly available to the system follow the directions [here](https://www.java.com/en/download/help/path.xml)_

You can test you have Java correctly installed by opening a terminal and running `java -version`.

### Installing JMeter
Then, you need to download [Apache JMeter](https://jmeter.apache.org/download_jmeter.cgi), and extract the zip file to a folder. That folder needs to be added to the system path, just like for Java.

You can test you have Apache JMeter correctly installed by opening a terminal and running `jmeter --version`. If you get an error that jmeter doesn't exist, the path was likely not set correctly.

### Installing loadtest

You can install the loadtest package by running:

```r
remotes::install_github("tmobile/loadtest")
```