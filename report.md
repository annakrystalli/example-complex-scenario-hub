
<!-- README.md is generated from README.Rmd. Please edit that file -->

``` r
library(dplyr)
```

There’s a lot going on here. I’ve done some experiments which can be
found in this [fork and branch of
`example-complex-scenario-hub`](https://github.com/annakrystalli/example-complex-scenario-hub/tree/test-dataset-schema).
I made a few changes to ensure files conformed to hub standards adn
could be validated.

All the code can be found in `report.Rmd`.

## Differences in reading csvs between arrow and readr

The first issue appears to be a result of how `arrow` reads csvs,
especially compared to `readr`.

Below I create two csvs, both with `US` removed from the `location`
column. I then write them out using `arrow` and `readr`.

``` r
dir.create("test", showWarnings = FALSE)

arrow::read_csv_arrow(
  "model-output/HUBuni-simexamp/2021-03-07-HUBuni-simexamp.csv"
) %>%
  filter(location != "US") %>%
  slice(1:1000) %>%
  arrow::write_csv_arrow(file.path("test", "2021-03-07-HUBuni-simexamp-arrow.csv"))

readr::read_csv("model-output/HUBuni-simexamp/2021-03-07-HUBuni-simexamp.csv") %>%
  filter(location != "US") %>%
  slice(1:1000) %>%
  readr::write_csv(file.path("test", "2021-03-07-HUBuni-simexamp-readr.csv"))
#> Rows: 838656 Columns: 8
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr  (4): scenario_id, location, target, output_type
#> dbl  (3): horizon, output_type_id, value
#> date (1): origin_date
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

CSVs can often be difficult to work with because they don’t explicitly
encapsulate a schema like e.g. parquet files do.

Looking at the structure of the two files created:

##### `readr` csv

    origin_date,scenario_id,location,target,horizon,output_type,output_type_id,value
    2021-03-07,A-2021-03-05,02,inc death,1,quantile,0.01,0
    2021-03-07,A-2021-03-05,02,inc death,1,quantile,0.025,0
    2021-03-07,A-2021-03-05,02,inc death,1,quantile,0.05,1
    2021-03-07,A-2021-03-05,02,inc death,1,quantile,0.1,1
    2021-03-07,A-2021-03-05,02,inc death,1,quantile,0.15,2

##### `arrow` csv

    "origin_date","scenario_id","location","target","horizon","output_type","output_type_id","value"
    2021-03-07,"A-2021-03-05","02","inc death",1,"quantile",0.01,0
    2021-03-07,"A-2021-03-05","02","inc death",1,"quantile",0.025,0
    2021-03-07,"A-2021-03-05","02","inc death",1,"quantile",0.05,1
    2021-03-07,"A-2021-03-05","02","inc death",1,"quantile",0.1,1
    2021-03-07,"A-2021-03-05","02","inc death",1,"quantile",0.15,2

you’d think `arrow` had done a better job of encoding the character
nature of the `location` column.

However, it seems that `arrow` does a worse job of reading in the
`location` column, regardless of what function was used to write the
file.

`arrow` does NOT correctly read in location, regardless of what function
was used to write the file,

``` r
arrow::read_csv_arrow(
  file.path("test", "2021-03-07-HUBuni-simexamp-arrow.csv")
)
#> # A tibble: 1,000 × 8
#>    origin_date scenario_id  location target   horizon output_type output_type_id
#>    <date>      <chr>           <int> <chr>      <int> <chr>                <dbl>
#>  1 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.01 
#>  2 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.025
#>  3 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.05 
#>  4 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.1  
#>  5 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.15 
#>  6 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.2  
#>  7 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.25 
#>  8 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.3  
#>  9 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.35 
#> 10 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.4  
#> # ℹ 990 more rows
#> # ℹ 1 more variable: value <dbl>
```

``` r
arrow::read_csv_arrow(
  file.path("test", "2021-03-07-HUBuni-simexamp-readr.csv")
)
#> # A tibble: 1,000 × 8
#>    origin_date scenario_id  location target   horizon output_type output_type_id
#>    <date>      <chr>           <int> <chr>      <int> <chr>                <dbl>
#>  1 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.01 
#>  2 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.025
#>  3 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.05 
#>  4 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.1  
#>  5 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.15 
#>  6 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.2  
#>  7 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.25 
#>  8 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.3  
#>  9 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.35 
#> 10 2021-03-07  A-2021-03-05        2 inc dea…       1 quantile             0.4  
#> # ℹ 990 more rows
#> # ℹ 1 more variable: value <dbl>
```

while `readr` correctly reads in `location` regardless of what function
was used to write the file.

``` r
readr::read_csv(file.path("test", "2021-03-07-HUBuni-simexamp-readr.csv"))
#> Rows: 1000 Columns: 8
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr  (4): scenario_id, location, target, output_type
#> dbl  (3): horizon, output_type_id, value
#> date (1): origin_date
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#> # A tibble: 1,000 × 8
#>    origin_date scenario_id  location target   horizon output_type output_type_id
#>    <date>      <chr>        <chr>    <chr>      <dbl> <chr>                <dbl>
#>  1 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.01 
#>  2 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.025
#>  3 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.05 
#>  4 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.1  
#>  5 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.15 
#>  6 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.2  
#>  7 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.25 
#>  8 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.3  
#>  9 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.35 
#> 10 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.4  
#> # ℹ 990 more rows
#> # ℹ 1 more variable: value <dbl>
```

``` r
readr::read_csv(file.path("test", "2021-03-07-HUBuni-simexamp-arrow.csv"))
#> Rows: 1000 Columns: 8
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr  (4): scenario_id, location, target, output_type
#> dbl  (3): horizon, output_type_id, value
#> date (1): origin_date
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#> # A tibble: 1,000 × 8
#>    origin_date scenario_id  location target   horizon output_type output_type_id
#>    <date>      <chr>        <chr>    <chr>      <dbl> <chr>                <dbl>
#>  1 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.01 
#>  2 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.025
#>  3 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.05 
#>  4 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.1  
#>  5 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.15 
#>  6 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.2  
#>  7 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.25 
#>  8 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.3  
#>  9 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.35 
#> 10 2021-03-07  A-2021-03-05 02       inc dea…       1 quantile             0.4  
#> # ℹ 990 more rows
#> # ℹ 1 more variable: value <dbl>
```

### Why is this relevant?

I suspect what is going on in the case of hub transformations and the
errors @bsweger is experiencing is that arrow might be converting
`location` to integers on read (in at least one file where there is no
“US” value?) and then writing them out as integers as well.

I have worried from the beginning about issues with the transformations
if they are not schema aware and I think this may be an example of that.
I think the best way to handle this is to make sure that the schema is
preserved when writing out the parquet files and test for it as well.

## Differences in how schemas are applied to datasets between `csv` and `parquet`

Having said all that, there seem to be additional issues with arrow
datasets. All of this reminds of [edge cases I’ve come across
before](https://github.com/Infectious-Disease-Modeling-Hubs/hubData/issues/9)
and even reached out to the arrow team about but we were never able to
reproduce the issue when building up from smaller examples… I think we
may have just figured it out!

The problem seems to be parquet files and that applying the schema does
not appear to work in the same way as for csv files.

### Create a csv file with integer `location` column

``` r
readr::read_csv("model-output/hubcomp_examp/2021-03-07.csv") %>%
  filter(location != "US") %>%
  mutate(location = as.integer(location)) %>%
  readr::write_csv(
    file.path(
      "model-output/hubcomp-examp",
      "2021-03-07-hubcomp-examp.csv"
    )
  )
#> Rows: 838656 Columns: 8
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr  (4): scenario_id, location, target, output_type
#> dbl  (3): horizon, output_type_id, value
#> date (1): origin_date
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

### Create a parquet file with integer `location` column

``` r
readr::read_csv("model-output/hubcomp_examp/2021-03-07.csv") %>%
  filter(location != "US") %>%
  mutate(location = as.integer(location)) %>%
  arrow::write_parquet(
    file.path(
      "model-output/hubcomp-examp",
      "2021-03-07-hubcomp-examp.parquet"
    )
  )
#> Rows: 838656 Columns: 8
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr  (4): scenario_id, location, target, output_type
#> dbl  (3): horizon, output_type_id, value
#> date (1): origin_date
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

## Connect to whole hub

Dataset opens but we get the familiar error when filtering

``` r
hubData::connect_hub(".")
#> 
#> ── <hub_connection/UnionDataset> ──
#> 
#> • hub_name: "Complex Scenario Hub"
#> • hub_path: '.'
#> • file_format: "csv(3/3)" and "parquet(5/5)"
#> • file_system: "LocalFileSystem"
#> • model_output_dir: "./model-output"
#> • config_admin: 'hub-config/admin.json'
#> • config_tasks: 'hub-config/tasks.json'
#> 
#> ── Connection schema
#> hub_connection
#> origin_date: date32[day]
#> scenario_id: string
#> location: string
#> target: string
#> horizon: int32
#> output_type: string
#> output_type_id: double
#> value: double
#> model_id: string
#> age_group: string
#> target_date: date32[day]
```

``` r

hubData::connect_hub(".") %>%
  filter(location == "US") %>%
  hubData::collect_hub()
#> Error in `dplyr::collect()`:
#> ! NotImplemented: Function 'equal' has no kernel matching input types
#>   (string, int32)
```

### Connect to parquet only

Same error as above

``` r
hubData::connect_hub(".", file_format = "parquet")
#> 
#> ── <hub_connection/FileSystemDataset> ──
#> 
#> • hub_name: "Complex Scenario Hub"
#> • hub_path: '.'
#> • file_format: "parquet(5/5)"
#> • file_system: "LocalFileSystem"
#> • model_output_dir: "./model-output"
#> • config_admin: 'hub-config/admin.json'
#> • config_tasks: 'hub-config/tasks.json'
#> 
#> ── Connection schema
#> hub_connection with 5 Parquet files
#> origin_date: date32[day]
#> scenario_id: string
#> location: string
#> target: string
#> horizon: int32
#> age_group: string
#> target_date: date32[day]
#> output_type: string
#> output_type_id: double
#> value: double
#> model_id: string
```

``` r

hubData::connect_hub(".", file_format = "parquet") %>%
  filter(location == "US") %>%
  hubData::collect_hub()
#> Error in `dplyr::collect()`:
#> ! NotImplemented: Function 'equal' has no kernel matching input types
#>   (string, int32)
```

### Connect to csv only

Works!

``` r
hubData::connect_hub(".", file_format = "csv")
#> 
#> ── <hub_connection/FileSystemDataset> ──
#> 
#> • hub_name: "Complex Scenario Hub"
#> • hub_path: '.'
#> • file_format: "csv(3/3)"
#> • file_system: "LocalFileSystem"
#> • model_output_dir: "./model-output"
#> • config_admin: 'hub-config/admin.json'
#> • config_tasks: 'hub-config/tasks.json'
#> 
#> ── Connection schema
#> hub_connection with 3 csv files
#> origin_date: date32[day]
#> scenario_id: string
#> location: string
#> target: string
#> horizon: int32
#> output_type: string
#> output_type_id: double
#> value: double
#> model_id: string
```

``` r

hubData::connect_hub(".", file_format = "csv") %>%
  filter(location == "US") %>%
  hubData::collect_hub()
#> # A tibble: 29,952 × 9
#>    model_id      origin_date scenario_id  location target    horizon output_type
#>  * <chr>         <date>      <chr>        <chr>    <chr>       <int> <chr>      
#>  1 hubcomp_examp 2021-03-07  A-2021-03-05 US       inc death       1 quantile   
#>  2 hubcomp_examp 2021-03-07  A-2021-03-05 US       inc death       1 quantile   
#>  3 hubcomp_examp 2021-03-07  A-2021-03-05 US       inc death       1 quantile   
#>  4 hubcomp_examp 2021-03-07  A-2021-03-05 US       inc death       1 quantile   
#>  5 hubcomp_examp 2021-03-07  A-2021-03-05 US       inc death       1 quantile   
#>  6 hubcomp_examp 2021-03-07  A-2021-03-05 US       inc death       1 quantile   
#>  7 hubcomp_examp 2021-03-07  A-2021-03-05 US       inc death       1 quantile   
#>  8 hubcomp_examp 2021-03-07  A-2021-03-05 US       inc death       1 quantile   
#>  9 hubcomp_examp 2021-03-07  A-2021-03-05 US       inc death       1 quantile   
#> 10 hubcomp_examp 2021-03-07  A-2021-03-05 US       inc death       1 quantile   
#> # ℹ 29,942 more rows
#> # ℹ 2 more variables: output_type_id <dbl>, value <dbl>
```

I suspect this is most likely a result of the fact that the schema is
supplied differently when opening csv vs parquet datasets (including
under the hood in `connect_hub()`).

``` r
config_tasks <- hubUtils::read_config(".", "tasks")
schema <- hubData::create_hub_schema(
  config_tasks,
  partitions = list(model_id = arrow::utf8())
)
```

In csv datasets we use argument `col_types` to specify the schema:

``` r
arrow::open_dataset(
  "model-output",
  format = "csv",
  partitioning = "model_id",
  col_types = schema,
  unify_schemas = TRUE,
  strings_can_be_null = TRUE,
  factory_options = list(exclude_invalid_files = TRUE)
) %>%
  filter(location == "US") %>%
  hubData::collect_hub()
#> # A tibble: 29,952 × 9
#>    model_id        origin_date scenario_id  location target  horizon output_type
#>  * <chr>           <date>      <chr>        <chr>    <chr>     <int> <chr>      
#>  1 HUBuni-simexamp 2021-03-07  A-2021-03-05 US       inc de…       1 quantile   
#>  2 HUBuni-simexamp 2021-03-07  A-2021-03-05 US       inc de…       1 quantile   
#>  3 HUBuni-simexamp 2021-03-07  A-2021-03-05 US       inc de…       1 quantile   
#>  4 HUBuni-simexamp 2021-03-07  A-2021-03-05 US       inc de…       1 quantile   
#>  5 HUBuni-simexamp 2021-03-07  A-2021-03-05 US       inc de…       1 quantile   
#>  6 HUBuni-simexamp 2021-03-07  A-2021-03-05 US       inc de…       1 quantile   
#>  7 HUBuni-simexamp 2021-03-07  A-2021-03-05 US       inc de…       1 quantile   
#>  8 HUBuni-simexamp 2021-03-07  A-2021-03-05 US       inc de…       1 quantile   
#>  9 HUBuni-simexamp 2021-03-07  A-2021-03-05 US       inc de…       1 quantile   
#> 10 HUBuni-simexamp 2021-03-07  A-2021-03-05 US       inc de…       1 quantile   
#> # ℹ 29,942 more rows
#> # ℹ 2 more variables: output_type_id <dbl>, value <dbl>
```

while in parquet datasets we use argument `schema` to specify the
schema:

``` r
arrow::open_dataset(
  "model-output",
  format = "parquet",
  partitioning = "model_id",
  schema = schema,
  unify_schemas = TRUE,
  factory_options = list(exclude_invalid_files = TRUE)
) %>%
  filter(location == "US") %>%
  hubData::collect_hub()
#> Error in `dplyr::collect()`:
#> ! NotImplemented: Function 'equal' has no kernel matching input types
#>   (string, int32)
```

This does sort of beg the question of what is the purpose of schemas for
parquet files if they are not applied in the same way as for csv files.
I think this is a question for the arrow team.

## The importance of validation

Note also that none of these files would pass validation. Given the
brittleness of arrow, validation of data on the way in is very
important. It seems like a lot of this issues are coming from hubs that
have not been fully validated or have been transformed in a non schema
aware way. There is a limit to what can be done to fix issues with the
data at access time. It seems like this is especially important for
parquet files, which effectively include a schema as part of file
metadata.

``` r
hubValidations::validate_submission(
  hub_path = ".",
  file_path = file.path(
    "hubcomp-examp",
    "2021-03-07-hubcomp-examp.parquet"
  ),
  skip_submit_window_check = TRUE
)
#> ✔ example-complex-scenario-hub: All hub config files are valid.
#> ✔ 2021-03-07-hubcomp-examp.parquet: File exists at path
#>   'model-output/hubcomp-examp/2021-03-07-hubcomp-examp.parquet'.
#> ✔ 2021-03-07-hubcomp-examp.parquet: File name
#>   "2021-03-07-hubcomp-examp.parquet" is valid.
#> ✔ 2021-03-07-hubcomp-examp.parquet: File directory name matches `model_id`
#>   metadata in file name.
#> ✔ 2021-03-07-hubcomp-examp.parquet: `round_id` is valid.
#> ✔ 2021-03-07-hubcomp-examp.parquet: File is accepted hub format.
#> ✔ 2021-03-07-hubcomp-examp.parquet: Metadata file exists at path
#>   'model-metadata/hubcomp-examp.yaml'.
#> ✔ 2021-03-07-hubcomp-examp.parquet: File could be read successfully.
#> ✔ 2021-03-07-hubcomp-examp.parquet: `round_id_col` name is valid.
#> ✔ 2021-03-07-hubcomp-examp.parquet: `round_id` column "origin_date" contains a
#>   single, unique round ID value.
#> ✔ 2021-03-07-hubcomp-examp.parquet: All `round_id_col` "origin_date" values
#>   match submission `round_id` from file name.
#> ✔ 2021-03-07-hubcomp-examp.parquet: Column names are consistent with expected
#>   round task IDs and std column names.
#> ! 2021-03-07-hubcomp-examp.parquet: Column data types do not match hub schema.
#>   `location ` should be "character " not "integer ", `horizon ` should be
#>   "integer " not "double "
#> ✖ 2021-03-07-hubcomp-examp.parquet: `tbl` contains invalid values/value
#>   combinations.  Column `location` contains invalid values "2", "1", "5", "4",
#>   "6", "8", and "9".
```

``` r

hubValidations::validate_submission(
  hub_path = ".",
  file_path = file.path(
    "hubcomp-examp",
    "2021-03-07-hubcomp-examp.csv"
  ),
  skip_submit_window_check = TRUE
)
#> ✔ example-complex-scenario-hub: All hub config files are valid.
#> ✔ 2021-03-07-hubcomp-examp.csv: File exists at path
#>   'model-output/hubcomp-examp/2021-03-07-hubcomp-examp.csv'.
#> ✔ 2021-03-07-hubcomp-examp.csv: File name "2021-03-07-hubcomp-examp.csv" is
#>   valid.
#> ✔ 2021-03-07-hubcomp-examp.csv: File directory name matches `model_id` metadata
#>   in file name.
#> ✔ 2021-03-07-hubcomp-examp.csv: `round_id` is valid.
#> ✔ 2021-03-07-hubcomp-examp.csv: File is accepted hub format.
#> ✔ 2021-03-07-hubcomp-examp.csv: Metadata file exists at path
#>   'model-metadata/hubcomp-examp.yaml'.
#> ✔ 2021-03-07-hubcomp-examp.csv: File could be read successfully.
#> ✔ 2021-03-07-hubcomp-examp.csv: `round_id_col` name is valid.
#> ✔ 2021-03-07-hubcomp-examp.csv: `round_id` column "origin_date" contains a
#>   single, unique round ID value.
#> ✔ 2021-03-07-hubcomp-examp.csv: All `round_id_col` "origin_date" values match
#>   submission `round_id` from file name.
#> ✔ 2021-03-07-hubcomp-examp.csv: Column names are consistent with expected round
#>   task IDs and std column names.
#> ✔ 2021-03-07-hubcomp-examp.csv: Column data types match hub schema.
#> ✖ 2021-03-07-hubcomp-examp.csv: `tbl` contains invalid values/value
#>   combinations.  Column `location` contains invalid values "2", "1", "5", "4",
#>   "6", "8", and "9".
```

## Summary

- Any transformations should be schema aware and we should have
  integrity tests to ensure this.
- when setting up hubs from historic data we should indeed try to run
  validations on files (which may bump up the priority of
  <https://github.com/Infectious-Disease-Modeling-Hubs/hubValidations/issues/83>
  and
  <https://github.com/Infectious-Disease-Modeling-Hubs/hubverse-actions/issues/12>).
  This could help in identify potential issues earlier on. Having said
  that, that might not always be possible. In such cases however, I
  don’t think we can expect downstream data access functions to work
  seamlessly on data that has not been validated.
- Maybe we should just plug everything into a duckdb database!
