# Germany vs Belgium vs Netherlands vs Switzerland: whose trains are late?

The analysis behind the video [Is Deutsche Bahn Really That Bad? I Checked 242 Million Train Arrivals]([video link])

Every railway grades itself with its own rules. Switzerland counts a train as on time if it's
less than 3 minutes late, Germany and Belgium under 6, Romania under 20. Most only check the
last stop, and cancelled trains usually don't count at all. So the notebook measures four
countries with **one ruler**: every arrival of every train at every station in Germany,
Belgium, the Netherlands and Switzerland, from 1 November 2025 to 31 October 2026.
That's 242 million arrivals.

## The ruler

- **Every stop counts**, not just the final one (arrivals at stations inside the country)
- **On time = less than 6 minutes late**: Deutsche Bahn's own definition (up to 5:59)
- **Cancelled stops are counted**: separately, and once as a fail
- Trains only: rail-replacement buses, extra and charter trains are left out

## What it found

| | Germany | Belgium | Netherlands | Switzerland |
|---|---|---|---|---|
| All trains under 6 min late | 84% | 94% | 96% | 99% |
| Long-distance trains under 6 min late | **53%** | 92% | 94% | 97% |
| Long-distance trains 15+ min late | 27% | 3% | 2% | 1% |
| Trains with a cancelled stop | 6.8% | 3.8%\* | **8.6%** | 4.7% |
| Two-train trip, at least one 15+ min late | **47%** | 5.3% | 3.5% | 1.6% |

\* Infrabel's own count of trains fully or partly cancelled, November 2025 – February 2026
(Belgium's raw data has no fully cancelled trains, and Infrabel's series stops in February).

More findings in the notebook:

- **It's the long-distance trains.** German regional trains and S-Bahns get 85%, a few points
  behind the others. In Germany the gap between regional and long-distance is 32 points;
  elsewhere it's a few.
- **My numbers vs DB's own.** DB publishes its long-distance punctuality every month. Mine follow
  it month by month (correlation 0.995) and come out 3 to 5 points lower every month.
- **Belgium, three numbers for one year:** 93.6% on this ruler, 92.9% officially (final stop
  and Brussels, cancelled trains left out), 95.0% "neutralised" (delays blamed on outside causes
  taken out).
- **Delays cross borders.** German ICEs get 53% in Germany, 56% at Dutch stations and 63% in
  Belgium, next to 95% and 92% for the local intercity trains. In Switzerland they do better
  (87%), but **1 in 4 ICE stops there is cancelled** (Swiss intercity trains: 5.6%).
- **Who cuts late trains short?** The chance that a train is terminated at a station it was
  meant to carry on from climbs with its delay in every country. Belgium's climb is the
  steepest: about 120 times as likely at 30–59 minutes late as when on time. Germany cuts late
  trains the least (2.6% at 30–59 minutes late). This isn't proof anyone is gaming the figures:
  ending a late train early also keeps the return trip on time.

## Running it

```
pip install -r requirements.txt
```

Open `europe_trains.ipynb` and run all cells. It has three parts:

1. **Download** (about 26 GB: Switzerland's archive alone is 17 GB because it includes buses
   and trams). Files that are already there are skipped, so you can stop and restart.
2. **Prepare** the raw files into one table per country and month (`data/std/*.parquet`, one
   row per train arrival). About an hour, mostly the Swiss files.
3. **Analyse**: every number and chart above, with [DuckDB](https://duckdb.org) reading the 48
   prepared files directly.

Data quirks handled in the prepare step:

- **Germany**: `-9999` placeholder delays are dropped; station names lost their umlauts in the source.
- **Netherlands**: some delays carry a +1440 (24 h) midnight wrap-around.
- **Switzerland**: only measured (`REAL`) or estimated (`GESCHAETZT`) arrival times are used.
  A few daily files inside the monthly zips can be corrupt; they're skipped and listed
  (re-download that month).
- **Belgium**: only trains that ran are in the raw files. Skipped planned stops count as
  cancelled arrivals; an early termination is a last row whose plan said "stop and continue".

## Limits

Four countries, not all of Europe. Some German and Swiss times are the last forecast shown to
passengers rather than a measured time. Belgian cancellations come from Infrabel's own counts.
The two-train odds treat the two trains as independent.

## Data and credits

- Germany: Deutsche Bahn timetable API, collected by
  [piebro/deutsche-bahn-data](https://huggingface.co/datasets/piebro/deutsche-bahn-data), CC BY 4.0
- Netherlands: [Rijden de Treinen](https://www.rijdendetreinen.nl/en/open-data) train archive, CC BY 4.0
- Switzerland: [opentransportdata.swiss](https://archive.opentransportdata.swiss), actual data (Ist-Daten) archive
- Belgium: [Infrabel open data](https://opendata.infrabel.be): raw punctuality files,
  `nationale-stiptheid-per-maand` and `afgeschafte-treinen-per-maand-vanaf-2020`
- DB's monthly figures: [deutschebahn.com, Pünktlichkeitswerte](https://www.deutschebahn.com/de/konzern/konzernprofil/zahlen_fakten/puenktlichkeitswerte-6878476)
- Official thresholds: IRG-Rail, 14th Market Monitoring Report (2026), section 6.7
