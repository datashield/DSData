# DataSHIELD Data

[![Build Status](https://travis-ci.com/datashield/DSData.svg?branch=main)](https://travis-ci.com/datashield/DSData)

DataSHIELD test datasets.

## How to add new test datasets

1. prepare and save the test datasets in the data folder : provide 3 different subsets of **simulated** data

```
# prepare a dataset 'TestDataset1'
# ...

# save it a compressed file
save(TestDataset1, file="data/TestDataset1.rda", compress = TRUE, compression_level = 9)

# do the same for 'TestDataset2' and 'TestDataset3'
```

2. document these test datasets

* Declare and document the datasets in the `R/data.testdataset.R` file
* Build the project
* Install and test

## How to use test datasets

To load the test datasets with base name 'TestDataset'

```
data('TestDataset1')
data('TestDataset2')
data('TestDataset3')
```

## How to provision a DataSHIELD infrastructure

### DSLite

[DSLite](https://github.com/datashield/DSLite) is a pure R implementation of the DataSHIELD infrastructure. A DSLite server 
interfaces operations on local datasets.

```
# load some test datasets in global environment
library(DSData)
data('CNSIM1')
data('CNSIM2')
data('CNSIM3')

# build a DSLite server with the datasets inside
library(DSLite)
dslite.server <- DSLite::newDSLiteServer(tables=list(table1=CNSIM1, table2=CNSIM2, table3=CNSIM3))

# build DS login information
builder <- DSI::newDSLoginBuilder()
builder$append(server = 'server1', driver = 'DSLiteDriver', url = 'dslite.server')
builder$append(server = 'server2', driver = 'DSLiteDriver', url = 'dslite.server')
builder$append(server = 'server3', driver = 'DSLiteDriver', url = 'dslite.server')
logindata <- builder$build()

# do login and table assignment
conns <- datashield.login(logindata)
datashield.assign.table(conns, 'D', table = list(server1='table1', server2='table2', server3='table3'))
# ...
datashield.logout(conns)
```

### Opal

Use [opalr](https://github.com/obiba/opalr) to seed a Opal server with test dataset.

```
# load some test datasets in global environment
library(DSData)
data('CNSIM1')
data('CNSIM2')
data('CNSIM3')

# seed a opal server
# connect as an administrator
opal.url <- 'https://opal-demo.obiba.org'
opal.user <- 'administrator'
opal.password <- 'password'
library(opalr)
o <- opal.login(username = opal.user, password = opal.password, url = opal.url)
# create the project if it does not exist, with the appropriate database
if (!opal.project_exists(o, 'test')) {
  opal.project_create(o, 'test', database = 'mongodb')
}
# make sure the dataset is a tibble with a column of identifiers
table <- tibble::as_tibble(CNSIM1)
table$ID <- rownames(CNSIM1)
opal.table_save(o, tibble = table, project = 'test', table = 'CNSIM1', id.name = 'ID', overwrite = TRUE, force = TRUE)
# repeat for CNSIM2 and CNSIM3 ...
opal.logout(o)

# build DS login information
# do not connect as an adnimistrator
opal.user <- 'dsuser'
opal.password <- 'password'
builder <- DSI::newDSLoginBuilder()
builder$append(server = 'server1', driver = 'OpalDriver', url = opal.url, user = opal.user, password = opal.password)
builder$append(server = 'server2', driver = 'OpalDriver', url = opal.url, user = opal.user, password = opal.password)
builder$append(server = 'server3', driver = 'OpalDriver', url = opal.url, user = opal.user, password = opal.password)
logindata <- builder$build()

# do login and table assignment
conns <- datashield.login(logindata)
datashield.assign.table(conns, 'D', table = list(server1='test.CNSIM1', server2='test.CNSIM2', server3='test.CNSIM3'))
# ...
datashield.logout(conns)
```


Article about DataSHIELD:
* [DataSHIELD: taking the analysis to the data, not the data to the analysis](https://doi.org/10.1093/ije/dyu188)
