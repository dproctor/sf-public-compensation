# SF Public compensation data

## Development

To run the analysis

```
# Download data set from https://data.sfgov.org/City-Management-and-Ethics/Employee-Compensation/88g8-5mnd/about_data to sources/sf/employee_compensation.csv.

nix develop
npm install
npm run sources
npm run dev
```

## Data sources

### `sources/sf`

`Employee_Compensation_20260814.csv` is public compensation data, for all SF employees, going back to 2014. From [DataSF](https://data.sfgov.org/City-Management-and-Ethics/Employee-Compensation/88g8-5mnd/about_data), accessed 2026-08-14.
