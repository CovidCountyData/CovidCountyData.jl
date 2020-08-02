# CovidCountyData.jl

Welcome to the Julia client library documentation for the [Covid County Data](https://covidcountydata.org) (CCD) database.


## Installation

Install the CovidCountyData package from Julia pkg mode as follows:

```julia
(@v1.5) pkg> add https://github.com/CovidCountyData/CovidCountyData.jl
```

## API keys

Our data is free and open for anyone to use (and always will be). Our team agreed that this was
central to our mission when we agreed to begin this project. However, we do find it useful to
have information about our users and to see how they use the data for two reasons:

1. It helps us focus and improve the datasets that are seeing the most use.
2. The number of users, as measured by active API keys, is one metric that we use to show that the
   project is useful when we are discussing additional grant funding.

We are grateful to everyone who is willing to register for and use their API key when interacting
with our data.

To register for an API key, you can register [on our website](https://covidcountydata.org#register)
or from the Julia package using the `register` method.

```julia
using CovidCountyData

c = Client()
register(c)
```

You will be prompted for your email address. After entering a valid email address we will issue
an API key, store it on your machine, and automatically apply it to all future requests made from
this package to our servers.

If at any time you would like to remove your API key, please delete the file `~/.covidcountydata/apikey`.


## Data


### Datasets

You can see a list of the available datasets in our API from the Julia library by doing:

```julia
using CovidCountyData

println(datasets())
```

For more information on each of these datasets, we recommend that you visit our
[data documentation page](https://covidcountydata.org/data-api#rest).


### Data keys

Many of the datasets in our database are indexed by one or more common "keys". These keys are:

- `vintage`: The date and time that the data was downloaded into our database. We collect this
  because of the rapidly evolving nature of COVID-19 -- It allows us to have a record of when data was
  changed/corrected/updated.
- `dt`: The date and time that an observation corresponds to. For series like COVID tests
  administered this may a daily frequency, but, for others like unemployment it may be a weekly or
  monthly frequency.
- `location`: A geographic identifier for the location. For the counties/states in the dataset,
  this variable corresponds to the Federal Information Processing Standards number.

Whenever two series with common keys are loaded together, they will be merged on their common keys.


### Requesting data

Requesting data using the Julia client library involves three steps:


#### 1. Create a client

To create a client, use the `Client` class.

```julia
using CovidCountyData

c = Client()
```

You can optionally pass in an API key if you have one (see the section on API keys).

```julia
c = Client("my api key")
```

If you have previously registered for an API key on your current machine, it will be loaded and
used automatically for you.

In practice you should rarely need to pass the API key by hand unless you are loading the key from
an environment variable or another source.


#### 2. Build a request

Each of the datasets in the API have an associated method.

To add datasets to the current request, call `request!(::Client, dataset())`. For example, to add
the `covid_us` dataset to the request, you would call:

```julia
request!(c, covid_us(location=6))
```

If you wanted to add another dataset, such as `demographics`, you would call `request!` again:
well.

```julia
request!(c, demographics())
```

You can see that the printed form of the client is updated to show you what the current request
looks like by printing the current client.

```julia
c
```

To clear the current request, use `reset!(c)`:

Multiple datasets can be addded in one call to `request!`, which can clean up code in some situations. For example, the calls to `covid_us` and `demographics` could have been written 

```julia
request!(c, demographics(), covid_us(location=6))
```

The order of the requests does not matter and will not change the dataset you ultimately end up with.

**Filtering data**

Each of the dataset functions has a number of filters that can be applied.

These filters allow you to select certain rows and/or columns.

For example, in the above example we had `covid_us(location=6)`. This instructs the client to
only fetch data for geographic regions that are in the state of California.

**NOTE:** If a filter is passed to one dataset in the request but is applicable to other datasets
in the request, it will be applied to *all* datasets.

For example in `request!(c, demographics(), covid_us(location=6))` we only specify a `location` filter on the
`covid_us` dataset, but when the data is collected it will also be applied to `demographics`.

We do this because we end up doing an inner join on all requested datasets, so when we filter the
state in `covid_us` they also get filtered in `demographics`.


#### 3. Fetch the data

To fetch the data, call the `fetch` method from the client.

```julia
df = fetch(c)
```

Note that after a successfully request, the client is reset so there are no "built-up" requests
remaining.



## Examples

We provide a few simple examples here in the README, but you can find additional examples in the `examples.jl` file.

**Simple Example: Single dataset for all FIPS**

The example below loads all within county mobility data.

```julia
using CovidCountyData
c = Client()

request!(c, mobility_devices())
df = fetch(c)
```


**Simple Example: Single dataset for single county**

The example below loads just demographic information for Travis County in Texas.

Notice that we can select a particular geography by specifying the fips code using the `location` keyword.

```julia
c = Client()
request!(c, demographics(location=48453))
df = fetch(c)
```


**Simple Example: Single dataset for all states**

The example below loads demographic information for all counties in Texas.

```julia
c = Client()
request!(c, demographics(location="<100"))
df = fetch(c)
```

**Intermediate Example: Multiple datasets for single county**

The example below loads covid and demographic data and showcases how to request multiple datasets together. It will automatically merge and return these datasets.

Note that applying a filter to any of the datasets (in this case `location=6037`) will apply it to all datasets.

```julia
c = Client()
request!(
    c,
    covid_us(location=6037),
    demographics(),
)
df = fetch(c)
```


**Advanced Example: Multiple datasets with multiple filters and variable selection**

The example below loads data from three datasets for a particular FIPS code, using a particular date of demographics, and selects certain variables from the datasets.

```julia
c = Client()
request!(
    c,
    economic_snapshots(variable="GDP_All industry total"),
    covid_us(location=6037),
    demographics(variable="Total population"),
)
df = fetch(c)
```

There are more examples in the [`examples.jl`](https://github.com/CovidCountyData/CovidCountyData.jl/blob/master/examples.jl) file. We encourage you to explore them and to reach out if you have questions!
