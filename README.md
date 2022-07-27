# Covidestim API

[https://api2.covidestim.org](https://api2.covidestim.org) serves a simple REST
API that is freely available to the public, no authentication required. This page
documents several useful queries. Additionally, if you have OpenAPI tools, our
API serves an OpenAPI specification at the root URL.

We have two requests for use of this API:

> - [Use our data dictionary][data_dict] to interpret model outcomes.
> 
> - Don't abuse the API with unnecessary traffic, especially polling. Contact us
>   if you wish to use the API in a manner likely to generate large numbers of
>   requests.

This API is served by [PostgREST](https://postgrest.org). You can build more
complex queries than what we show here using the syntax explained in the
[PostgREST docs](https://postgrest.org).

The four endpoints we support are:

- [`/runs`](#runs) - all runs
- [`/latest_runs`](#latest_runs) - latest runs for each county/state
- [`/historical_runs`](#historical_runs) - historical runs for each county/state
- [`/enclosed_runs`](#enclosed_runs) - runs for counties in a particular state or states

<h2 name="runs">Endpoint `/runs`: GET any model run</h2>

Without parameters, a call to the `/runs` endpoint returns a list of every model
run in our database. This is not very useful, but we will use [filtering][pgrst]
and [embedding][pgrst_embed] to generate more valuable responses.

**Example:**

*Query:* <a href="https://api2.covidestim.org/runs?geo_name=eq.Connecticut&run_date=gt.2022-07-01&select=*,timeseries(*)" target="_blank"><code>/runs?geo_name=eq.Connecticut&run_date=gt.2022-07-01&select=*,timeseries(*)</code></a>

*Query meaning: All Connecticut runs after July 1st, including timeseries results.*

```json
[
  {
    "run_id": 14917,
    "run_date": "2022-07-21",
    "created_at": "2022-07-24T13:10:40.560945+00:00",
    "geo_name": "Connecticut",
    "input_id": 14917,
    "geo_type": "state",
    "method": "sampling",
    "timeseries": [
      {
        "run_id": 14917,
        "date": "2021-12-02",
        "data_available": true,
        "deaths": 18.8298564777493,
        "deaths_of_diagnosed": 17.2268102439229,
        "diagnoses": 2769.54656378994,
        "diagnoses_of_symptomatic": 1661.41185729121,
        "effective_protection_inf_prvl": 1843934.61054172,
        "effective_protection_inf_vax_boost_prvl": null,
        ...
      }
    ]
  },
  ...
]
```

This API response embeds model results under the `.timeseries` key. Important
notes:

- Each element of `.timeseries` contains estimates for the week ending on date `date`.

- `date` is always `YYYY-MM-DD`.

- **Model runs of counties** *(`geo_type == 'county'`)* **do not have uncertainty
  intervals**. All `*_p2_5`, `*_p25`, `*_p75`, `*_p97_5` keys are set to `null`.

- `geo_name` can either be:
  - A state name
  - For counties only, a FIPS code. For example, `geo_name=eq.09009` selects
    model runs for New Haven County, Connecticut.

<iframe src="//api.apiembed.com/?source=https://covidestim.s3.amazonaws.com/api-har-files/1.json&targets=http:1.1,shell:curl,python:requests,javascript:fetch" frameborder="0" scrolling="no" width="80%" height="180px" seamless></iframe>

<h2 name="latest_runs">Endpoint `/latest_runs`: newest run for each geography</h2>

**Example: Latest results for all states**

*Query:* <a href="https://api2.covidestim.org/latest_runs?geo_type=eq.state&select=*,timeseries(*)" target="_blank"><code>/latest_runs?geo_type=eq.state&select=*,timeseries(*)</code></a>

This returns an array of model runs, as with the previous query, except that
each array represents the latest model run for each U.S. state.

<iframe src="//api.apiembed.com/?source=https://covidestim.s3.amazonaws.com/api-har-files/2.json&targets=http:1.1,shell:curl,python:requests,javascript:fetch" frameborder="0" scrolling="no" width="80%" height="180px" seamless></iframe>

**Example: Latest results for all counties**

*Query:* <a href="https://api2.covidestim.org/latest_runs?geo_type=eq.county&select=*,timeseries(*)" target="_blank"><code>/latest_runs?geo_type=eq.county&select=*,timeseries(*)</code></a>

<h2 name="latest_enclosed_runs">Endpoint `/latest_enclosed_runs`: Latest runs for child geographies</h2>

**Example: Latest `r_t`, `infections` estimates for all Connecticut counties**

*Query:* <a href="https://api2.covidestim.org/latest_enclosed_runs?parent_geo=eq.Connecticut&select=*,timeseries(date,r_t,infections)" target="_blank"><code>/latest_enclosed_runs?parent_geo=eq.Connecticut&select=*,timeseries(date,r_t,infections)</code></a>

*Query meaning: Latest runs for all geographies whose parent geography is
Connecticut, embedding the `date`, `r_t`, and `infections` columns from the
timeseries results for each model run.*

<iframe src="//api.apiembed.com/?source=https://covidestim.s3.amazonaws.com/api-har-files/4.json&targets=http:1.1,shell:curl,python:requests,javascript:fetch" frameborder="0" scrolling="no" width="80%" height="180px" seamless></iframe>

<h2 name="historical_runs">Endpoint `/historical_runs`: Older runs for a particular geography</h2>

**Example: Historical runs for Connecticut**

<a href="https://api2.covidestim.org/historical_runs?geo_name=eq.Connecticut&select=*,timeseries(*)" target="_blank"><code>/historical_runs?geo_name=eq.Connecticut&select=*,timeseries(*)</code></a>

*Query meaning: The first run from every month, plus the latest four runs, for
Connecticut, including timeseries results for each run.*

This endpoint returns a selection of historical model runs for each geography.
Namely, the following model runs are selected from the collection of runs from
each geography, if they exist:

- The 3 latest model runs, excluding the most recent run
- First model run of each month

<iframe src="//api.apiembed.com/?source=https://covidestim.s3.amazonaws.com/api-har-files/5.json&targets=http:1.1,shell:curl,python:requests,javascript:fetch" frameborder="0" scrolling="no" width="80%" height="180px" seamless></iframe>

[pgrst]: https://postgrest.org/en/stable/api.html#horizontal-filtering-rows
[pgrst_embed]: https://postgrest.org/en/stable/api.html#resource-embedding
[data_dict]: http://pkg.covidestim.org/reference/summary.covidestim_result.html
