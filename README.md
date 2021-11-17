# BentoML Heroku Deployment Tool

[![Generic badge](https://img.shields.io/badge/Release-Alpha-<COLOR>.svg)](https://shields.io/)

Heroku is a popular platform as a service(PaaS) based on managed container system. It provides
a complete solution for building, running, and scaling applications.

This tool can be used as an Operator for the [Bento Cloud Deployment Tool](https://github.com/bentoml/cloud-deployment-tool/tree/prototype). See steps on how to add Heroku Deployment Tool as an Operator [here](#operator-deployment). 

## Prerequisites

- An active Heroku account configured on the machine with AWS CLI installed and configured
    - Install instruction: https://devcenter.heroku.com/articles/heroku-cli#getting-started
    - Login Heroku CLI: `$heroku login`
- Docker is installed and running on the machine.
    - Install instruction: https://docs.docker.com/install
- Built bento. 
    - Checkout [BentoML quickstart guide](https://github.com/bentoml/BentoML/blob/master/guides/quick-start/bentoml-quick-start-guide.ipynb) for how to get it started.


## Deploy to Heroku with [Bentoctl](https://github.com/bentoml/bentoctl)

1. Install BentoCTL
    ```bash
    $ pip install bentoctl
    ```

2. Add Heroku operator
    ```bash
    $ bentoctl operator add heroku
    ```

3. Deploy to Heroku use BentoCTL deploy command
    ```bash
    # Use the interactive mode
    $ bentoctl deploy 
    #
    # or provide deployment spec yaml. See BentoCTL repo for more detail
    $ bentoctl deploy --file my_deployment_spec.yaml
    
    #example response
    Login Heroku registry
    Create Heroku app btml-test-script
    Build Heroku app btml-test-script
    Deploy Heroku app btml-test-script
    === btml-test-script
    Auto Cert Mgmt: false
    Dynos:          web: 1
    Git URL:        https://git.heroku.com/btml-test-script.git
    Owner:          your-email@email.com
    Region:         us
    Repo Size:      0 B
    Slug Size:      0 B
    Stack:          container
    Web URL:        https://btml-test-script.herokuapp.com/
    ```

4. Get deployment information

    ```bash
    $ bentoctl describe my_deployment_spec.yaml
    ```

5. Make sample request

    ```bash
    $ curl -i \
        --header "Content-Type: application/json" \
        --request POST \
        --data '[[5.1, 3.5, 1.4, 0.2]]' \
        https://btml-test-script.herokuapp.com/predict

    # Output
    HTTP/1.1 200 OK
    Connection: keep-alive
    Content-Type: application/json
    X-Request-Id: f499b6d0-ad9b-4d79-850a-3dc058bd67b2
    Content-Length: 3
    Date: Mon, 28 Jun 2021 02:50:35 GMT
    Server: Python/3.7 aiohttp/3.7.4.post0
    Via: 1.1 vegur

    [0]%
    ```

6. Delete deployment with Bentoctl

    ```bash
    $ bentoctl delete my_deployment_spec.yaml
    ```


## Deploy to Heroku with opeartor scripts


1. Download Heroku deployment and Install the required packages
    
    ```bash
    $ git clone https://github.com/bentoml/heroku-deploy.git
    $ cd heroku-deploy
    $ pip install -r requirements.txt
    ```


2. Create Heroku deployment with deployment

    Run deploy script in the command line:

    ```bash
    $ BENTO_BUNDLE_PATH=$(bentoml get IrisClassifier:latest --print-location -q)
    $ ./deploy $BENTO_BUNDLE_PATH test-script heroku_config.json

    # Output
    Login Heroku registry
    Create Heroku app btml-test-script
    Build Heroku app btml-test-script
    Deploy Heroku app btml-test-script
    === btml-test-script
    Auto Cert Mgmt: false
    Dynos:          web: 1
    Git URL:        https://git.heroku.com/btml-test-script.git
    Owner:          yubz86@gmail.com
    Region:         us
    Repo Size:      0 B
    Slug Size:      0 B
    Stack:          container
    Web URL:        https://btml-test-script.herokuapp.com/
    ```

3. Get deployment information

    ```bash
    $ ./describe test-script

    # Output
    === btml-test-script
    Auto Cert Mgmt: false
    Dynos:          web: 1
    Git URL:        https://git.heroku.com/btml-test-script.git
    Owner:          yubz86@gmail.com
    Region:         us
    Repo Size:      0 B
    Slug Size:      0 B
    Stack:          container
    Web URL:        https://btml-test-script.herokuapp.com/
    ```

4. Make sample request against deployed service

    ```bash
    $ curl -i \
        --header "Content-Type: application/json" \
        --request POST \
        --data '[[5.1, 3.5, 1.4, 0.2]]' \
        https://btml-test-script.herokuapp.com/predict

    # Output
    HTTP/1.1 200 OK
    Connection: keep-alive
    Content-Type: application/json
    X-Request-Id: f499b6d0-ad9b-4d79-850a-3dc058bd67b2
    Content-Length: 3
    Date: Mon, 28 Jun 2021 02:50:35 GMT
    Server: Python/3.7 aiohttp/3.7.4.post0
    Via: 1.1 vegur

    [0]%
    ```

5. Delete Heroku deployment

    ```bash
    $ ./delete test-script

    # Output
    Removing app btml-test-script
    ```

## Deployment command reference

### Create a deployment

Use command line

```bash
$ ./deploy <Bento_bundle_path> <Deployment_name> <Config_JSON, default is heroku_config.json>
```

Example:

```bash
BENTO_BUNDLE_PATH=${bentoml get IrisClassifier:latest --print-location -q)
$ ./deploy $BENTO_BUNDLE_PATH my_first_deployment heroku_config.json
```

Use Python API

```python
from heroku_deploy import deploy

deploy_heroku(BENTO_BUNDLE_PATH, DEPLOYMENT_NAME, HEROKU_CONFIG)
```
* where `HEROKU_CONFIG` is a dictionary with keys for `"dyno_counts"` and `"dyno_type"`

#### Available options

* `dyno_counts`: Amount of dyno running for the deployment, see https://devcenter.heroku.com/articles/dyno-types#default-scaling-limits for more information.
* `dyno_type`: Heroku dyno(instance) type, see https://devcenter.heroku.com/articles/dyno-types for more information

### Update a deployment

Use command line

```bash
$ ./update <Bento_bundle_path> <Deployment_name> <Config_JSON>
```

Use Python API

```python
from heroku_deploy import update

update(BENTO_BUNDLE_PATH, DEPLOYMENT_NAME, HEROKU_CONFIG)
```
* where `HEROKU_CONFIG` is a dictionary with keys for `"dyno_counts"` and `"dyno_type"`

### Get a deployment's status and information

Use command line

```bash
$ ./describe <Deployment_name>
```

Use Python API

```python
from heroku_deploy import describe

describe(DEPLOYMENT_NAME)
```

### Delete a deployment

Use command line

```bash
$ ./delete <Deployment_name>
```

Use Python API

```python
from heroku_deploy import delete

delete(DEPLOYMENT_NAME)
```
