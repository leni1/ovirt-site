---
title: Integrating Kibana/Elasticsearch on top of OpenShift with oVirt Engine SSO
author: rnori
tags: community, documentation, howto, OpenShift, oVirt, sso
date: 2017-04-04 11:40:00 CET
comments: true
published: true
---

Integrating Kibana/Elasticsearch on top of OpenShift with oVirt Engine SSO

Step by step instructions on how to integrate Kibana/Elasticsearch on top of OpenShift with oVirt Engine SSO

READMORE

## Background

The goal is to integrate Kibana/Elasticsearch on top of OpenShift with oVirt Engine SSO, so existing engine users can access Kibana/Elasticsearch without reauthentication (we don't need to maintain authentication configuration separately for oVirt Engine and Kibana/Elasticsearch) 

## Prerequisites

* A fully working and configured oVirt engine instance
* A fully working and configured instance of Kibana/Elasticsearch on top of OpenShift

## Installing Kibana/Elasticsearch and OpenShift backend

Install Kibana/Elasticsearch/OpenShift on CentOS7 or RHEL 7.3 as described in https://github.com/ViaQ/Main/blob/master/README-mux.md

## Installing oVirt Engine

Setup oVirt Engine on a separate host ovirt-engine.example.com as described in https://www.ovirt.org/download/

## Setting up oVirt Engine certificate on OpenShift machine

Get the oVirt Engine CA as described here https://www.ovirt.org/documentation/how-to/guest-console/connect-to-spice-console-without-portal/

```sh
scp root@${OVIRT}:/etc/pki/ovirt-engine/ca.pem ${CA_FILE}
```

Add the certificate to system wide trusted certificates
Copy certificate to /etc/pki/ca-trust/source/anchors/ and run update-ca-trust

## Register a new sso client on ovirt-engine host

Run the client registration tool ovirt-register-sso-client to register a new sso client. The tool will prompt the user to enter the client id, location of the client certificate (downloaded to oVirt engine host) and the callback url prefix. Make note of the client id and client secret generated by the tool. The client id and client secret need to be entered in the master configuration file on the OpenShift host to configure authentication with oVirt Engine.

## Setup oauthconfig on Kibana/Elasticsearch/OpenShift host 

On Kibana/Elasticsearch/OpenShift host edit /etc/origin/master/master-config.yaml to setup oauthconfig as below

```yaml
oauthConfig:
  assetPublicURL: https://openstack.example.com:8443/console/
  grantConfig:
    method: auto
  identityProviders:
  - challenge: false
    login: true
    mappingMethod: claim
    name: my_openid_connect
    provider:
      apiVersion: v1
      kind: OpenIDIdentityProvider
      clientID: <client id specified in previous step>
      clientSecret: <client id generated in previous step>
      extraScopes:
      - ovirt-app-api
      - ovirt-ext=auth:sequence-priority=~
      extraAuthorizeParameters:
        include_granted_scopes: "true"
claims:
        id:
        - custom_id_claim
        - sub
        preferredUsername:
        - preferred_username
        - email
        name:
        - nickname
        - given_name
        - name
        email:
        - custom_email_claim
        - email
      urls:
        authorize: https://ovirt-engine.example.com/ovirt-engine/sso/oauth/authorize
        token: https://ovirt-engine.example.com/ovirt-engine/sso/oauth/token
  masterCA: ca-bundle.crt
  masterPublicURL: https://openstack.example.com:8443
  masterURL: https://openstack.example.com:8443
  sessionConfig:
    sessionMaxAgeSeconds: 3600
    sessionName: ssn
    sessionSecretsFile: /etc/origin/master/session-secrets.yaml
  tokenConfig:
    accessTokenMaxAgeSeconds: 86400
    authorizeTokenMaxAgeSeconds: 500
```
## Restart oVirt Engine

```ssh
service ovirt-engine restart
```

## Restart origin-master and origin-node on OpenShift host

```ssh
service origin-master restart
service origin-node restart
```

## Configure hostnames

Make sure the hosts are reachable by their hostnames if required add host aliases in /etc/hosts

```config
10.16.19.48 openstack.example.com
10.10.116.110 ovirtengine.example.com
```

## Grant permissions

The user needs to be granted permissions manually in OpenShift, using the console UI or the command line, so that the user can view the data in Kibana.
Accessing https://kibana.example.com should redirect to the engine login page
Enter credentials and login will redirect user back to Kibana

