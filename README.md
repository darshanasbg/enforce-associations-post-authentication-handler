# enforce-associations-post-authentication-handler

## Background
Post authentication handlers are a set of handlers which executes upon successful authentication through WSO2 Identity Server (IS) authentication framework. For example, if you do a Single Sign On(SSO) your application through an IS instance or any WSO2 product which supports SSO authentication then upon successful authentication these handlers will get executed.

An important fact you need to keep in mind is even though you successfully finished your authentication steps (basic, totp etc) you will still not be an authenticated person until you pass the post authentication flow. Existing post authenticators are AuthorizationHandler and missing mandatory claim handler which are getting executed upon successful authentication.

## Enforce Associations
WSO2 IS supports to create a mapping (ie. association) between federated identities and local identities.

And when using federated authentication, instead of sending federated user's subject\attributes, WSO2 IS can send associated local user's subject
attribute if there is an associated account. This is done by enabling `Assert identity using mapped local subject identifier` flag in the [service provider configuration](https://docs.wso2.com/display/IS580/Configuring+Local+and+Outbound+Authentication+for+a+Service+Provider).

This capability is handled by [PostAuthAssociationHandler](https://github.com/wso2/carbon-identity-framework/blob/v5.12.387/components/authentication-framework/org.wso2.carbon.identity.application.authentication.framework/src/main/java/org/wso2/carbon/identity/application/authentication/framework/handler/request/impl/PostAuthAssociationHandler.java), but it's treating mapping from federated subject to local subject as an optional step. ie. It's convert federated subject to local subject if there is a association for that federated subject. Otherwise its returns federated subject to the service provider.

As some service providers always expects local subjects as those cannot handle federated subjects or those expects some additional attributes\features that need local subject, in some cases its need to break the authentication flow if there is no association exists.

This extension can be used to,
* Enforce associations (and break the authentication flow) for federated subjects
* For service providers that has enabled "Assert identity using mapped local subject identifier" flag
* During federated logins

## Table of contents

- [Tested versions](#tested-versions)
- [How to build](#how-to-build)
- [How to deploy](#how-to-deploy)
- [How to test](#how-to-test)

## Tested versions
* IS 5.8.0

## How to build

### Build from source

#### Prerequisites

* [Maven](https://maven.apache.org/download.cgi)
* [Java](http://www.oracle.com/technetwork/java/javase/downloads)

1. Get a clone or download source from this repository
2. Run the Maven command mvn clean install from base pom.

## How to deploy

1. Build the sample as mentioned in [How to build](#how-to-build) section.
2. Copy `target/org.wso2.is.sample.post.authn.handler.association.enforcer-1.0.0-SNAPSHOT.jar` to `<IS_HOME>/repository/components/dropins/` directory.
3. Open `<IS_HOME>/repository/conf/identity/identity.xml`.
4. Search for default association handle: `PostAuthAssociationHandler` and turn if off by setting `enable` attribute to `false` as follows,
```
        <EventListener type="org.wso2.carbon.identity.core.handler.AbstractIdentityHandler"
                       name="org.wso2.carbon.identity.application.authentication.framework.handler.request.impl.PostAuthAssociationHandler"
                       orderId="25" enable="false"/>
```
5. Add new `EventListener` for the new `AssociationEnforcerPostAuthenticationHandler` as follows,
```
        <EventListener type="org.wso2.carbon.identity.core.handler.AbstractIdentityHandler"
                       name="org.wso2.carbon.identity.post.authn.handler.association.enforcer.AssociationEnforcerPostAuthenticationHandler"
                       orderId="25" enable="true"/>
```
6. Save and close `<IS_HOME>/repository/conf/identity/identity.xml`.
7. Start the server.

## How to test

Try to login with a federated user, that does not have an association. It will show an [error page](https://localhost:9443/authenticationendpoint/retry.do?status=Authentication+attempt+failed.&statusMsg=Authentication+failed.+Cannot+find+mapped+local+user.&tenantDomain=carbon.super).
