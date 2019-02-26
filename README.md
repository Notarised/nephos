[![Build Status](https://travis-ci.com/aidtechnology/nephos.svg?branch=master)](https://travis-ci.com/aidtechnology/nephos)
[![Known Vulnerabilities](https://snyk.io/test/github/aidtechnology/nephos/badge.svg?targetFile=requirements.txt)](https://snyk.io/test/github/aidtechnology/nephos?targetFile=requirements.txt)
[![<Sonarcloud quality gate>](https://sonarcloud.io/api/project_badges/measure?project=aidtechnology_nephos&metric=alert_status)](https://sonarcloud.io/dashboard?id=aidtechnology_nephos)

# nephos

Library to deploy Hyperledger Fabric projects to Kubernetes.

Source resides here: https://github.com/aidtechnology/nephos
Documentation resides here: https://nephos.readthedocs.io

   * [Prerequisites](#prerequisites)
   * [Installation](#installation)
      * [Pip](#pip)
      * [Git repository](#git-repository)
         * [Virtual environment](#virtual-environment)
         * [Requirements](#requirements)
   * [Testing](#testing)
      * [Unit tests](#unit-tests)
   * [Usage](#usage)
   * [Examples](#examples)
      * [Development](#development)
      * [QA and Production](#qa-and-production)

## Prerequisites

This library requires an existing Kubernetes cluster.

For best results, use a real cluster (e.g. on a cloud like AWS, GCP, Azure, IBM Cloud, etc.). However, you may also use [Minikube](https://kubernetes.io/docs/setup/minikube/).

Either way, you will need to have the following tools installed:

- [python 3.7.0](https://www.python.org/downloads/release/python-370/) or above
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [helm](https://docs.helm.sh/using_helm/#installing-helm)

## Installation

### Pip

You can install nephos from PyPI by running:

    pip install nephos

### Git repository

You can also download the git repository with:

    git clone https://github.com/aidtechnology/nephos.git

And work locally by installing the following:

#### Virtual environment

This library currently only supports Python 3:

    python3 -m venv ./venv

    source ./venv/bin/activate

#### Requirements

All requirments are held in the requirements.txt file

    pip install -r requirements.txt

## Testing

### Unit tests

Once you have all requirments installed, all the unit tests should pass:

    PYTHONPATH=. pytest --cov=. --cov-report term-missing

## Usage

To use *nephos*, run the `deploy.py` executable CLI script.

For instance, you can see available commands/options by running:

    ./nephos/deploy.py --help

To install a full end-to-end fabric network, you can run:

    ./nephos/deploy.py -f ./PATH_TO_YOUR_SETTINGS/file.yaml fabric

You can also upgrade a network:

    ./nephos/deploy.py --upgrade -f ./PATH_TO_YOUR_SETTINGS/file.yaml fabric


## Examples

### Development

Example of development/QA/production(-ish) networks are provided in the examples folder.

To run the dev example from the git repository, use this command:

    ./nephos/deploy.py --verbose -f ./examples/dev/nephos_config.yaml fabric

> Note: The `nephos_config.yaml` is by default set to point to the `minikube` context (even for the `prod` example) to prevent accidental deployments to production clusters. If your K8S context name is different, please update this file.

### QA and Production

For the QA and production examples, you will need to replace the CA hostname to one pointing to your K8S cluster Ingress Controller  (e.g. NGINX or Traefik) IP address.

In a real cluster, you will wish to install an ingress controller and a certificate manager. We include in the repository two example Cluster Issuers (you will need to modify the email field in them) for the `cert-manager` deployment:

    helm install stable/nginx-ingress -n nginx-ingress --namespace ingress-controller

    helm install stable/cert-manager -n cert-manager --namespace cert-manager

    kubectl create -f ./examples/certManagerCI_staging.yaml

    kubectl create -f ./examples/certManagerCI_production.yaml

To use the Composer examples, you will need a Cloud system capable of a "ReadWriteMany" policy (e.g. "azurefile" on Azure).

### Minikube

Given that we may wish to test locally on Minikube, we will need to use a local ingress controller and ignore cert-manager in favour of self-cooked SSL certificates.

In `./examples` we include the `ca-nephos-local.*` self-signed certificates, created with OpenSSL as follows:

    openssl req -new -newkey rsa:4096 -days 3650 -nodes -x509 -subj "/C=IE/ST=Dublin/L=Dublin/O=AID:Tech/CN=ca.nephos.local" -keyout ca-nephos-local.key -out ca-nephos-local.crt

    openssl x509 -in ca-nephos-local.crt -out ca-nephos-local.pem -outform PEM

    kubectl create ns cas

    kubectl -n cas create secret tls ca--tls --cert=ca-nephos-local.crt --key=ca-nephos-local.key

We can save them to the `cas` namespace as follows

    cd ./examples

    kubectl create ns cas

    kubectl -n cas create secret tls ca--tls --cert=ca-nephos-local.crt --key=ca-nephos-local.key

We can then enable the ingress on minikube and update `/etc/hosts` with the IP of `minikube`:

    minikube addons enable ingress

    echo "$(minikube ip)  ca.nephos.local" | sudo tee -a /etc/hosts
