# US car shipping prices, transit times and broker data (Transportli)

> Also on Hugging Face: <https://huggingface.co/datasets/Transportli/us-car-shipping-prices>. Canonical source and explanations: <https://transportli.com/auto-transport/price-index/>. Releases of this repository are archived on Zenodo with a DOI.

Six small tables of **real, measured** car shipping data from [Transportli](https://transportli.com/), a US marketplace that matches vehicle shipments with vetted carriers. Nothing here is a forecast (the one modelled table, `state_effects`, is fitted to measured prices): prices are quotes returned by our platform's pricing tool, transit times come from real delivered orders, and the broker table is public FMCSA data. Companion datasets: LTL (pallet) freight in [Transportli/us-ltl-freight-prices](https://huggingface.co/datasets/Transportli/us-ltl-freight-prices) and container drayage in [Transportli/us-container-drayage-rates](https://huggingface.co/datasets/Transportli/us-container-drayage-rates). Every row carries a `source_url` that points to the Transportli page that explains the numbers.

- Snapshot: **November 2026** (the price index keeps growing weekly; the other tables are refreshed monthly)
- License: **CC BY 4.0**. Please credit **Transportli** and link to the `source_url` of the table you use (see Citation).
- Site: <https://transportli.com/> · Price index: <https://transportli.com/auto-transport/price-index/> · Data license: <https://transportli.com/data-license/>

## Tables

| Config | Rows | What a row is | Source page |
|---|---|---|---|
| `price_index` | 80 | One lane in one week: open and enclosed sedan price, miles, price per mile, enclosed premium and the tool's confidence label | [Price index](https://transportli.com/auto-transport/price-index/) |
| `distance_bands` | 6 | One distance band: median sedan, SUV, pickup and van price from 1,617 real lanes, sedan 10th and 90th percentile, price per mile | [Cost page](https://transportli.com/auto-transport/car-shipping-cost/) |
| `direction_by_band` | 7 | One distance band: how far apart the sedan price is between the two directions of a corridor (790 corridors quoted both ways) | [Direction study](https://transportli.com/blog/is-it-cheaper-to-ship-a-car-in-one-direction/) |
| `state_effects` | 49 | One state: price effect of leaving and of arriving there, as % against a typical lane of the same length | [Statistics](https://transportli.com/auto-transport/car-shipping-statistics/) |
| `transit_times` | 10 | One distance band (or vehicle type): days from pickup to delivery on 377 delivered orders | [Transit times](https://transportli.com/auto-transport/how-long-does-it-take-to-ship-a-car/) |
| `broker_states` | 51 | One state: active FMCSA-registered brokers, per 100,000 residents, and how many had a revocation in five years | [Broker report](https://transportli.com/blog/freight-brokers-by-state/) |

```python
# From Hugging Face (or read the CSVs in data/ directly)
from datasets import load_dataset
ds = load_dataset("Transportli/us-car-shipping-prices", "distance_bands", split="train")
print(ds.to_pandas())
```

## How the numbers are made

**Prices.** Every price is the result of entering a pickup city and a delivery city into our platform's pricing tool for one **running vehicle on an open trailer** (enclosed where the column says so), door to door. Nothing is adjusted or smoothed. The tool returns a confidence label of roughly 60% to 90% with each quote.

- `price_index`: a fixed basket of 20 lanes (10 corridors in both directions), quoted every Monday for a running sedan, open and enclosed. Method: <https://transportli.com/auto-transport/how-we-price/>.
- `distance_bands`: medians over the 1,617 lanes we have quoted for our city and state guides, grouped by the distance the tool reports. Lanes of 3,100 miles or more are left out, as on the cost page, and a blank `miles_max` means the band has no upper limit. `*_vs_sedan_pct` compares the median of each vehicle type with the sedan median.
- `direction_by_band`: for each corridor quoted both ways, the gap is the higher sedan price divided by the lower, minus 1. Bands use the average of the two distances.
- `state_effects`: an additive model, ln(price) = distance curve + origin-state effect + destination-state effect, fitted by ridge regression on sedan prices. `leave_effect_pct` is the effect of the car leaving the state, `arrive_effect_pct` of arriving, both against a typical lane of the same length. `*_sd_pts` is a bootstrap standard deviation in percentage points. Use `min_12_lanes_each_way` to keep only states with enough lanes.
- `transit_times`: calendar days between the pickup and delivery dates on signed bills of lading, 377 delivered cars and light trucks from June 2024 to November 2026. Names, addresses, VINs and prices of single orders are never published. `vehicle_type_500_to_1499_miles` rows only cover trips of 500 to 1,499 miles.
- `broker_states`: the FMCSA public registry of active property brokers (snapshot 2026-11-04), by business state, with 2020 Census population. `brokers_with_revocation_5y` counts brokers with at least one involuntary revocation notice or revocation between 2020-11-01 and 2025-10-31. A revocation is a paperwork event, usually a lapsed bond or filing, not proof of fraud. The CI columns are 95% intervals.

## Limits

- These are **quotes from one platform, not booked prices or market averages.** A carrier sets the final price when it accepts a load, and the same lane can move a few dollars between runs.
- The lanes are the ones we publish city and state guides for, so large metros and busy corridors are over-represented. They are not a random sample of US trips.
- `state_effects` are model estimates. States with few lanes have wide uncertainty.
- Transit times come from our own orders, which lean toward dealer, auction and collector cars.
- The price index began on October 19, 2026 and says nothing about earlier seasons yet. Seasonal swings (for example the snowbird season) have not been measured across a full year.

## Citation

> Transportli, *Car Shipping Prices, Transit Times and Broker Data*, November 2026. <https://transportli.com/auto-transport/price-index/>

```bibtex
@misc{transportli_car_shipping_data_2026_11,
  title  = {US Car Shipping Prices, Transit Times and Broker Data},
  author = {Transportli},
  year   = {2026},
  url    = {https://transportli.com/auto-transport/price-index/},
  note   = {Snapshot November 2026. Hugging Face dataset Transportli/us-car-shipping-prices}
}
```

## About Transportli

Transportli is a marketplace that connects customers with independent, vetted carriers. It is not a carrier and does not move vehicles itself, and it earns revenue by matching a shipment with a carrier. Questions and corrections: <https://transportli.com/contact-us/>.

## Changelog

- November 2026: first release (six tables). LTL freight data lives in the companion dataset.
