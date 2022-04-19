# Single vs Monolith Serverless Function

The goal of this sample application is to show how you can use a single serverless function or a monolith single server function that performs all the tasks.
It's part of our blog post on Single or Monolith Serverless Functions - What should you choose?

## Application

This sample application is a Stock Keeper application that allows the user to check the stock of a particular product and re-order them when the stock is low.
To keep the application simple, the stock quantity is returned as a random number generated by the function and the reorder date is also a random date.
In reality this can be a fission function that connects to a database and gets the actual value for the product along with another function that places the order and returns the estimated delivery date.

The application is written in Python, uses flask along with standard Python libraries.

### Single Function

This folder contains 3 Fission functions:

* `app.py` - *web UI for the user to interact*
* `getStock.py` - *function that returns the stock count*
* `reorder.py` - *function that returns the estimated delivery date*

The user launches the application and enters a product code. *abc123 or xyz123*.
It internally calls getstock fission function that returns a stock.
If the returned stock is less than 5, the user will see another form or reorder.
The user will enter the reorder quantity which will be passed to the reorder function that will return the estimated delivery date.

### Monolith

The monolight application contains only one fission function that is `app.py` which does all the work.

## Steps to Execute

### Single Function

Create a Python environment

```bash
fission environment create --name python --image fission/python-env --builder fission/python-builder:latest
```

Create a zip archive with app.py and templates director

```bash
zip -r main.zip app.py templates
```

Create a package

```bash
fission package create --name frontend-pkg --sourcearchive main.zip --env python
```

Create the `app`, `getstock` and `reorder` fission functions

```bash
fission fn create --name app --pkg frontend-pkg --entrypoint "app.main" 
fission fn create --name reorder --env python --code reorder.py
fission fn create --name getstock --env python --code getstock.py
```

Create three routes so that we can access these functions

```bash
fission route create --name single --method POST --method GET --url /main --function app
fission route create --name getstock --method POST --url /getstock --function getstock
fission route create --name reorder --method POST --url /reorder --function reorder
```

Port forward the service to access it from browser

```bash
kubectl port-forward svc/router 8888:80 -nfissionouter 8888:80 -nfission
```

Access the application at http://localhost:8888/main

### Monolith

Create a Python environment

```bash
fission environment create --name python --image fission/python-env --builder fission/python-builder:latest
```

Create a zip archive with app.py and templates director

```bash
zip -r monolith.zip app.py templates
```

Create a package

```bash
fission package create --name frontend-monolith --sourcearchive monolith.zip --env python
```

Create a fission function

```bash
fission fn create --name monolith --pkg frontend-monolith --entrypoint "app.main" 
```

Create a route

```bash
fission route create --name monolith --method POST --method GET --url /monolith --function monolith 
```

Port forward the service to access it from browser

```bash
kubectl port-forward svc/router 8888:80 -nfissionouter 8888:80 -nfission
```

Access the application at http://localhost:8888/monolith

## Fission Spec

### Single

```bash
fission spec init
fission environment create --name python --image fission/python-env --builder fission/python-builder:latest --spec
fission package create --name frontend-pkg --sourcearchive main.zip --env python --spec
fission fn create --name app --pkg frontend-pkg --entrypoint "app.main" --spec
fission fn create --name reorder --env python --code reorder.py --spec
fission fn create --name getstock --env python --code getstock.py --spec
fission route create --name single --method POST --method GET --url /main --function app --spec
fission route create --name getstock --method POST --url /getstock --function getstock --spec
fission route create --name reorder --method POST --url /reorder --function reorder --spec
```

### Monolith

```bash
fission spec init
fission environment create --name python --image fission/python-env --builder fission/python-builder:latest --spec
fission package create --name frontend-monolith --sourcearchive monolith.zip --env python --spec
fission fn create --name monolith --pkg frontend-monolith --entrypoint "app.main" --spec
fission route create --name monolith --method POST --method GET --url /monolith --function monolith --spec
```