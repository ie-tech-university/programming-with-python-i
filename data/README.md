# Course datasets

## `countries.json`

A snapshot of 216 countries and territories. It has the same JSON shape as the
REST Countries API v3.1 (`name.common`, `name.official`, `capital`, `region`,
`subregion`, `population`, `area`, `languages`, `currencies`, …), which the
course used until that API version was shut down in 2026 (v5 now needs an API
key).

Sessions 12, 14 and 19 fetch it over HTTP like any other JSON API:

```
https://raw.githubusercontent.com/ie-tech-university/programming-with-python-i/main/data/countries.json
```

If you are offline, read the same file from your clone of the repository
instead (`data/countries.json`).

### Sources and licences

- Country names, codes, capitals, regions, languages, currencies and area:
  [mledoze/countries](https://github.com/mledoze/countries), available under the
  [Open Database License (ODbL) 1.0](https://opendatacommons.org/licenses/odbl/1-0/).
  This derived file is shared under the same licence.
- Population (2025, indicator `SP.POP.TOTL`):
  [World Bank Open Data](https://data.worldbank.org/indicator/SP.POP.TOTL),
  [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

The file only includes countries and territories that have a World Bank
population figure. The other 34 entries in mledoze/countries are left out
(mostly dependent territories, plus places the World Bank does not report on,
such as Taiwan and Vatican City).
