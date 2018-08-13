---
layout: docs_page
weight: 2
title: Events API to System Log API Migration Guide
excerpt: How to migrate from the deprecated Events API to its System Log API replacement.
redirect_from: "/docs/guides/events-api-migration.html"
---

# Event Migration Guide

## Introduction

This migration guide aims to help organizations migrate from the deprecated [Events API](https://developer.okta.com/docs/api/resources/events) to its [System Log API](https://developer.okta.com/docs/api/resources/system_log) replacement. It highlights some of the key semantic, structural, and operational similarities and differences between the two APIs to ease the transition process.

Note that this guide does not attempt to cover specific use cases, detailed patterns of interaction or the intricacies of particular query parameters. For that, it is suggested to see the corresponding sections in the [System Log API](https://developer.okta.com/docs/api/resources/system_log) documentation.

## Key Differences

This section explores the notable differences between the two APIs, the resources and representations they expose, and how they are organized. These differences reflect a complete reimplementation of the system log platform. As a result, one can expect both gross and subtle differences.

### Resources

Both of the RESTful APIs provide a single read-only resource:

| Events APIs            | System Log API         |
| ---------------------- | ---------------------- |
| `GET` `/api/v1/events` | `GET` `/api/v1/logs`   |

For brevity, the Events API will often be referred to as `/events` and the System Log API as `/logs`.

### Data Structure

Each of the API resources has an associated data structure, also referred to as the resource "representation" or data model. The System Log API's representation is the [LogEvent object](https://developer.okta.com/docs/api/resources/system_log#logevent-object). It captures the occurrence of notable system events. The Events API's representation is the [Event object](https://developer.okta.com/docs/api/resources/events#event-model). LogEvent has more structure and a much richer set of data elements than Event. It is one of the principal improvements of the System Log API over the Events API.

One of the most important attributes of an event in the Okta system is its "event type" designation. 

In the Events API, the [`action.objectType` attribute](https://developer.okta.com/docs/api/resources/events#action-object) attribute denotes the event type. In the Logs API, the [`eventType` attribute](https://developer.okta.com/docs/api/resources/system_log#event-types) represents the event type. The values in each of these fields are generally different, although there is some overlap for historical purposes. In the interest of easing the transition from the Events API to the System Log API, LogEvent's [`legacyEventType` attribute](https://developer.okta.com/docs/api/resources/system_log#attributes) identifies the equivalent Event `action.objectType` value. The [Event Type Mapping](#event-type-mapping) section of this guide provides a static mapping of Events API event types to System Log API event types.

Another essential difference between the two systems is the manner in which detailed information is encoded. The Events API textually encodes the specifics of a particular event instance into the [`action.message` attribute](https://developer.okta.com/docs/api/resources/events#action-object). This encoding burdened consumers with having to correctly parse data themselves and led to brittleness in downstream systems when wording changed. The System Log API expands and enriches the data model to support storing these values as atomic, independent attributes. Context objects, such as the [AuthenticationContext object](https://developer.okta.com/docs/api/resources/system_log#authenticationcontext-object) and [GeographicalContext objects](https://developer.okta.com/docs/api/resources/system_log#geographicalcontext-object) objects, provide attributes that are common across event types. The [DebugContext object](https://developer.okta.com/docs/api/resources/system_log#debugcontext-object) houses event-type-specific attributes.

#### Event / LogEvent Comparison Example

This section illustrates the differences between the two APIs data model and attribute contents using a single admin user login event as captured by both systems as an illustrative example.

##### Events API Event

The following is an example of an Event API successful admin login event instance with the event type `app.admin.sso.login.success`:

```json
{
   "eventId":"tev2FSkoWAARbKaFBBfPPXUWA1533221531000",
   "sessionId":"102PfloXybbT3q1IOdqDAQoeQ",
   "requestId":"W2Mam7t4pcvodL-w@kNCrQAABSM",
   "published":"2018-08-02T14:52:11.000Z",
   "action":{
      "message":"User logged in to the Admin app",
      "categories":[

      ],
      "objectType":"app.admin.sso.login.success",
      "requestUri":"/admin/sso/request"
   },
   "actors":[
      {
         "id":"00u1qmc3wcC6KIsgi0g7",
         "displayName":"Jane Doe",
         "login":"jdoe@example.com",
         "objectType":"User"
      },
      {
         "id":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_3)...",
         "displayName":"CHROME",
         "ipAddress":"99.225.99.159",
         "objectType":"Client"
      }
   ],
   "targets":[
      {
         "id":"00u1qmc3wcC6KIsgi0g7",
         "displayName":"Jane Doe",
         "login":"jdoe@example.com",
         "objectType":"User"
      },
      {
         "id":"0oa1qmc3w1qLYTPVn0g7",
         "displayName":"Okta Administration",
         "objectType":"AppInstance"
      }
   ]
}
```

The data structure is both narrow in its top-level attributes and shallow in object attribute nesting.

##### System Log API LogEvent

The following is the corresponding event of a successful user session accessing the admin app as captured in the System Log API with the event type `user.session.access_admin_app`:

```json
   {
      "actor":{
         "id":"00u1qmc3wcC6KIsgi0g7",
         "type":"User",
         "alternateId":"jdoe@example.com",
         "displayName":"Jane Doe",
         "detailEntry":null
      },
      "client":{
         "userAgent":{
            "rawUserAgent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_3)...",
            "os":"Mac OS X",
            "browser":"CHROME"
         },
         "zone":"null",
         "device":"Computer",
         "id":null,
         "ipAddress":"99.225.99.159",
         "geographicalContext":{
            "city":"Toronto",
            "state":"Ontario",
            "country":"Canada",
            "postalCode":"M6G",
            "geolocation":{
               "lat":43.6655,
               "lon":-79.4204
            }
         }
      },
      "authenticationContext":{
         "authenticationProvider":null,
         "credentialProvider":null,
         "credentialType":null,
         "issuer":null,
         "interface":null,
         "authenticationStep":0,
         "externalSessionId":"102PfloXybbT3q1IOdqDAQoeQ"
      },
      "displayMessage":"User accessing Okta admin app",
      "eventType":"user.session.access_admin_app",
      "outcome":{
         "result":"SUCCESS",
         "reason":null
      },
      "published":"2018-08-02T14:52:11.272Z",
      "securityContext":{
         "asNumber":null,
         "asOrg":null,
         "isp":null,
         "domain":null,
         "isProxy":null
      },
      "severity":"INFO",
      "debugContext":{
         "debugData":{
            "requestUri":"/admin/sso/request"
         }
      },
      "legacyEventType":"app.admin.sso.login.success",
      "transaction":{
         "type":"WEB",
         "id":"W2Mam7t4pcvodL-w@kNCrQAABSM",
         "detail":{

         }
      },
      "uuid":"b5ef15a1-e78f-4125-b425-cc10f04e24f3",
      "version":"0",
      "request":{
         "ipChain":[
            {
               "ip":"99.225.99.159",
               "geographicalContext":{
                  "city":"Toronto",
                  "state":"Ontario",
                  "country":"Canada",
                  "postalCode":"M6G",
                  "geolocation":{
                     "lat":43.6655,
                     "lon":-79.4204
                  }
               },
               "version":"V4",
               "source":null
            }
         ]
      },
      "target":[
         {
            "id":"0ua1qmc3wf2xDawpN0g7",
            "type":"AppUser",
            "alternateId":"unknown",
            "displayName":"Jane Doe",
            "detailEntry":null
         }
      ]
   }
```

Immediately obvious is the increased structure and additional embedded information. For example, the `client.geographicalContext` captures the geolocation of the client accessing the system which unavailable in the `/events` API.

##### Event / System Log API Event Attribute Mapping

Given the above events from each API, the following compares each leaf-level attribute. [JSON Pointer](https://tools.ietf.org/html/rfc6901) notation is used to specify the compared attribute values.

| Event                     | LogEvent                                                 | Notes                                        |
| ------------------------- | -------------------------------------------------------- | -------------------------------------------- |
| `/action/categories`      |                                                          | Always empty |
| `/actors/0/login`         | `/actor/alternateId`                                     | Generally same values |
| `/actors/0/displayName`   | `/actor/displayName`                                     | Generally same values |
| `/actors/0/id`            | `/actor/id`                                              | Generally same values |
| `/actors/0/objectType`    | `/actor/type`                                            | Generally same values |
|                           | `/authenticationContext/authenticationStep`              | New |
| `/sessionId`              | `/authenticationContext/externalSessionId`               | New |
| `/actors/1/objectType`    | `/client/device`                                         | Different values |
|                           | `/client/geographicalContext/city`                       | New |
|                           | `/client/geographicalContext/country`                    | New |
|                           | `/client/geographicalContext/geolocation`                | New |
|                           | `/client/geographicalContext/geolocation/lat`            | New |
|                           | `/client/geographicalContext/geolocation/lon`            | New |
|                           | `/client/geographicalContext/postalCode`                 | New |
|                           | `/client/geographicalContext/state`                      | New |
| `/actors/1/ipAddress`     | `/client/ipAddress`                                      | New |
| `/actors/1/displayName`   | `/client/userAgent/browser`                              | New |
|                           | `/client/userAgent/os`                                   | New |
| `/actors/1/id`            | `/client/userAgent/rawUserAgent`                         | New |
|                           | `/client/zone`                                           | New |
| `/action/requestUri`      | `/debugContext/debugData/requestUri`                     | New |
| `/action/message`         | `/displayMessage`                                        | Generally less content |
| `/action/objectType`      | `/eventType`                                             | Generally contains different ids (see /Event Type Mappings(#event-type-mappings)) |
|                           | `/legacyEventType`                                       | Contains `/action/objectType` as its value |
|                           | `/outcome/result`                                        | Contains value that is encoded in `/action/objectType` suffix |
| `/published`              | `/published`                                             | Contains slightly different values |
|                           | `/request/ipChain/0/geographicalContext`                 | New |
|                           | `/request/ipChain/0/geographicalContext/city`            | New |
|                           | `/request/ipChain/0/geographicalContext/country`         | New |
|                           | `/request/ipChain/0/geographicalContext/geolocation`     | New |
|                           | `/request/ipChain/0/geographicalContext/geolocation/lat` | New |
|                           | `/request/ipChain/0/geographicalContext/geolocation/lon` | New |
|                           | `/request/ipChain/0/geographicalContext/postalCode`      | New |
|                           | `/request/ipChain/0/geographicalContext/state`           | New |
| `/actors/1/ipAddress`     | `/request/ipChain/0/ip`                                  | New |
|                           | `/request/ipChain/0/version`                             | New |
|                           | `/securityContext`                                       | New |
|                           | `/severity`                                              | New |
| `/targets/0/displayName`  | `/target/0/displayName`                                  | Generally same values |
| `/targets/0/id`           | `/target/0/id`                                           | Generally same values |
| `/targets/0/login`        | `/target/0/alternateId`                                  | Generally same values |
| `/targets/0/objectType`   | `/target/0/type`                                         | Generally same values |
|                           | `/transaction/detail`                                    | Generally same values |
| `/requestId`              | `/transaction/id`                                        | When `/transaction/type` is `WEB` |
|                           | `/transaction/type`                                      | New |
| `/eventId`                | `/uuid`                                                  | Different values |
|                           | `/version`                                               | New |

A key point to note is that `performed_by` is usually `actor.id` where the actor is a user and `performed_for` is usually the `target.id` target, if one exists when `target.type` is `USER`.

### Identity

While an event type broadly classifies related entities, the identity of an event is used to uniquely distinguish it from any other. The Events API encodes this information in the `eventId` as a 25 character alpha-numeric value with the `tev` prefix (e.g., `tev2FSkoWAARbKaFBBfPPXUWA1533221531000 `). On the other hand, the System Log API represents identity using a completely different scheme in the `uuid` attribute. As the field name suggests, these are UUIDs (e.g., `b5ef15a1-e78f-4125-b425-cc10f04e24f3`) that are randomly generated and unique. There is no identity value mapping between corresponding events of the two APIs. As a consequnce, you cannot infer the one from the other.

All other system IDs, such as users, applications, etc., are unchanged.

### Event Types

Event types are the primary method of organization within the Okta event system. They broadly categorize classes of events by an event type identifier. `/logs` has reduced the number of event types by a 2:1 ratio to aid consumers of event streams identify and filter more easily.

#### EventType Naming Conventions

Event types manifest themselves as dot separated strings of hierarchical object names and actions. The top-most object is the first component in the event type id. This represents the broadest categorization of the event type. The id typical pattern is:

`<object>.<object>.<action>`

Where the action is being applied to the object preceding it.

In `/events`, the top-level object naming convention (with associated event type counts) is as follows:

- `agents.*` (6)
- `api.*` (16)
- `app.*` (793)
- `core.*` (139)
- `cvd.*` (8)
- `github.*` (6)
- `gooddata.*` (4)
- `group.*` (4)
- `huddle.*` (2)
- `invalidate_app_list.*` (5)
- `iwa.*` (24)
- `mim.*` (13)
- `moveit_dmz.*` (1)
- `network_zone.*` (1)
- `omm.*` (9)
- `org.*` (1)
- `platform.*` (14)
- `plugin.*` (2)
- `policy.*` (14)
- `roambi.*` (5)
- `security.*` (6)
- `zone.*` (7)

In `/logs`, the top-level object naming convention is as follows:

- `app.*` (200)
- `application.*` (79)
- `callback.*` (4)
- `core.*` (3)
- `directory.*` (5)
- `group.*` (12)
- `master_application.*` (1)
- `mim.*` (18)
- `network_zone.*` (1)
- `oauth2.*` (11)
- `omm.*` (9)
- `org.*` (1)
- `pki.*` (3)
- `plugin.*` (2)
- `policy.*` (15)
- `scheduled_action.*` (4)
- `security.*` (6)
- `self_service.*` (2)
- `system.*` (92)
- `task.*` (5)
- `tokens.*` (2)
- `user.*` (57)
- `zone.*` (7)

#### Outcome Agnostic Event Types

To the extent possible, event types have removed the logical outcome of the occurrence from the event type id. For example, the `user.session.start` event type replaces the following `/events` equivalents:

-  `core.user_auth.login_success`
-  `core.user_auth.login_denied`
-  `core.user_auth.login_failed`
-  `core.user_auth.login_failed.policy_denied`
-  `core.user_auth.invalid_certificate`


Instead, this information has been moved to the body of the event and is encoded in the [Outcome object](https://developer.okta.com/docs/api/resources/system_log#outcome-object):

```json
{
  "outcome": {
    "result": "FAILURE",
    "reason": "INVALID_LOGIN"
  },
  ...
}
```

This general pattern results in a reduced number of event types making them easier to comprehend and navigate.

#### Vendor Agnostic Event Types

In `/events`, there are a multitude of events that include partner specific context information into the message. e.g.:

- `app.boxnet.api.error.personal_folder_sync_state`
- `app.concur.api.error.check_user_exists`
- `app.confluence.api.error.get.user`

These were primarily used to log errors and create debug context. With `/logs`, we've used a more generic event (e.g., `application.call_api`) to log a severity "Debug" type message to capture this type of information. If the event is related to an app, that will be included in the "target" and can be easily queried and accessed.

### Querying

#### Filtering

Syntactically, filtering between the two APIs is largely unchanged. For example, the `filter` parameter continues to use the [SCIM filter expressions](https://tools.ietf.org/html/rfc7644#section-3.4.2.2) for expressing which events to return by constraining attribute values by various operators. However, the allowable attribute that can be searched is now unrestricted. Any model attribute that exists can be queried unrestricted. Furthermore, the new API now supports the `co` "contains" operator where the specified value must be a substring of the attribute value.

A new "keyword filtering" feature has been introduced via the [`q` parameter](https://developer.okta.com/docs/api/resources/system_log#keyword-filter).

#### Time Range

In the Events API, there was only one query parameter that supported defining the temporal scope of the events returned: `startDate`. In the Logs API, there is now `since` (the equivalent of `startDate`) and a new [`until` parameter](https://developer.okta.com/docs/api/resources/system_log#request-parameters) which defines the end time bound of the query interval. Both of these operate against the [`published ` attribute](https://developer.okta.com/docs/api/resources/system_log#attributes). 

A subtle difference between `stateDate` and `since`/`until` is that the former was very liberal in the format that was accepted. In the System Log API, `since`/`until` values are required to conform to [Internet Date/Time Format profile of ISO 8601](https://tools.ietf.org/html/rfc3339#page-8). This is to reduce the chance for format ambiguity (e.g. timezone offsets, etc.) and accidental misuse by consumers.

#### Sorting

Limited sort ordering by `published` is now possible via the [`sortOrder` parameter](https://developer.okta.com/docs/api/resources/system_log#request-parameters). When combined with the `after` parameter, this enables queries to paginate backwards through events in a lossless fashion. Iterating forward is possible in both systems.

In order to ensure no loss of events while polling, sorting changes to become the "persisted time" of the LogEvent, rather than it's "published time". This is in contrast to the non-polling case where events are always sorted with respect to `published`. Please see [Polling Requests](https://developer.okta.com/docs/api/resources/system_log#polling-requests) for details.

### Polling

Polling is the process used to reliably ingest data from Okta into an external system. Both APIs use the `after` parameter in conjunction with `Link` response headers to safely pull the event stream. 

When you first make an API call and get a cursor-paged list of objects, the end of the list will be the point at which you do not receive another `next` link value with the response. This holds true for all but two cases:

1. [Events API](/docs/api/resources/events): The `next` link always exists, since the [Events API](/docs/api/resources/events) is like a stream of data with a cursor.

2. [System Log API](/docs/api/resources/system_log): The `next` link will always exist in polling queries in the [System Log API](/docs/api/resources/system_log). A polling query is defined as an `ASCENDING` query with an empty or absent `until` parameter. Like in the [Events API](/docs/api/resources/events), the polling query is a stream of data.

Please see [Transferring Data to a Separate System](https://developer.okta.com/docs/api/resources/system_log#transferring-data-to-a-separate-system) and the general information on [Link Header](https://developer.okta.com/docs/api/getting_started/design_principles#link-header)s for additional details.

## Event Type Mapping

The following table describes the relationship between the Events API (1,076) and System Log (541) event types. This relationship is generally many-to-one, but there are a few exceptions. Note that there are currently some event types which do not have an Events API equivalent. **Going forward the Events API will not be tracking new event types added to the System Log API. For this reason we highly recommend upgrading to the System Log API.** 

| Event API | System Log API |
| --------- | -------------- |
| `missing` | `app.ad.api.user_import.warn.skipped_user.missing_required_attribute` |
| `missing` | `app.radius.agent.listener.failed` |
| `missing` | `app.radius.agent.listener.succeeded` |
| `missing` | `application.provision.group_membership.verify_exists` |
| `missing` | `group.user_membership.rule.error` |
| `missing` | `group.user_membership.rule.evaluation` |
| `missing` | `mim.createEnrollment.ANDROID` |
| `missing` | `mim.createEnrollment.IOS` |
| `missing` | `mim.createEnrollment.OSX` |
| `missing` | `mim.createEnrollment.UNKNOWN` |
| `missing` | `mim.createEnrollment.WINDOWS` |
| `missing` | `policy.evaluate_sign_on` |
| `missing` | `scheduled_action.user_suspension.canceled` |
| `missing` | `scheduled_action.user_suspension.completed` |
| `missing` | `scheduled_action.user_suspension.scheduled` |
| `missing` | `scheduled_action.user_suspension.updated` |
| `missing` | `system.agent.ad.create` |
| `missing` | `task.lifecycle.activate` |
| `missing` | `task.lifecycle.create` |
| `missing` | `task.lifecycle.deactivate` |
| `missing` | `task.lifecycle.delete` |
| `missing` | `task.lifecycle.update` |
| `missing` | `user.authentication.verify` |
| `agents.connector_agent.agent_deactivated` | `system.agent.connector.deactivate` |
| `agents.connector_agent.agent_deleted` | `system.agent.connector.delete` |
| `agents.connector_agent.agent_disconnected` | `system.agent.connector.connect` |
| `agents.connector_agent.agent_reactivate_failed_missing_token` | `system.agent.connector.reactivate` |
| `agents.connector_agent.agent_reactivated` | `system.agent.connector.reactivate` |
| `agents.connector_agent.agent_reconnected` | `system.agent.connector.connect` |
| `api.error.logged_event.exception` | `missing` |
| `api.error.logged_event.unknown_exception` | `missing` |
| `api.oauth2.as.activated` | `oauth2.as.activated` |
| `api.oauth2.as.created` | `oauth2.as.created` |
| `api.oauth2.as.deactivated` | `oauth2.as.deactivated` |
| `api.oauth2.as.deleted` | `oauth2.as.deleted` |
| `api.oauth2.as.updated` | `oauth2.as.updated` |
| `api.oauth2.claim.created` | `oauth2.claim.created` |
| `api.oauth2.claim.deleted` | `oauth2.claim.deleted` |
| `api.oauth2.claim.updated` | `oauth2.claim.updated` |
| `api.oauth2.scope.created` | `oauth2.scope.created` |
| `api.oauth2.scope.deleted` | `oauth2.scope.deleted` |
| `api.oauth2.scope.updated` | `oauth2.scope.updated` |
| `api.token.create` | `system.api_token.create` |
| `api.token.enable` | `system.api_token.enable` |
| `api.token.revoke` | `system.api_token.revoke` |
| `app.access_request.approver.approve` | `app.access_request.approver.approve` |
| `app.access_request.approver.deny` | `app.access_request.approver.deny` |
| `app.access_request.delete` | `app.access_request.delete` |
| `app.access_request.deny` | `app.access_request.deny` |
| `app.access_request.expire` | `app.access_request.expire` |
| `app.access_request.grant` | `app.access_request.grant` |
| `app.access_request.request` | `app.access_request.request` |
| `app.ad.agent.config.error` | `system.agent.ad.update` |
| `app.ad.agent.config` | `system.agent.ad.update` |
| `app.ad.agent.dir-invoke.error` | `system.agent.ad.invoke_dir` |
| `app.ad.agent.dir-invoke` | `system.agent.ad.invoke_dir` |
| `app.ad.agent.disconnected` | `system.agent.ad.connect` |
| `app.ad.agent.fetch-logs.error` | `system.agent.ad.upload_log` |
| `app.ad.agent.fetch-logs` | `system.agent.ad.upload_log` |
| `app.ad.agent.modify-config.error` | `system.agent.ad.update` |
| `app.ad.agent.modify-config` | `system.agent.ad.update` |
| `app.ad.agent.read-config.error` | `system.agent.ad.read_config` |
| `app.ad.agent.read-config` | `system.agent.ad.read_config` |
| `app.ad.agent.read-dirsync.error` | `system.agent.ad.read_dirsync` |
| `app.ad.agent.read-dirsync` | `system.agent.ad.read_dirsync` |
| `app.ad.agent.read-forest-topology.error` | `system.agent.ad.read_toplogy` |
| `app.ad.agent.read-forest-topology` | `system.agent.ad.read_toplogy` |
| `app.ad.agent.read-ldap.error` | `system.agent.ad.read_ldap` |
| `app.ad.agent.read-ldap` | `system.agent.ad.read_ldap` |
| `app.ad.agent.read-schema.error` | `system.agent.ad.read_schema` |
| `app.ad.agent.read-schema` | `system.agent.ad.read_schema` |
| `app.ad.agent.real-time-sync.error` | `system.agent.ad.realtimesync` |
| `app.ad.agent.real-time-sync` | `system.agent.ad.realtimesync` |
| `app.ad.agent.reconnected` | `system.agent.ad.connect` |
| `app.ad.agent.scan.error` | `missing` |
| `app.ad.agent.scan` | `missing` |
| `app.ad.agent.start` | `system.agent.ad.start` |
| `app.ad.agent.upgrade.error` | `system.agent.ad.upgrade` |
| `app.ad.agent.upgrade` | `system.agent.ad.upgrade` |
| `app.ad.agent.user-auth-and-update.error` | `missing` |
| `app.ad.agent.user-auth-and-update` | `system.agent.ad.update_user` |
| `app.ad.agent.user_auth.error` | `user.authentication.auth_via_AD_agent` |
| `app.ad.agent.user_auth` | `user.authentication.auth_via_AD_agent` |
| `app.ad.agent.write-ldap.error` | `system.agent.ad.write_ldap` |
| `app.ad.agent.write-ldap` | `system.agent.ad.write_ldap` |
| `app.ad.api.user_import.account_locked` | `app.ad.api.user_import.account_locked` |
| `app.ad.api.user_import.warn.skipped_contact.attribute_invalid_value` | `app.ad.api.user_import.warn.skipped_contact.attribute_invalid_value` |
| `app.ad.api.user_import.warn.skipped_ou.missing_required_attribute` | `system.agent.ad.import_ou` |
| `app.ad.api.user_import.warn.skipped_user.attribute_invalid_value` | `app.ad.api.user_import.warn.skipped_user.attribute_invalid_value` |
| `app.ad.api.user_import.warn.skipped_user.attribute_too_long` | `system.agent.ad.import_user` |
| `app.ad.api.user_import.warn.skipped_user.internal_object.unknown_user` | `system.agent.ad.import_user` |
| `app.ad.api.user_import.warn.skipped_user.internal_object` | `system.agent.ad.import_user` |
| `app.ad.api.user_import.warn.skipped_user.invalid_user_account_control.unknown_user` | `system.agent.ad.import_user` |
| `app.ad.api.user_import.warn.skipped_user.invalid_user_account_control_computed.unknown_user` | `system.agent.ad.import_user` |
| `app.ad.api.user_import.warn.skipped_user.invalid_user_account_control_computed` | `system.agent.ad.import_user` |
| `app.ad.api.user_import.warn.skipped_user.invalid_user_account_control` | `system.agent.ad.import_user` |
| `app.ad.api.user_import.warn.skipped_user.missing_required_attribute.unknown_user` | `system.agent.ad.import_user` |
| `app.ad.api.user_import.warn.skipped_user.missing_required_attribute` | `system.agent.ad.import_user` |
| `app.ad.config.agent.agent_created` | `missing` |
| `app.ad.config.agent.agent_deactivated` | `system.agent.ad.deactivate` |
| `app.ad.config.agent.agent_deleted` | `system.agent.ad.delete` |
| `app.ad.config.agent.agent_reactivate_failed_missing_token` | `system.agent.ad.reactivate` |
| `app.ad.config.agent.agent_reactivated` | `system.agent.ad.reactivate` |
| `app.ad.login.bad_password` | `user.authentication.auth_via_AD_agent` |
| `app.ad.login.expired_password` | `user.authentication.auth_via_AD_agent` |
| `app.ad.login.locked_account` | `user.authentication.auth_via_AD_agent` |
| `app.ad.login.success` | `user.authentication.auth_via_AD_agent` |
| `app.ad.login.unknown_failure` | `user.authentication.auth_via_AD_agent` |
| `app.ad.outbound.delauth.no_connected_agent` | `user.authentication.auth_via_AD_agent` |
| `app.ad.outbound.delauth.timeout` | `user.authentication.auth_via_AD_agent` |
| `app.ad.password.reset.failure` | `system.agent.ad.reset_user_password` |
| `app.ad.password.reset.success` | `system.agent.ad.reset_user_password` |
| `app.ad.password.reset.unlock-failed` | `system.agent.ad.reset_user_password` |
| `app.ad.user.account.unlock.failure` | `system.agent.ad.unlock_user_account` |
| `app.ad.user.account.unlock.success` | `system.agent.ad.unlock_user_account` |
| `app.admin.sso.bad_response` | `user.session.access_admin_app` |
| `app.admin.sso.login.success` | `user.session.access_admin_app` |
| `app.admin.sso.no_response` | `user.session.access_admin_app` |
| `app.amazon_aws.api.error.get.roles` | `application.provision.integration.call_api` |
| `app.api.error.activate_user` | `application.provision.user.activate` |
| `app.api.error.add_group_membership` | `application.provision.group_membership.add` |
| `app.api.error.api.validation` | `application.configuration.update` |
| `app.api.error.auth` | `application.integration.authentication_failure` |
| `app.api.error.check_group_exists` | `application.provision.group.verify_exists` |
| `app.api.error.check_user_exists` | `application.provision.user.verify_exists` |
| `app.api.error.create.group` | `application.provision.group.add` |
| `app.api.error.create_pending_user` | `application.provision.user.push` |
| `app.api.error.create_user` | `application.provision.user.push` |
| `app.api.error.deactivate_user` | `application.provision.user.deactivate` |
| `app.api.error.delete_group` | `application.provision.group.remove` |
| `app.api.error.download_app_schema` | `application.configuration.import_schema` |
| `app.api.error.download_custom_objects` | `application.configuration.update` |
| `app.api.error.download_groups` | `application.provision.group.import` |
| `app.api.error.download_memberships` | `application.provision.group_membership.import` |
| `app.api.error.download_schema_enum_values` | `application.configuration.update` |
| `app.api.error.download_users` | `application.provision.user.import` |
| `app.api.error.empty_password` | `application.provision.user.password` |
| `app.api.error.generic` | `application.integration.general_failure` |
| `app.api.error.get_group_by_id` | `application.provision.group.import` |
| `app.api.error.group.more_than_one_with_same_id` | `application.provision.group.verify_exists` |
| `app.api.error.group.not_found` | `application.provision.group.verify_exists` |
| `app.api.error.group_name_long_length` | `application.provision.group.update` |
| `app.api.error.import_user_by_id` | `application.provision.user.import_profile` |
| `app.api.error.import_user_profile` | `application.provision.user.import_profile` |
| `app.api.error.manager.not_found_for_user` | `application.provision.user.push_profile` |
| `app.api.error.oauth.get.token` | `application.integration.authentication_failure` |
| `app.api.error.oauth.refresh.token` | `application.integration.authentication_failure` |
| `app.api.error.push_password_update` | `application.provision.user.password` |
| `app.api.error.push_profile_update` | `application.provision.user.push_profile` |
| `app.api.error.rate.limit.exceeded` | `application.integration.rate_limit_exceeded` |
| `app.api.error.reactivate_user` | `application.provision.user.reactivate` |
| `app.api.error.remove_group_membership` | `application.provision.group_membership.remove` |
| `app.api.error.update.group` | `application.provision.group.update` |
| `app.api.error.update_group_membership` | `application.provision.group_membership.update` |
| `app.api.error.upsert_group_duplicate` | `application.provision.group.add` |
| `app.api.error.upsert_group` | `application.provision.group.add` |
| `app.api.error.user.more_than_one_with_same_id` | `application.provision.user.verify_exists` |
| `app.api.error.user.not_found_or_deleted` | `application.provision.user.push` |
| `app.api.error.user.not_found` | `application.provision.user.push` |
| `app.app_editor.app.create` | `application.lifecycle.create` |
| `app.app_editor.app.update` | `application.lifecycle.update` |
| `app.app_instance.config-error` | `application.configuration.detect_error` |
| `app.app_instance.csr.generate` | `app.app_instance.csr.generate` |
| `app.app_instance.csr.publish` | `app.app_instance.csr.publish` |
| `app.app_instance.csr.revoke` | `app.app_instance.csr.revoke` |
| `app.app_instance.logo_reset` | `application.configuration.reset_logo` |
| `app.app_instance.logo_update` | `application.configuration.update_logo` |
| `app.app_instance.outbound_delauth_disabled` | `application.configuration.disable_delauth_outbound` |
| `app.app_instance.outbound_delauth_enabled` | `application.configuration.enable_delauth_outbound` |
| `app.app_instance.sign_on_policy.access_denied` | `application.policy.sign_on.deny_access` |
| `app.app_instance.sign_on_policy.change` | `application.policy.sign_on.update` |
| `app.app_instance.sign_on_policy.delete_rule` | `application.policy.sign_on.rule.delete` |
| `app.app_instance.sign_on_policy.new_rule` | `application.policy.sign_on.rule.create` |
| `app.audit_report.download.local.active` | `app.audit_report.download.local.active` |
| `app.audit_report.download.local.deprov` | `app.audit_report.download.local.deprov` |
| `app.audit_report.download.rogue.report` | `app.audit_report.download.rogue.report` |
| `app.audit_report.download` | `app.audit_report.download` |
| `app.auth.delegated.outbound` | `missing` |
| `app.auth.slo.saml.invalid_issuer` | `user.authentication.slo` |
| `app.auth.slo.saml.invalid_nameid` | `user.authentication.slo` |
| `app.auth.slo.saml.invalid_signature` | `user.authentication.slo` |
| `app.auth.slo.saml.malformed_request.invalid_type` | `user.authentication.slo` |
| `app.auth.slo.saml.malformed_request` | `user.authentication.slo` |
| `app.auth.slo.with_reason` | `user.authentication.slo` |
| `app.auth.slo` | `user.authentication.slo` |
| `app.auth.sso` | `user.authentication.sso` |
| `app.auth_error.INVALID_CREDENTIALS` | `application.integration.authentication_failure` |
| `app.bigmachines.api.error.activate` | `application.provision.user.reactivate` |
| `app.bigmachines.api.error.check.user.exists` | `application.provision.user.verify_exists` |
| `app.bigmachines.api.error.connection` | `application.integration.authentication_failure` |
| `app.bigmachines.api.error.create` | `application.provision.user.push` |
| `app.bigmachines.api.error.deactivate` | `application.provision.user.deactivate` |
| `app.bigmachines.api.error.import` | `application.provision.user.import_profile` |
| `app.bigmachines.api.error.login` | `application.integration.authentication_failure` |
| `app.bigmachines.api.error.logout` | `application.integration.authentication_failure` |
| `app.bigmachines.api.error.profile.update` | `application.provision.user.push_profile` |
| `app.bloomfire.api.error.api.validation` | `application.integration.authentication_failure` |
| `app.bloomfire.api.error.check_user_exists` | `application.provision.user.verify_exists` |
| `app.bloomfire.api.error.create_user` | `application.provision.user.push` |
| `app.bloomfire.api.error.download_users` | `application.provision.user.import` |
| `app.bloomfire.api.error.generic` | `application.integration.general_failure` |
| `app.bloomfire.sso.error.api_key_empty` | `application.integration.authentication_failure` |
| `app.bloomfire.sso.error.user_not_extracted` | `application.integration.authentication_failure` |
| `app.boxnet.api.error.add.email.alias` | `application.provision.user.push_profile` |
| `app.boxnet.api.error.assign_folder_permissions` | `application.provision.user.push` |
| `app.boxnet.api.error.check_group_exists` | `application.provision.group.verify_exists` |
| `app.boxnet.api.error.check_user_exists` | `application.provision.user.verify_exists` |
| `app.boxnet.api.error.create.group` | `application.provision.group.add` |
| `app.boxnet.api.error.create_new_user` | `application.provision.user.push` |
| `app.boxnet.api.error.create_personal_folder.conflict` | `application.provision.user.push` |
| `app.boxnet.api.error.create_personal_folder` | `application.provision.user.push` |
| `app.boxnet.api.error.deactivate_user` | `application.provision.user.deactivate` |
| `app.boxnet.api.error.delete.group` | `application.provision.group.remove` |
| `app.boxnet.api.error.download.group_users` | `application.provision.group_membership.import` |
| `app.boxnet.api.error.download.groups` | `application.provision.group.import` |
| `app.boxnet.api.error.download.users` | `application.provision.user.import` |
| `app.boxnet.api.error.import.user.profile` | `application.provision.user.import_profile` |
| `app.boxnet.api.error.invalid_user_login` | `application.provision.user.push` |
| `app.boxnet.api.error.personal_folder_name` | `application.provision.user.push` |
| `app.boxnet.api.error.personal_folder_sync_state` | `application.provision.user.push` |
| `app.boxnet.api.error.push.groups_set` | `application.provision.group_membership.add` |
| `app.boxnet.api.error.push.profile.update` | `application.provision.user.push_profile` |
| `app.boxnet.api.error.push.remove_from_groups` | `application.provision.group_membership.remove` |
| `app.boxnet.api.error.rate_limit_exceeded` | `application.integration.rate_limit_exceeded` |
| `app.boxnet.api.error.reactivate_user` | `application.provision.user.reactivate` |
| `app.boxnet.api.error.transfer.files` | `application.integration.transfer_files` |
| `app.boxnet.api.error.update.group` | `application.provision.group.update` |
| `app.boxnet.api.error.user.push.conflict_in_group` | `application.provision.user.push_profile` |
| `app.boxnet.api.error.validate_parent_folder` | `application.configuration.update` |
| `app.clarizen.api.error.entity.not_found` | `application.provision.user.deactivate` |
| `app.clarizen.api.error.rate_limit.exceeded` | `application.integration.rate_limit_exceeded` |
| `app.clarizen.api.error.update_group` | `application.provision.group.update` |
| `app.confluence.api.error.add.user.to.group` | `application.provision.group_membership.add` |
| `app.confluence.api.error.check.group.exists` | `application.provision.group.verify_exists` |
| `app.confluence.api.error.check.user.exists` | `application.provision.user.verify_exists` |
| `app.confluence.api.error.convert.app.user.to.remote.user` | `application.provision.user.push_profile` |
| `app.confluence.api.error.convert.remote.user.to.app.user` | `application.provision.user.import_profile` |
| `app.confluence.api.error.create.new.group` | `application.provision.group.add` |
| `app.confluence.api.error.create.new.user` | `application.provision.user.push` |
| `app.confluence.api.error.deactivate.user` | `application.provision.user.deactivate` |
| `app.confluence.api.error.download.users` | `application.provision.user.import` |
| `app.confluence.api.error.get.user.groups` | `application.provision.user.import_profile` |
| `app.confluence.api.error.get.user` | `application.provision.user.import_profile` |
| `app.confluence.api.error.import.user.profile` | `application.provision.user.import_profile` |
| `app.confluence.api.error.login` | `application.integration.authentication_failure` |
| `app.confluence.api.error.logout` | `application.integration.authentication_failure` |
| `app.confluence.api.error.parse.groups` | `application.provision.group.import` |
| `app.confluence.api.error.push.password.update` | `application.provision.user.password` |
| `app.confluence.api.error.push.profile.update` | `application.provision.user.push_profile` |
| `app.confluence.api.error.reactivate.user` | `application.provision.user.reactivate` |
| `app.confluence.api.error.remove.group` | `application.provision.group.remove` |
| `app.confluence.api.error.remove.user.to.group` | `application.provision.group_membership.remove` |
| `app.confluence.api.error.remove.user` | `application.provision.user.deactivate` |
| `app.cornerstone.api.error.api.check_user_exists` | `application.provision.user.verify_exists` |
| `app.cornerstone.api.error.api.create_user` | `application.provision.user.push` |
| `app.cornerstone.api.error.api.deactivate_user` | `application.provision.user.deactivate` |
| `app.cornerstone.api.error.api.import_profile` | `application.provision.user.import_profile` |
| `app.cornerstone.api.error.api.password_push` | `application.provision.user.password` |
| `app.cornerstone.api.error.api.push_profile` | `application.provision.user.push_profile` |
| `app.cornerstone.api.error.api.reactivate_user` | `application.provision.user.reactivate` |
| `app.cornerstone.api.error.api.validation` | `application.integration.authentication_failure` |
| `app.cornerstone.api.error.init` | `application.integration.authentication_failure` |
| `app.coupa.api.connection.error` | `application.integration.authentication_failure` |
| `app.coupa.api.error` | `application.integration.general_failure` |
| `app.crashplanpro.api.ambiguous_search_results_by_user` | `application.provision.user.verify_exists` |
| `app.crashplanpro.api.auth.invalid_login_url` | `application.integration.authentication_failure` |
| `app.crashplanpro.api.invalid_set_of_roles` | `application.integration.authentication_failure` |
| `app.crashplanpro.api.rest.unexpected_response_status` | `application.integration.general_failure` |
| `app.crashplanpro.api.user_has_invalid_fields` | `application.provision.user.push_profile` |
| `app.crashplanpro.api.user_not_found` | `application.provision.user.import_profile` |
| `app.csv.import_user.skipped_user.unknown_user` | `system.csv.import_user` |
| `app.csv.import_user.skipped_user` | `system.csv.import_user` |
| `app.docusign.api.error.import.inactive.user` | `application.provision.user.import_profile` |
| `app.docusign.api.error.import.permission.profile` | `application.provision.user.import_profile` |
| `app.docusign.api.error.no.accounts` | `application.integration.authentication_failure` |
| `app.docusign.api.error.not.account.member` | `application.integration.authentication_failure` |
| `app.docusign.api.error.update.inactive.user` | `application.provision.user.push_profile` |
| `app.docusign.api.error.update.permission.profile` | `application.provision.user.push_profile` |
| `app.dropbox.api.error.check.user` | `application.provision.user.verify_exists` |
| `app.dropbox.api.error.create.user` | `application.provision.user.push` |
| `app.dropbox.api.error.deactivation` | `application.provision.user.deactivate` |
| `app.dropbox.api.error.download.users` | `application.provision.user.import` |
| `app.dropbox.api.error.import.profile` | `application.provision.user.import_profile` |
| `app.dropbox.api.error.push.password.update` | `application.provision.user.password` |
| `app.dropbox.api.error.push.profile` | `application.provision.user.push_profile` |
| `app.dropbox.api.error.query` | `application.provision.user.import_profile` |
| `app.dropbox.api.error.rateLimit.exceeded` | `application.integration.rate_limit_exceeded` |
| `app.dropbox.api.error.set.user.permissions` | `application.provision.user.push_profile` |
| `app.dropbox.api.error.validation` | `application.integration.authentication_failure` |
| `app.echosign.api.error.connection` | `application.integration.authentication_failure` |
| `app.echosign.api.error.create` | `application.provision.user.push` |
| `app.echosign.api.error.download.users` | `application.provision.user.import` |
| `app.echosign.api.error.import.profile` | `application.provision.user.import_profile` |
| `app.echosign.api.error.search.by.id` | `application.provision.user.verify_exists` |
| `app.echosign.api.error.search.by.login` | `application.provision.user.verify_exists` |
| `app.egnyte.auth.type.validation.failure` | `application.integration.authentication_failure` |
| `app.egnyte.rate.limiting.exceeded` | `application.integration.rate_limit_exceeded` |
| `app.egnyte.username.validation.failure` | `application.integration.authentication_failure` |
| `app.eqanalyzer.url.encoding` | `application.integration.general_failure` |
| `app.evernote_business.api.error.create.user.limit.reached` | `application.provision.user.push` |
| `app.evernote_business.api.error.create.user` | `application.provision.user.push` |
| `app.evernote_business.api.error.deactivation` | `application.provision.user.deactivate` |
| `app.evernote_business.api.error.validation` | `application.integration.authentication_failure` |
| `app.exacttarget.api.error.check_user_exists` | `application.provision.user.verify_exists` |
| `app.exacttarget.api.error.create_user` | `application.provision.user.push` |
| `app.exacttarget.api.error.deactivate_user` | `application.provision.user.deactivate` |
| `app.exacttarget.api.error.download_users` | `application.provision.user.import` |
| `app.exacttarget.api.error.import_user_profile` | `application.provision.user.import_profile` |
| `app.exacttarget.api.error.init` | `application.integration.general_failure` |
| `app.exacttarget.api.error.push_password_update` | `application.provision.user.password` |
| `app.exacttarget.api.error.push_profile_update` | `application.provision.user.push_profile` |
| `app.exacttarget.api.error.reactivate_user` | `application.provision.user.reactivate` |
| `app.generic.config.app_activated` | `application.lifecycle.activate` |
| `app.generic.config.app_deactivated` | `application.lifecycle.deactivate` |
| `app.generic.config.app_deleted` | `application.lifecycle.delete` |
| `app.generic.config.app_password_update` | `application.user_membership.change_password` |
| `app.generic.config.app_updated` | `application.lifecycle.update` |
| `app.generic.config.app_user_property_update` | `application.user_membership.update` |
| `app.generic.config.app_username_update` | `application.user_membership.change_username` |
| `app.generic.config.fed_broker_mode_disabled` | `application.configuration.disable_fed_broker_mode` |
| `app.generic.config.fed_broker_mode_enabled` | `application.configuration.enable_fed_broker_mode` |
| `app.generic.import.batch.complete` | `system.import.complete_batch` |
| `app.generic.import.complete` | `system.import.complete` |
| `app.generic.import.details.add_custom_object` | `system.import.custom_object.create` |
| `app.generic.import.details.add_group` | `system.import.group.create` |
| `app.generic.import.details.add_user` | `system.import.user.create` |
| `app.generic.import.details.delete_custom_object` | `system.import.custom_object.update` |
| `app.generic.import.details.delete_group` | `system.import.group.delete` |
| `app.generic.import.details.delete_user` | `system.import.user.delete` |
| `app.generic.import.details.suspend_user` | `system.import.user.suspend` |
| `app.generic.import.details.unsuspend_user` | `system.import.user.unsuspend` |
| `app.generic.import.details.update_custom_object` | `system.import.custom_object.delete` |
| `app.generic.import.details.update_group` | `system.import.group.update` |
| `app.generic.import.details.update_user` | `system.import.user.update` |
| `app.generic.import.fail.roadblock.reschedule_and_resume` | `system.import.roadblock.reschedule_and_resume` |
| `app.generic.import.fail.roadblock.resume` | `system.import.roadblock.resume` |
| `app.generic.import.fail.roadblock` | `system.import.roadblock` |
| `app.generic.import.import_groups` | `system.import.group.start` |
| `app.generic.import.import_user` | `system.import.user.start` |
| `app.generic.import.provisioning_data` | `system.import.import_provisioning_info` |
| `app.generic.import.started` | `system.import.start` |
| `app.generic.import.summary.custom_object` | `system.import.custom_object.complete` |
| `app.generic.import.summary.group_membership` | `system.import.group_membership.complete` |
| `app.generic.import.summary.group` | `system.import.group.complete` |
| `app.generic.import.summary.user` | `system.import.user.complete` |
| `app.generic.import.user_match.unsuspend_after_confirm` | `system.import.user.unsuspend_after_confirm` |
| `app.generic.provision.approve_user_for_app` | `application.user_membership.approve` |
| `app.generic.provision.assign_user_to_app` | `application.user_membership.add` |
| `app.generic.provision.assign_user_to_app` | `master_application.user_membership.add` |
| `app.generic.provision.deactivate_user_from_app` | `application.user_membership.remove` |
| `app.generic.provision.deprovision_user_from_app` | `application.user_membership.deprovision` |
| `app.generic.provision.provision_user_for_app` | `application.user_membership.provision` |
| `app.generic.provision.revoke_user_from_app` | `application.user_membership.revoke` |
| `app.generic.reversibility.credentials.recovery` | `application.user_membership.restore_password` |
| `app.generic.reversibility.individual.app.recovery` | `application.user_membership.restore` |
| `app.generic.reversibility.personal.app.recovery` | `application.user_membership.restore` |
| `app.generic.show.password` | `application.user_membership.show_password` |
| `app.generic.unauth_app_access_attempt` | `app.generic.unauth_app_access_attempt` |
| `app.generic.user_management.error.add_manager_to_user` | `application.provision.user.push_profile` |
| `app.google.api.error.InsufficientPermission` | `application.provision.user.push_profile` |
| `app.google.license_management.error.assign_license` | `application.provision.user.push_profile` |
| `app.google.license_management.error.remove_license` | `application.provision.user.push_profile` |
| `app.google.sso.failure.domain_not_found` | `application.integration.general_failure` |
| `app.google.sso.failure.invalid_continue_url` | `application.integration.general_failure` |
| `app.google.sso.failure.invalid_domain` | `application.integration.general_failure` |
| `app.google.sso.failure.relay_state_not_found` | `application.integration.general_failure` |
| `app.google.user_management.error.add_member_to_group` | `application.provision.group_membership.add` |
| `app.google.user_management.error.check_group_exists.invalid_domain` | `application.provision.group.verify_exists` |
| `app.google.user_management.error.check_group_exists` | `application.provision.group.verify_exists` |
| `app.google.user_management.error.check_user_exists.invalid_domain` | `application.provision.user.verify_exists` |
| `app.google.user_management.error.check_user_exists` | `application.provision.user.verify_exists` |
| `app.google.user_management.error.create_group_duplicate` | `application.provision.group.add` |
| `app.google.user_management.error.create_group` | `application.provision.group.add` |
| `app.google.user_management.error.create_new_user` | `application.provision.user.push` |
| `app.google.user_management.error.deactivate_user` | `application.provision.user.deactivate` |
| `app.google.user_management.error.delete_group` | `application.provision.group.remove` |
| `app.google.user_management.error.download_app_schema` | `application.configuration.import_schema` |
| `app.google.user_management.error.download_custom_objects` | `application.configuration.update` |
| `app.google.user_management.error.download_group_members` | `application.provision.group_membership.import` |
| `app.google.user_management.error.download_groups` | `application.provision.group.import` |
| `app.google.user_management.error.download_org_units` | `application.provision.user.import` |
| `app.google.user_management.error.download_users` | `application.provision.user.import` |
| `app.google.user_management.error.import_user_profile` | `application.provision.user.import_profile` |
| `app.google.user_management.error.invalid_manager` | `application.provision.user.push` |
| `app.google.user_management.error.invalid_orgunit_id` | `application.provision.user.push` |
| `app.google.user_management.error.push_password_update` | `application.provision.user.password` |
| `app.google.user_management.error.push_profile_update` | `application.provision.user.push_profile` |
| `app.google.user_management.error.rateLimit` | `application.integration.rate_limit_exceeded` |
| `app.google.user_management.error.reactivate_user` | `application.provision.user.reactivate` |
| `app.google.user_management.error.reconcile_email_aliases` | `application.provision.user.push_profile` |
| `app.google.user_management.error.remove_member_from_group` | `application.provision.group_membership.remove` |
| `app.google.user_management.error.update_group` | `application.provision.group.update` |
| `app.gotomeeting.user_management.config.failure.api_login_failure` | `application.integration.authentication_failure` |
| `app.gotomeeting.user_management.config.failure.api_not_available` | `application.integration.general_failure` |
| `app.gotomeeting.user_management.config.failure.api_url_is_malformed` | `application.integration.general_failure` |
| `app.gotomeeting.user_management.config.failure.user_import` | `application.provision.user.import` |
| `app.gotomeeting.user_management.config.failure.user_provisioning` | `application.provision.user.push` |
| `app.gotomeeting_rest.user_management.config.failure.api_auth_failed` | `application.integration.authentication_failure` |
| `app.gotomeeting_rest.user_management.config.failure.api_not_available` | `application.integration.general_failure` |
| `app.gotomeeting_rest.user_management.config.failure.user_import` | `application.provision.user.import` |
| `app.gotomeeting_rest.user_management.config.failure.user_provisioning` | `application.provision.user.push` |
| `app.hipchat.api.error.check.user` | `application.provision.user.verify_exists` |
| `app.hipchat.api.error.create.user` | `application.provision.user.push` |
| `app.hipchat.api.error.deactivation` | `application.provision.user.deactivate` |
| `app.hipchat.api.error.download.users` | `application.provision.user.import` |
| `app.hipchat.api.error.import.profile` | `application.provision.user.import_profile` |
| `app.hipchat.api.error.push.password` | `application.provision.user.password` |
| `app.hipchat.api.error.push.profile` | `application.provision.user.push_profile` |
| `app.hipchat.api.error.query` | `application.integration.api_query` |
| `app.hipchat.api.error.reactivation` | `application.provision.user.reactivate` |
| `app.hipchat.api.error.validation` | `application.configuration.update` |
| `app.hipchat.rateLimit.exceeded` | `application.integration.rate_limit_exceeded` |
| `app.inbound_del_auth.failure.account_not_found` | `user.authentication.auth_via_inbound_delauth` |
| `app.inbound_del_auth.failure.instance_not_found` | `user.authentication.auth_via_inbound_delauth` |
| `app.inbound_del_auth.failure.invalid_login_credentials` | `user.authentication.auth_via_inbound_delauth` |
| `app.inbound_del_auth.failure.invalid_request.could_not_parse_credentials` | `user.authentication.auth_via_inbound_delauth` |
| `app.inbound_del_auth.failure.not_supported` | `user.authentication.auth_via_inbound_delauth` |
| `app.inbound_del_auth.login_success` | `app.inbound_del_auth.login_success` |
| `app.jira.api.error.binding` | `application.configuration.update` |
| `app.jira.api.error.check.get.user` | `application.provision.user.import_profile` |
| `app.jira.api.error.check.user.exists` | `application.provision.user.verify_exists` |
| `app.jira.api.error.convert.app.user.to.remote.user` | `application.provision.user.push_profile` |
| `app.jira.api.error.convert.remote.user.to.app.user` | `application.provision.user.import_profile` |
| `app.jira.api.error.create.new.user` | `application.provision.user.push` |
| `app.jira.api.error.delete.group` | `application.provision.group.remove` |
| `app.jira.api.error.download.server.set.values` | `application.configuration.import_schema` |
| `app.jira.api.error.download.users` | `application.provision.user.import` |
| `app.jira.api.error.import.user.profile` | `application.provision.user.import_profile` |
| `app.jira.api.error.login` | `application.configuration.update` |
| `app.jira.api.error.logout` | `application.configuration.update` |
| `app.jira.api.error.push.password.update` | `application.provision.user.password` |
| `app.jira.api.error.push.profile.update` | `application.provision.user.push_profile` |
| `app.jira.api.error.update.group.membership` | `application.provision.group_membership.update` |
| `app.jira.api.error.upsert.group` | `application.provision.group.add` |
| `app.kerberos_rich_client.account_not_found` | `app.kerberos_rich_client.account_not_found` |
| `app.kerberos_rich_client.instance_not_found` | `app.kerberos_rich_client.instance_not_found` |
| `app.kerberos_rich_client.invalid_org` | `app.kerberos_rich_client.invalid_org` |
| `app.kerberos_rich_client.multiple_accounts_found` | `app.kerberos_rich_client.multiple_accounts_found` |
| `app.kerberos_rich_client.user_authentication_successful` | `app.kerberos_rich_client.user_authentication_successful` |
| `app.keys.clone_legacy` | `app.keys.clone` |
| `app.keys.generate_legacy` | `app.keys.generate` |
| `app.keys.rotate_legacy` | `app.keys.rotate` |
| `app.ldap.agent.disconnected` | `system.agent.ldap.disconnect` |
| `app.ldap.agent.password_change.timeout` | `system.agent.ldap.change_user_password` |
| `app.ldap.agent.password_change` | `system.agent.ldap.change_user_password` |
| `app.ldap.agent.password_reset.error` | `system.agent.ldap.reset_user_password` |
| `app.ldap.agent.password_reset.timeout` | `system.agent.ldap.reset_user_password` |
| `app.ldap.agent.password_reset` | `system.agent.ldap.reset_user_password` |
| `app.ldap.agent.password_update.error` | `system.agent.ldap.update_user_password` |
| `app.ldap.agent.password_update` | `system.agent.ldap.update_user_password` |
| `app.ldap.agent.reconnected` | `system.agent.ldap.reconnect` |
| `app.ldap.jit.ambiguous` | `system.agent.ldap.create_user_JIT` |
| `app.ldap.login.bad_password` | `user.authentication.auth_via_LDAP_agent` |
| `app.ldap.login.disabled_account` | `user.authentication.auth_via_LDAP_agent` |
| `app.ldap.login.expired_password` | `user.authentication.auth_via_LDAP_agent` |
| `app.ldap.login.locked_account` | `user.authentication.auth_via_LDAP_agent` |
| `app.ldap.login.success` | `user.authentication.auth_via_LDAP_agent` |
| `app.ldap.login.unknown_failure` | `user.authentication.auth_via_LDAP_agent` |
| `app.ldap.password.change.failed` | `app.ldap.password.change.failed` |
| `app.ldap.password.reset.constraint.error` | `system.agent.ldap.reset_user_password` |
| `app.ldap.password.reset.failed` | `system.agent.ldap.reset_user_password` |
| `app.ldap.password.reset.invalid.old.password` | `system.agent.ldap.reset_user_password` |
| `app.ldap.password.reset.succeeded` | `system.agent.ldap.reset_user_password` |
| `app.ldap.password.reset.systemic.error` | `system.agent.ldap.reset_user_password` |
| `app.ldap.password_reset.attribs_not_set` | `system.agent.ldap.reset_user_password` |
| `app.ldap.password_reset.new_confirm_password_empty` | `system.agent.ldap.reset_user_password` |
| `app.ldap.password_reset.new_password_empty` | `system.agent.ldap.reset_user_password` |
| `app.ldap.password_reset.old_new_passwords_equal` | `system.agent.ldap.reset_user_password` |
| `app.ldap.password_reset.old_password_empty` | `system.agent.ldap.reset_user_password` |
| `app.ldap.password_reset.passwords_do_not_match` | `system.agent.ldap.reset_user_password` |
| `app.ldap.password_reset.restriction.error` | `system.agent.ldap.reset_user_password` |
| `app.ldap.unlock.account.failed` | `system.agent.ldap.unlock_user_account` |
| `app.ldap.unlock.account.succeeded` | `system.agent.ldap.unlock_user_account` |
| `app.litmos.import.rate_limit_exceeded` | `application.integration.rate_limit_exceeded` |
| `app.netsuite.api.error.auth` | `application.integration.authentication_failure` |
| `app.netsuite.api.error.check_user_exists` | `application.provision.user.verify_exists` |
| `app.netsuite.api.error.create_user` | `application.provision.user.push` |
| `app.netsuite.api.error.deactivate_user` | `application.provision.user.deactivate` |
| `app.netsuite.api.error.download_custom_objects` | `application.configuration.update` |
| `app.netsuite.api.error.download_users` | `application.provision.user.import` |
| `app.netsuite.api.error.import_user_profile` | `application.provision.user.import_profile` |
| `app.netsuite.api.error.push_password_update` | `application.provision.user.password` |
| `app.netsuite.api.error.push_profile_update` | `application.provision.user.push_profile` |
| `app.netsuite.api.error.reactivate_user` | `application.provision.user.reactivate` |
| `app.oauth2.as.authorize.code_success` | `app.oauth2.as.authorize.code` |
| `app.oauth2.as.authorize.implicit.access_token_success` | `app.oauth2.as.authorize.implicit.access_token` |
| `app.oauth2.as.authorize.implicit.id_token_success` | `app.oauth2.as.authorize.implicit.id_token` |
| `app.oauth2.as.authorize.scope_denied_failure` | `app.oauth2.as.authorize.scope_denied` |
| `app.oauth2.as.authorize_failure` | `app.oauth2.as.authorize` |
| `app.oauth2.as.consent.grant_failure` | `app.oauth2.as.consent.grant` |
| `app.oauth2.as.consent.grant_success` | `app.oauth2.as.consent.grant` |
| `app.oauth2.as.consent.revoke.implicit.as_success` | `app.oauth2.as.consent.revoke.implicit.as` |
| `app.oauth2.as.consent.revoke.implicit.client_success` | `app.oauth2.as.consent.revoke.implicit.client` |
| `app.oauth2.as.consent.revoke.implicit.scope_success` | `app.oauth2.as.consent.revoke.implicit.scope` |
| `app.oauth2.as.consent.revoke.implicit.user_success` | `app.oauth2.as.consent.revoke.implicit.user` |
| `app.oauth2.as.consent.revoke.user.client_failure` | `app.oauth2.as.consent.revoke.user.client` |
| `app.oauth2.as.consent.revoke.user.client_success` | `app.oauth2.as.consent.revoke.user.client` |
| `app.oauth2.as.consent.revoke.user_failure` | `app.oauth2.as.consent.revoke.user` |
| `app.oauth2.as.consent.revoke.user_success` | `app.oauth2.as.consent.revoke.user` |
| `app.oauth2.as.consent.revoke_failure` | `app.oauth2.as.consent.revoke` |
| `app.oauth2.as.consent.revoke_success` | `app.oauth2.as.consent.revoke` |
| `app.oauth2.as.evaluate.claim_failure` | `app.oauth2.as.evaluate.claim` |
| `app.oauth2.as.key.rollover.legacy` | `app.oauth2.as.key.rollover` |
| `app.oauth2.as.token.grant.access_token_success` | `app.oauth2.as.token.grant.access_token` |
| `app.oauth2.as.token.grant.id_token_success` | `app.oauth2.as.token.grant.id_token` |
| `app.oauth2.as.token.grant.refresh_token_success` | `app.oauth2.as.token.grant.refresh_token` |
| `app.oauth2.as.token.grant_failure` | `app.oauth2.as.token.grant` |
| `app.oauth2.as.token.revoke_failure` | `app.oauth2.as.token.revoke` |
| `app.oauth2.as.token.revoke_success` | `app.oauth2.as.token.revoke` |
| `app.oauth2.authorize.code_success` | `app.oauth2.authorize.code` |
| `app.oauth2.authorize.implicit.access_token_success` | `app.oauth2.authorize.implicit.access_token` |
| `app.oauth2.authorize.implicit.id_token_success` | `app.oauth2.authorize.implicit.id_token` |
| `app.oauth2.authorize_failure` | `app.oauth2.authorize` |
| `app.oauth2.client.lifecycle.activate` | `app.oauth2.client.lifecycle.activate` |
| `app.oauth2.client.lifecycle.create` | `app.oauth2.client.lifecycle.create` |
| `app.oauth2.client.lifecycle.deactivate` | `app.oauth2.client.lifecycle.deactivate` |
| `app.oauth2.client.lifecycle.delete` | `app.oauth2.client.lifecycle.delete` |
| `app.oauth2.client.lifecycle.update` | `app.oauth2.client.lifecycle.update` |
| `app.oauth2.invalid_client_credentials_failure` | `app.oauth2.invalid_client_credentials` |
| `app.oauth2.invalid_client_ids_failure` | `app.oauth2.invalid_client_ids` |
| `app.oauth2.key.rollover.legacy` | `app.oauth2.key.rollover` |
| `app.oauth2.signon_failure` | `app.oauth2.signon` |
| `app.oauth2.signon_success` | `app.oauth2.signon` |
| `app.oauth2.token.grant.access_token_success` | `app.oauth2.token.grant.access_token` |
| `app.oauth2.token.grant.id_token_success` | `app.oauth2.token.grant.id_token` |
| `app.oauth2.token.grant.refresh_token_success` | `app.oauth2.token.grant.refresh_token` |
| `app.oauth2.token.grant_failure` | `app.oauth2.token.grant` |
| `app.oauth2.token.revoke.implicit.as_success` | `app.oauth2.token.revoke.implicit.as` |
| `app.oauth2.token.revoke.implicit.client_success` | `app.oauth2.token.revoke.implicit.client` |
| `app.oauth2.token.revoke.implicit.user_success` | `app.oauth2.token.revoke.implicit.user` |
| `app.oauth2.token.revoke_failure` | `app.oauth2.token.revoke` |
| `app.oauth2.token.revoke_success` | `app.oauth2.token.revoke` |
| `app.office365.api.change.domain.federation.success` | `app.office365.api.change.domain.federation.success` |
| `app.office365.api.error.ad.user` | `app.office365.api.error.ad.user` |
| `app.office365.api.error.check.user.exists` | `app.office365.api.error.check.user.exists` |
| `app.office365.api.error.create.user` | `app.office365.api.error.create.user` |
| `app.office365.api.error.deactivate.user` | `app.office365.api.error.deactivate.user` |
| `app.office365.api.error.download.custom.objects` | `app.office365.api.error.download.custom.objects` |
| `app.office365.api.error.download.groups` | `app.office365.api.error.download.groups` |
| `app.office365.api.error.download.users` | `app.office365.api.error.download.users` |
| `app.office365.api.error.endpoint.unavailable` | `app.office365.api.error.endpoint.unavailable` |
| `app.office365.api.error.get.company.dirsync.failure` | `app.office365.api.error.get.company.dirsync.failure` |
| `app.office365.api.error.get.company.dirsync.status.failure` | `app.office365.api.error.get.company.dirsync.status.failure` |
| `app.office365.api.error.get.company.dirsync.status.pending` | `app.office365.api.error.get.company.dirsync.status.pending` |
| `app.office365.api.error.group.create.failure.name.in.use` | `app.office365.api.error.group.create.failure.name.in.use` |
| `app.office365.api.error.group.create.failure` | `app.office365.api.error.group.create.failure` |
| `app.office365.api.error.group.delete.failure` | `app.office365.api.error.group.delete.failure` |
| `app.office365.api.error.group.membership.update.assignment.failure` | `app.office365.api.error.group.membership.update.assignment.failure` |
| `app.office365.api.error.group.membership.update.failure` | `app.office365.api.error.group.membership.update.failure` |
| `app.office365.api.error.group.membership.update.group.not.found.failure` | `app.office365.api.error.group.membership.update.group.not.found.failure` |
| `app.office365.api.error.group.membership.update.removal.failure` | `app.office365.api.error.group.membership.update.removal.failure` |
| `app.office365.api.error.group.update.failure.not.found` | `app.office365.api.error.group.update.failure.not.found` |
| `app.office365.api.error.group.update.failure` | `app.office365.api.error.group.update.failure` |
| `app.office365.api.error.import.profile.photo` | `app.office365.api.error.import.profile.photo` |
| `app.office365.api.error.import.profile` | `app.office365.api.error.import.profile` |
| `app.office365.api.error.no.endpoints.found` | `app.office365.api.error.no.endpoints.found` |
| `app.office365.api.error.push.password` | `app.office365.api.error.push.password` |
| `app.office365.api.error.push.profile.object` | `app.office365.api.error.push.profile.object` |
| `app.office365.api.error.push.profile` | `app.office365.api.error.push.profile` |
| `app.office365.api.error.reactivate.user` | `app.office365.api.error.reactivate.user` |
| `app.office365.api.error.remove.domain.federation.failure.access.denied` | `app.office365.api.error.remove.domain.federation.failure.access.denied` |
| `app.office365.api.error.remove.domain.federation.failure.domain.not.found` | `app.office365.api.error.remove.domain.federation.failure.domain.not.found` |
| `app.office365.api.error.remove.domain.federation.failure` | `app.office365.api.error.remove.domain.federation.failure` |
| `app.office365.api.error.revoke.refresh.token` | `app.office365.api.error.revoke.refresh.token` |
| `app.office365.api.error.set.company.dirsync.failure` | `app.office365.api.error.set.company.dirsync.failure` |
| `app.office365.api.error.set.company.dirsync.status.failure` | `app.office365.api.error.set.company.dirsync.status.failure` |
| `app.office365.api.error.set.domain.federation.failure.access.denied` | `app.office365.api.error.set.domain.federation.failure.access.denied` |
| `app.office365.api.error.set.domain.federation.failure.domain.default` | `app.office365.api.error.set.domain.federation.failure.domain.default` |
| `app.office365.api.error.set.domain.federation.failure.domain.not.found` | `app.office365.api.error.set.domain.federation.failure.domain.not.found` |
| `app.office365.api.error.set.domain.federation.failure` | `app.office365.api.error.set.domain.federation.failure` |
| `app.office365.api.error.sync.contact` | `app.office365.api.error.sync.contact` |
| `app.office365.api.error.sync.finalize` | `app.office365.api.error.sync.finalize` |
| `app.office365.api.error.sync.group` | `app.office365.api.error.sync.group` |
| `app.office365.api.error.sync.not.activated` | `app.office365.api.error.sync.not.activated` |
| `app.office365.api.error.sync.set.attribute` | `app.office365.api.error.sync.set.attribute` |
| `app.office365.api.error.sync.user` | `app.office365.api.error.sync.user` |
| `app.office365.api.error.unable.to.create.graph.client` | `app.office365.api.error.unable.to.create.graph.client` |
| `app.office365.api.error.validate.admin.creds` | `app.office365.api.error.validate.admin.creds` |
| `app.office365.api.error.validate.creds.unknown.exception` | `app.office365.api.error.validate.creds.unknown.exception` |
| `app.office365.api.error.validate.creds` | `app.office365.api.error.validate.creds` |
| `app.office365.api.error.x-ms-forwarded-client-ip-header.absent` | `app.office365.api.error.x-ms-forwarded-client-ip-header.absent` |
| `app.office365.api.remove.domain.federation.success` | `app.office365.api.remove.domain.federation.success` |
| `app.office365.api.set.domain.federation.success` | `app.office365.api.set.domain.federation.success` |
| `app.office365.api.set.wsfed.configure.type.success` | `app.office365.api.set.wsfed.configure.type.success` |
| `app.office365.api.sync.complete` | `app.office365.api.sync.complete` |
| `app.office365.api.sync.heartbeat.sent` | `app.office365.api.sync.heartbeat.sent` |
| `app.office365.api.sync.job.complete.contact` | `app.office365.api.sync.job.complete.contact` |
| `app.office365.api.sync.job.complete.group` | `app.office365.api.sync.job.complete.group` |
| `app.office365.api.sync.job.complete.user` | `app.office365.api.sync.job.complete.user` |
| `app.office365.api.sync.job.complete` | `app.office365.api.sync.job.complete` |
| `app.office365.clientplatform.conversion.job.processing.app.instance` | `app.office365.clientplatform.conversion.job.processing.app.instance` |
| `app.office365.clientplatform.conversion.job.skipping.migration` | `app.office365.clientplatform.conversion.job.skipping.migration` |
| `app.office365.dirsync.skipping.conflict-object` | `app.office365.dirsync.skipping.conflict-object` |
| `app.office365.dirsync.skipping.critical-system-object` | `app.office365.dirsync.skipping.critical-system-object` |
| `app.office365.dirsync.skipping.non-security-group-invalid-mail` | `app.office365.dirsync.skipping.non-security-group-invalid-mail` |
| `app.office365.dirsync.skipping.reserved-attribute-value` | `app.office365.dirsync.skipping.reserved-attribute-value` |
| `app.office365.dirsync.skipping.systemmailbox` | `app.office365.dirsync.skipping.systemmailbox` |
| `app.office365.dirsync.skipping.without-name-and-displayname` | `app.office365.dirsync.skipping.without-name-and-displayname` |
| `app.office365.error.importing.user` | `app.office365.error.importing.user` |
| `app.office365.graph.api.error.no.mailbox.found` | `app.office365.graph.api.error.no.mailbox.found` |
| `app.office365.graph.api.error.rate-limit.exceeded` | `app.office365.graph.api.error.rate-limit.exceeded` |
| `app.office365.license.conversion.jo.begin.migration.gaas.v1.to.v0` | `app.office365.license.conversion.jo.begin.migration.gaas.v1.to.v0` |
| `app.office365.license.conversion.job.begin.downloading.custom.objects` | `app.office365.license.conversion.job.begin.downloading.custom.objects` |
| `app.office365.license.conversion.job.begin.migration.gaas.v0.to.v1` | `app.office365.license.conversion.job.begin.migration.gaas.v0.to.v1` |
| `app.office365.license.conversion.job.begin.migration.users.v0.to.v1` | `app.office365.license.conversion.job.begin.migration.users.v0.to.v1` |
| `app.office365.license.conversion.job.begin.migration.users.v1.to.v0` | `app.office365.license.conversion.job.begin.migration.users.v1.to.v0` |
| `app.office365.license.conversion.job.download.or.migration.failed` | `app.office365.license.conversion.job.download.or.migration.failed` |
| `app.office365.license.conversion.job.end.count.migration.gaas.v0.to.v1` | `app.office365.license.conversion.job.end.count.migration.gaas.v0.to.v1` |
| `app.office365.license.conversion.job.end.count.migration.gaas.v1.to.v0` | `app.office365.license.conversion.job.end.count.migration.gaas.v1.to.v0` |
| `app.office365.license.conversion.job.end.count.migration.users.v0.to.v1` | `app.office365.license.conversion.job.end.count.migration.users.v0.to.v1` |
| `app.office365.license.conversion.job.end.count.migration.users.v1.to.v0` | `app.office365.license.conversion.job.end.count.migration.users.v1.to.v0` |
| `app.office365.license.conversion.job.end.downloading.custom.objects` | `app.office365.license.conversion.job.end.downloading.custom.objects` |
| `app.office365.license.conversion.job.end.migration.gaas.v0.to.v1` | `app.office365.license.conversion.job.end.migration.gaas.v0.to.v1` |
| `app.office365.license.conversion.job.end.migration.gaas.v1.to.v0` | `app.office365.license.conversion.job.end.migration.gaas.v1.to.v0` |
| `app.office365.license.conversion.job.end.migration.users.v0.to.v1` | `app.office365.license.conversion.job.end.migration.users.v0.to.v1` |
| `app.office365.license.conversion.job.end.migration.users.v1.to.v0` | `app.office365.license.conversion.job.end.migration.users.v1.to.v0` |
| `app.office365.license.conversion.job.migration.gaas.v0.to.v1.failed` | `app.office365.license.conversion.job.migration.gaas.v0.to.v1.failed` |
| `app.office365.license.conversion.job.migration.gaas.v1.to.v0.failed` | `app.office365.license.conversion.job.migration.gaas.v1.to.v0.failed` |
| `app.office365.license.conversion.job.migration.users.v0.to.v1.failed` | `app.office365.license.conversion.job.migration.users.v0.to.v1.failed` |
| `app.office365.license.conversion.job.migration.users.v1.to.v0.failed` | `app.office365.license.conversion.job.migration.users.v1.to.v0.failed` |
| `app.office365.license.conversion.job.no.custom.objects.downloaded` | `app.office365.license.conversion.job.no.custom.objects.downloaded` |
| `app.office365.license.conversion.job.processing.app.instance` | `app.office365.license.conversion.job.processing.app.instance` |
| `app.office365.license.conversion.job.skipping.inactive.app.instance` | `app.office365.license.conversion.job.skipping.inactive.app.instance` |
| `app.office365.license.conversion.job.skipping.migration.gaa.v0.to.v1` | `app.office365.license.conversion.job.skipping.migration.gaa.v0.to.v1` |
| `app.office365.license.conversion.job.skipping.migration.gaa.v1.to.v0` | `app.office365.license.conversion.job.skipping.migration.gaa.v1.to.v0` |
| `app.office365.license.conversion.job.skipping.migration.user.v0.to.v1` | `app.office365.license.conversion.job.skipping.migration.user.v0.to.v1` |
| `app.office365.license.conversion.job.skipping.migration.user.v1.to.v0` | `app.office365.license.conversion.job.skipping.migration.user.v1.to.v0` |
| `app.office365.license.conversion.job.skipping.missing.creds.app.instance` | `app.office365.license.conversion.job.skipping.missing.creds.app.instance` |
| `app.office365.license.conversion.job.skipping.no.change.licenses.app.instance` | `app.office365.license.conversion.job.skipping.no.change.licenses.app.instance` |
| `app.office365.service.principal.cleanup.job.complete` | `app.office365.service.principal.cleanup.job.complete` |
| `app.office365.service.principal.cleanup.job.invalid.credentials` | `app.office365.service.principal.cleanup.job.invalid.credentials` |
| `app.office365.service.principal.cleanup.job.processing` | `app.office365.service.principal.cleanup.job.processing` |
| `app.office365.service.principal.cleanup.job.skipping.missing.creds` | `app.office365.service.principal.cleanup.job.skipping.missing.creds` |
| `app.office365.service.principal.cleanup.job.skipping.no.service.principal` | `app.office365.service.principal.cleanup.job.skipping.no.service.principal` |
| `app.office365.service.principal.cleanup.job.unable.to.delete.service.principal` | `app.office365.service.principal.cleanup.job.unable.to.delete.service.principal` |
| `app.office365.user.delete.success` | `app.office365.user.delete.success` |
| `app.office365.user.lifecycle.action.failed` | `app.office365.user.lifecycle.action.failed` |
| `app.office365.user.remove.licenses.success` | `app.office365.user.remove.licenses.success` |
| `app.okta_org2org.user_management.error.download_app_schema` | `application.configuration.import_schema` |
| `app.okta_org2org.user_management.error.download_user_type` | `application.configuration.import_schema` |
| `app.okta_org2org.user_management.error.parse_schema` | `application.configuration.import_schema` |
| `app.okta_org2org.user_management.error.schema.property.not.exist` | `application.configuration.import_schema` |
| `app.pagerduty.api.auth.error.invalid.admin.role` | `application.integration.authentication_failure` |
| `app.pagerduty.api.auth.error.invalid.admin.username` | `application.integration.authentication_failure` |
| `app.pagerduty.api.auth.error.invalid.api.key` | `application.integration.authentication_failure` |
| `app.pagerduty.api.deactivate.user.unexpected.status` | `application.provision.user.deactivate` |
| `app.pagerduty.api.push.profile.update.unexpected.status` | `application.provision.user.push_profile` |
| `app.postini.user_management.config.failure.api_login_failed` | `application.integration.authentication_failure` |
| `app.postini.user_management.config.failure.provisioning` | `application.provision.user.push` |
| `app.postini.user_management.failure.download_users` | `application.provision.user.import` |
| `app.radius.agent.port_inaccessible` | `app.radius.agent.port_inaccessible` |
| `app.radius.agent.port_reaccessible` | `app.radius.agent.port_reaccessible` |
| `app.realtimesync.import.details.add_user` | `app.realtimesync.import.details.add_user` |
| `app.realtimesync.import.details.delete_user` | `app.realtimesync.import.details.delete_user` |
| `app.realtimesync.import.details.suspend_user` | `missing` |
| `app.realtimesync.import.details.unsuspend_user` | `missing` |
| `app.realtimesync.import.details.update_user` | `app.realtimesync.import.details.update_user` |
| `app.realtimesync.import_failed.details.email_length` | `missing` |
| `app.registration_policy.lifecycle.create` | `application.registration_policy.lifecycle.create` |
| `app.registration_policy.lifecycle.update` | `application.registration_policy.lifecycle.update` |
| `app.rich_client.account_not_found` | `user.authentication.auth_via_richclient` |
| `app.rich_client.instance_not_found` | `user.authentication.auth_via_richclient` |
| `app.rich_client.invalid_org` | `missing` |
| `app.rich_client.login_failure` | `user.authentication.auth_via_richclient` |
| `app.rich_client.login_success` | `user.authentication.auth_via_richclient` |
| `app.rich_client.multiple_accounts_found` | `user.authentication.auth_via_richclient` |
| `app.rightscale.api.error.create.user` | `application.provision.user.push` |
| `app.rightscale.api.error.download.groups` | `application.provision.group.import` |
| `app.rightscale.api.error.download.users` | `application.provision.user.import` |
| `app.rightscale.api.error.get.users` | `application.provision.user.import_profile` |
| `app.rightscale.api.error.idp` | `application.integration.general_failure` |
| `app.rightscale.api.error.login` | `application.integration.authentication_failure` |
| `app.rightscale.api.error.push.profile` | `application.provision.user.push_profile` |
| `app.rightscale.api.error.validate` | `application.configuration.update` |
| `app.rum.config.validation.error` | `missing` |
| `app.rum.execution.security.exception` | `application.provision.integration.call_api` |
| `app.rum.execution.standard.attributes.exception` | `application.provision.integration.call_api` |
| `app.rum.failure.timeout.reschedule` | `application.provision.integration.call_api` |
| `app.rum.is.api.account.error` | `missing` |
| `app.rum.package.thrown.error` | `missing` |
| `app.rum.validation.error` | `missing` |
| `app.salesforce.user_management.failure.add_user_to_public_group` | `application.provision.group_membership.update` |
| `app.salesforce.user_management.failure.api_service_not_available` | `application.integration.authentication_failure` |
| `app.salesforce.user_management.failure.cant.push.password` | `application.provision.user.password` |
| `app.salesforce.user_management.failure.download_user_schema` | `application.configuration.import_schema` |
| `app.salesforce.user_management.failure.general_api_login_failure` | `application.integration.authentication_failure` |
| `app.salesforce.user_management.failure.invalid_api_credentials` | `application.integration.authentication_failure` |
| `app.salesforce.user_management.failure.password_expired` | `application.integration.authentication_failure` |
| `app.salesforce.user_management.failure.provisioning` | `application.provision.user.push` |
| `app.salesforce.user_management.failure.remove_user_from_public_group` | `application.provision.group_membership.update` |
| `app.salesforce.user_management.failure.user_import` | `application.provision.user.import` |
| `app.salesforce.user_management.sso.only.user.password.rejected` | `application.provision.user.password` |
| `app.samanage.api.error.incorrect.attribute` | `application.provision.user.push` |
| `app.samanage.api.error.long_group_name` | `application.provision.group.add` |
| `app.scim.is.api.account.error` | `missing` |
| `app.self_service.disabled` | `self_service.disabled` |
| `app.self_service.enabled` | `self_service.enabled` |
| `app.sendwordnow.api.error.auth` | `application.configuration.update` |
| `app.sendwordnow.api.error.create_user` | `application.provision.user.push` |
| `app.sendwordnow.api.error.get_user` | `application.provision.user.import_profile` |
| `app.sendwordnow.api.error.import_user_profile` | `application.provision.user.import_profile` |
| `app.sendwordnow.api.error.service` | `application.provision.user.import` |
| `app.sendwordnow.api.error.update_user_profile` | `application.provision.user.push_profile` |
| `app.sendwordnow.api.error.user_exists` | `application.provision.user.verify_exists` |
| `app.servicenow.api.error.check.user.exists` | `application.provision.user.verify_exists` |
| `app.servicenow.api.error.create.new.user` | `application.provision.user.push` |
| `app.servicenow.api.error.deactivate.user` | `application.provision.user.deactivate` |
| `app.servicenow.api.error.download.users` | `application.provision.user.import` |
| `app.servicenow.api.error.get.costcenters` | `application.configuration.import_schema` |
| `app.servicenow.api.error.get.departments` | `application.configuration.import_schema` |
| `app.servicenow.api.error.get.locations` | `application.configuration.import_schema` |
| `app.servicenow.api.error.import.manager.profile` | `application.provision.user.push_profile` |
| `app.servicenow.api.error.import.user.profile` | `application.provision.user.import_profile` |
| `app.servicenow.api.error.push.password.update` | `application.provision.user.password` |
| `app.servicenow.api.error.push.profile.update` | `application.provision.user.push_profile` |
| `app.servicenow.api.error.reactivate.user` | `application.provision.user.reactivate` |
| `app.servicenow.api.error.validation` | `application.integration.authentication_failure` |
| `app.servicenow_app2.api.error.add.group.memberships` | `application.provision.group_membership.add` |
| `app.servicenow_app2.api.error.check.user.exists` | `application.provision.user.verify_exists` |
| `app.servicenow_app2.api.error.create.new.user` | `application.provision.user.push` |
| `app.servicenow_app2.api.error.deactivate.user` | `application.provision.user.deactivate` |
| `app.servicenow_app2.api.error.delete.group.memberships` | `application.provision.group_membership.remove` |
| `app.servicenow_app2.api.error.delete.group` | `application.provision.group.remove` |
| `app.servicenow_app2.api.error.download.group.memberships` | `application.provision.group_membership.import` |
| `app.servicenow_app2.api.error.download.groups` | `application.provision.group.import` |
| `app.servicenow_app2.api.error.download.users` | `application.provision.user.import` |
| `app.servicenow_app2.api.error.get.costcenters` | `application.configuration.import_schema` |
| `app.servicenow_app2.api.error.get.departments` | `application.configuration.import_schema` |
| `app.servicenow_app2.api.error.get.locations` | `application.configuration.import_schema` |
| `app.servicenow_app2.api.error.import.manager.profile` | `application.provision.user.import_profile` |
| `app.servicenow_app2.api.error.import.user.profile` | `application.provision.user.import_profile` |
| `app.servicenow_app2.api.error.push.password.update` | `application.provision.user.password` |
| `app.servicenow_app2.api.error.push.profile.update` | `application.provision.user.push_profile` |
| `app.servicenow_app2.api.error.reactivate.user` | `application.provision.user.reactivate` |
| `app.servicenow_app2.api.error.upsert.group` | `application.provision.group.add` |
| `app.servicenow_app2.api.error.validation` | `application.integration.authentication_failure` |
| `app.servicenow_app2.api.warn.upsert.group` | `application.provision.group.verify_exists` |
| `app.sugarcrm.api.error.check.user.exists` | `application.provision.user.verify_exists` |
| `app.sugarcrm.api.error.create.new.user` | `application.provision.user.push` |
| `app.sugarcrm.api.error.deactivate.user` | `application.provision.user.deactivate` |
| `app.sugarcrm.api.error.download.users` | `application.provision.user.import` |
| `app.sugarcrm.api.error.get.entry.list` | `application.integration.general_failure` |
| `app.sugarcrm.api.error.hash.password` | `application.integration.general_failure` |
| `app.sugarcrm.api.error.import.user.profile` | `application.provision.user.import_profile` |
| `app.sugarcrm.api.error.login` | `application.integration.authentication_failure` |
| `app.sugarcrm.api.error.logout` | `application.integration.authentication_failure` |
| `app.sugarcrm.api.error.push.password.update` | `application.provision.user.password` |
| `app.sugarcrm.api.error.push.profile.update` | `application.provision.user.push_profile` |
| `app.sugarcrm.api.error.reactivate.user` | `application.provision.user.reactivate` |
| `app.sugarcrm.api.error.set.entry` | `application.integration.general_failure` |
| `app.user_management.activate_user` | `application.provision.user.activate` |
| `app.user_management.app_group_member_import.delete_failure` | `app.user_management` |
| `app.user_management.app_group_member_import.delete_success` | `app.user_management` |
| `app.user_management.app_group_member_import.insert_failure` | `app.user_management` |
| `app.user_management.app_group_member_import.insert_success` | `app.user_management` |
| `app.user_management.deactivate_user.api_account` | `application.provision.user.deactivate` |
| `app.user_management.deactivate_user` | `application.provision.user.deactivate` |
| `app.user_management.deprovision_task_complete` | `application.provision.user.deprovision` |
| `app.user_management.grouppush.mapping.and.groups.deleted.rule.deleted` | `application.provision.group_push.mapping.and.groups.deleted.rule.deleted` |
| `app.user_management.grouppush.mapping.app.group.renamed.failed` | `application.provision.group_push.mapping.app.group.renamed.failed` |
| `app.user_management.grouppush.mapping.app.group.renamed` | `application.provision.group_push.mapping.app.group.renamed` |
| `app.user_management.grouppush.mapping.created.from.rule.error.duplicate` | `app.user_management.grouppush.mapping.created.from.rule.error.duplicate` |
| `app.user_management.grouppush.mapping.created.from.rule.error.validation` | `app.user_management.grouppush.mapping.created.from.rule.error.validation` |
| `app.user_management.grouppush.mapping.created.from.rule.errors` | `app.user_management.grouppush.mapping.created.from.rule.errors` |
| `app.user_management.grouppush.mapping.created.from.rule.warning.duplicate.name.tobecreated` | `application.provision.group_push.mapping.created.from.rule.warning.duplicate.name.tobecreated` |
| `app.user_management.grouppush.mapping.created.from.rule.warning.duplicate.name` | `application.provision.group_push.mapping.created.from.rule.warning.duplicate.name` |
| `app.user_management.grouppush.mapping.created.from.rule.warning.upsertGroup.duplicate.name` | `application.provision.group_push.mapping.created.from.rule.warning.upsertGroup.duplicate.name` |
| `app.user_management.grouppush.mapping.created.from.rule` | `app.user_management.grouppush.mapping.created.from.rule` |
| `app.user_management.grouppush.mapping.created` | `application.provision.group_push.mapping.created` |
| `app.user_management.grouppush.mapping.deactivated.source.group.renamed.failed` | `application.provision.group_push.mapping.deactivated.source.group.renamed.failed` |
| `app.user_management.grouppush.mapping.deactivated.source.group.renamed` | `application.provision.group_push.mapping.deactivated.source.group.renamed` |
| `app.user_management.grouppush.mapping.okta.users.ignored` | `app.user_management.grouppush.mapping.okta.users.ignored` |
| `app.user_management.grouppush.mapping.update.or.delete.failed.with.error` | `application.provision.group_push.mapping.update.or.delete.failed.with.error` |
| `app.user_management.grouppush.mapping.update.or.delete.failed` | `application.provision.group_push.mapping.update.or.delete.failed` |
| `app.user_management.grouppush.pushed` | `application.provision.group_push.pushed` |
| `app.user_management.grouppush.removed` | `application.provision.group_push.removed` |
| `app.user_management.grouppush.updated` | `application.provision.group_push.updated` |
| `app.user_management.import.csv.line.error` | `app.user_management.import.csv.line.error` |
| `app.user_management.importing_profile_failed.email_length` | `system.import.import_profile` |
| `app.user_management.importing_profile_failed.missing_externalid` | `system.import.import_profile` |
| `app.user_management.importing_profile` | `system.import.import_profile` |
| `app.user_management.provision_user.user_inactive` | `application.provision.user.push_profile` |
| `app.user_management.provision_user_failed` | `application.provision.user.sync` |
| `app.user_management.provision_user` | `application.provision.user.sync` |
| `app.user_management.push_new_user_success` | `app.user_management.push_new_user_success` |
| `app.user_management.push_new_user` | `application.provision.user.push` |
| `app.user_management.push_okta_password_update` | `application.provision.user.push_okta_password` |
| `app.user_management.push_pending_user` | `application.provision.user.push` |
| `app.user_management.push_profile_failure` | `application.provision.user.push_profile` |
| `app.user_management.push_profile_success` | `application.provision.user.push_profile` |
| `app.user_management.push_profile_update` | `application.provision.user.push_profile` |
| `app.user_management.push_unique_password_update` | `application.provision.user.push_password` |
| `app.user_management.reactivate_user` | `application.provision.user.reactivate` |
| `app.user_management.unsuspend_user_after_confirm_failed` | `system.import.user.unsuspend_after_confirm` |
| `app.user_management.update_from_master_failed` | `app.user_management.update_from_master_failed` |
| `app.user_management.update_user_lifecycle_from_master_failed` | `system.import.user.update_user_lifecycle_from_master` |
| `app.user_management.updating_api_credentials_for_password_change` | `application.configuration.update_api_credentials_for_pass_change` |
| `app.user_management.user_group_import.create_failure` | `app.user_management.user_group_import.create_failure` |
| `app.user_management.user_group_import.delete_success` | `app.user_management.user_group_import.delete_success` |
| `app.user_management.user_group_import.update_failure` | `app.user_management.user_group_import.update_failure` |
| `app.user_management.user_group_import.upsert_success` | `app.user_management.user_group_import.upsert_success` |
| `app.user_management.verified_user_with_thirdparty` | `application.provision.user.verify_exists` |
| `app.veeva_vault.api.error.check.user.exists` | `application.provision.user.verify_exists` |
| `app.veeva_vault.api.error.create.new.user` | `application.provision.user.push` |
| `app.veeva_vault.api.error.deactivate.user` | `application.provision.user.deactivate` |
| `app.veeva_vault.api.error.download.custom.objects` | `application.configuration.import_schema` |
| `app.veeva_vault.api.error.download.users` | `application.provision.user.import` |
| `app.veeva_vault.api.error.import.user.profile` | `application.provision.user.import_profile` |
| `app.veeva_vault.api.error.push.profile.update` | `application.provision.user.push_profile` |
| `app.veeva_vault.api.error.reactivate.user` | `application.provision.user.reactivate` |
| `app.veeva_vault.api.error.validation` | `application.integration.authentication_failure` |
| `app.workday.api.error.bind` | `application.integration.general_failure` |
| `app.workday.api.error.connect-custom-report` | `application.provision.user.import` |
| `app.workday.api.error.custom-report-unknown-failure` | `application.provision.user.import` |
| `app.workday.api.error.get-employee-personal-info` | `application.provision.user.push_profile` |
| `app.workday.api.error.get-group-assignments` | `application.provision.group_membership.import` |
| `app.workday.api.error.get-groups` | `application.provision.group.import` |
| `app.workday.api.error.get-locations` | `application.provision.user.import` |
| `app.workday.api.error.get-tx-logs` | `application.provision.user.import` |
| `app.workday.api.error.get-worker-by-id` | `application.provision.user.import_profile` |
| `app.workday.api.error.get-worker-by-username` | `application.provision.user.import` |
| `app.workday.api.error.get-workers` | `application.provision.user.import` |
| `app.workday.api.error.parse-custom-report` | `application.provision.user.import` |
| `app.workday.api.error.parse-group-assignments` | `application.provision.group_membership.import` |
| `app.workday.api.error.parse-groups` | `application.provision.group.import` |
| `app.workday.api.error.parse-workers` | `application.provision.user.import` |
| `app.workday.api.error.universal-directory-setup-error` | `application.provision.user.import` |
| `app.workday.api.error.update-employee-personal-info` | `application.provision.user.push_profile` |
| `app.workday.api.error.user-management-error-download-app-schema` | `application.configuration.import_schema` |
| `app.workday.api.error.user-management-error-push-profile-update` | `application.provision.user.push_profile` |
| `app.workday.api.error.validate` | `application.configuration.update` |
| `app.workday.api.get-custom-report-data-empty` | `application.provision.user.import` |
| `app.yammer.api.error.check.user` | `application.provision.user.verify_exists` |
| `app.yammer.api.error.create.user` | `application.provision.user.push` |
| `app.yammer.api.error.deactivation` | `application.provision.user.deactivate` |
| `app.yammer.api.error.download.users` | `application.provision.user.import` |
| `app.yammer.api.error.import.profile` | `application.provision.user.import_profile` |
| `app.yammer.api.error.push.profile` | `application.provision.user.push_profile` |
| `app.yammer.api.error.validation` | `application.integration.authentication_failure` |
| `app.yammer.api.warn.send.invite` | `application.provision.user.push` |
| `app.zendesk.api.error.role.restriction` | `application.configuration.update` |
| `app.zendesk.api.error.validation.error` | `application.integration.authentication_failure` |
| `core.concurrency.org.limit.violation` | `core.concurrency.org.limit.violation` |
| `core.el.evaluate_failure` | `core.el.evaluate` |
| `core.framework.ratelimit.exceeded` | `system.org.rate_limit.violation` |
| `core.framework.ratelimit.suspicious.ip.exceeded` | `system.org.suspicious.ip.rate_time.violation` |
| `core.framework.ratelimit.upcoming_warning` | `system.org.rate_limit.upcoming_warning` |
| `core.framework.ratelimit.warning` | `system.org.rate_limit.warning` |
| `core.org.config.org_creation.failure` | `system.org.lifecycle.create` |
| `core.org.config.org_creation.success` | `system.org.lifecycle.create` |
| `core.org.task.remove` | `system.org.task.remove` |
| `core.security.escalation` | `missing` |
| `core.user.added_to_rule_exclusion` | `group.user_membership.rule.add_exclusion` |
| `core.user.admin_privilege.granted` | `user.account.privilege.grant` |
| `core.user.admin_privilege.revoked` | `user.account.privilege.revoke` |
| `core.user.call_made.factor` | `system.voice.send_call` |
| `core.user.call_to_send_otp.message_sent.mfa.challenge` | `system.voice.send_mfa_challenge_call` |
| `core.user.call_to_send_otp.message_sent.self_service.account_unlock` | `system.voice.send_account_unlock_call` |
| `core.user.call_to_send_otp.message_sent.self_service.password_reset` | `system.voice.send_password_reset_call` |
| `core.user.call_to_send_otp.message_sent.verify` | `system.voice.send_phone_verification_call` |
| `core.user.config.password_update.failure` | `user.account.update_password` |
| `core.user.config.password_update.success` | `user.account.update_password` |
| `core.user.config.profile_update.success` | `user.account.update_profile` |
| `core.user.config.update_primary_email` | `user.account.update_primary_email` |
| `core.user.config.update_secondary_email` | `user.account.update_secondary_email` |
| `core.user.config.user_activated` | `user.lifecycle.activate` |
| `core.user.config.user_creation.failure` | `user.lifecycle.create` |
| `core.user.config.user_creation.success` | `user.lifecycle.create` |
| `core.user.config.user_deactivated` | `user.lifecycle.deactivate` |
| `core.user.config.user_reactivation.success` | `user.lifecycle.reactivate` |
| `core.user.config.user_status.delete.completed` | `user.lifecycle.delete.completed` |
| `core.user.config.user_status.delete.initiated` | `user.lifecycle.delete.initiated` |
| `core.user.config.user_status.password_mass_expiry` | `user.lifecycle.password_mass_expiry` |
| `core.user.config.user_status.password_reset` | `user.account.reset_password` |
| `core.user.config.user_status.suspended` | `user.lifecycle.suspend` |
| `core.user.config.user_status.unsuspended` | `user.lifecycle.unsuspend` |
| `core.user.email.message_sent.new_device_notification` | `system.email.new_device_notification.sent_message` |
| `core.user.email.message_sent.self_service.account_unlock` | `system.email.account_unlock.sent_message` |
| `core.user.email.message_sent.self_service.password_reset` | `system.email.password_reset.sent_message` |
| `core.user.factor.activate` | `user.mfa.factor.activate` |
| `core.user.factor.attempt_fail` | `user.authentication.auth_via_mfa` |
| `core.user.factor.attempt_success` | `user.authentication.auth_via_mfa` |
| `core.user.factor.deactivate` | `user.mfa.factor.deactivate` |
| `core.user.factor.push_rejected` | `user.mfa.okta_verify.deny_push` |
| `core.user.factor.reset_all` | `user.mfa.factor.reset_all` |
| `core.user.factor.update` | `user.mfa.factor.update` |
| `core.user.impersonation.grant.enabled` | `user.session.impersonation.grant` |
| `core.user.impersonation.grant.extended` | `user.session.impersonation.extend` |
| `core.user.impersonation.grant.revoked` | `user.session.impersonation.revoke` |
| `core.user.impersonation.session.ended` | `user.session.impersonation.end` |
| `core.user.impersonation.session.initiated` | `user.session.impersonation.initiate` |
| `core.user.jit.error.read_only` | `user.lifecycle.jit.error.read_only` |
| `core.user.jit.error` | `missing` |
| `core.user.sms.message_sent.factor` | `system.sms.send_factor_verify_message` |
| `core.user.sms.message_sent.push_verify.activation` | `system.sms.send_okta_push_verify_message` |
| `core.user.sms.message_sent.self_service.account_unlock` | `system.sms.send_account_unlock_message` |
| `core.user.sms.message_sent.self_service.password_reset` | `system.sms.send_password_reset_message` |
| `core.user.sms.message_sent.verify` | `system.sms.send_phone_verification_message` |
| `core.user_auth.account_auto_unlocked` | `user.account.unlock` |
| `core.user_auth.account_locked` | `user.account.lock` |
| `core.user_auth.account_unlocked_by_admin` | `user.account.unlock_by_admin` |
| `core.user_auth.authentication.auth_via_3rd_party_failure` | `user.authentication.authenticate` |
| `core.user_auth.authentication.auth_via_3rd_party_success` | `user.authentication.authenticate` |
| `core.user_auth.authentication.auth_via_okta_mobile_failure` | `user.authentication.authenticate` |
| `core.user_auth.authentication.auth_via_okta_mobile_success` | `user.authentication.authenticate` |
| `core.user_auth.authentication.auth_via_omm_failure` | `user.authentication.authenticate` |
| `core.user_auth.authentication.auth_via_omm_success` | `user.authentication.authenticate` |
| `core.user_auth.authentication.authenticate` | `user.authentication.authenticate` |
| `core.user_auth.credential.enroll` | `user.credential.enroll` |
| `core.user_auth.duo.disabled_lockout` | `user.authentication.auth_via_mfa` |
| `core.user_auth.duo.duo_down` | `user.authentication.auth_via_mfa` |
| `core.user_auth.duo.invalid_integration` | `user.authentication.auth_via_mfa` |
| `core.user_auth.idp.cannot_update_user_profile_or_groups.server_read_only` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.cannot_update_user_profile_or_groups` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.general_schema_warning` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.invalid_user_status` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.login_failed` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.multiple_matching_users` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.no_matching_users` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.saml.assertion_received_same_assertion_id` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.saml.login_success` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.saml.response_received_in_response_to_no_matching_key` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.saml.saml_validation_failed` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.saml.unknown_endpoint` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.saml.unknown_profile_attribute` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.social.cannot_acquire_access_token` | `user.authentication.auth_via_social` |
| `core.user_auth.idp.social.cannot_acquire_profile` | `user.authentication.auth_via_social` |
| `core.user_auth.idp.social.jit_callout_denied_by_callout` | `user.authentication.auth_via_social` |
| `core.user_auth.idp.social.jit_callout_redirect` | `user.authentication.auth_via_social` |
| `core.user_auth.idp.social.jit_callout_response_invalid` | `user.authentication.auth_via_social` |
| `core.user_auth.idp.social.jit_callout_success` | `user.authentication.auth_via_social` |
| `core.user_auth.idp.social.link_callout_denied_by_callout` | `user.authentication.auth_via_social` |
| `core.user_auth.idp.social.link_callout_redirect` | `user.authentication.auth_via_social` |
| `core.user_auth.idp.social.link_callout_response_invalid` | `user.authentication.auth_via_social` |
| `core.user_auth.idp.social.link_callout_success` | `user.authentication.auth_via_social` |
| `core.user_auth.idp.social.link_denied_for_groups` | `user.authentication.auth_via_social` |
| `core.user_auth.idp.social.login_success` | `user.authentication.auth_via_social` |
| `core.user_auth.idp.username_filtered` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.username_transform_failed` | `user.authentication.auth_via_IDP` |
| `core.user_auth.idp.x509.crl_download_failure` | `core.user_auth.idp.x509.crl_download_failure` |
| `core.user_auth.idp.x509.login_success` | `user.authentication.auth_via_IDP` |
| `core.user_auth.invalid_certificate` | `user.authentication.auth` |
| `core.user_auth.invalid_certificate` | `user.session.start` |
| `core.user_auth.login_denied` | `user.session.start` |
| `core.user_auth.login_failed.policy_denied` | `user.session.start` |
| `core.user_auth.login_failed` | `user.authentication.auth` |
| `core.user_auth.login_failed` | `user.session.start` |
| `core.user_auth.login_success` | `user.authentication.auth` |
| `core.user_auth.login_success` | `user.session.start` |
| `core.user_auth.logout_success` | `user.session.end` |
| `core.user_auth.mfa_bypass_attempted` | `user.mfa.attempt_bypass` |
| `core.user_auth.mfa_okta_verify_response` | `user.mfa.okta_verify` |
| `core.user_auth.pki.cert.issue` | `pki.cert.issue` |
| `core.user_auth.pki.cert.renew` | `pki.cert.renew` |
| `core.user_auth.pki.cert.revoke` | `pki.cert.revoke` |
| `core.user_auth.radius.login.failed` | `user.authentication.auth_via_radius` |
| `core.user_auth.radius.login.succeeded` | `user.authentication.auth_via_radius` |
| `core.user_auth.saml2.inbound_saml_login_failed` | `user.authentication.auth_via_inbound_SAML` |
| `core.user_auth.self_service.account_unlock.already_unlocked` | `user.account.unlock` |
| `core.user_auth.self_service.account_unlock.invalid_recovery_token` | `user.account.unlock` |
| `core.user_auth.self_service.account_unlock.invalid_security_answer` | `user.account.unlock` |
| `core.user_auth.self_service.account_unlock.invalid_sms_code` | `user.account.unlock` |
| `core.user_auth.self_service.account_unlock.issued_recovery_token` | `user.account.unlock_token` |
| `core.user_auth.self_service.account_unlock.shared_email` | `user.account.unlock` |
| `core.user_auth.self_service.account_unlock.unknown_user` | `user.account.unlock` |
| `core.user_auth.self_service.account_unlock` | `user.account.unlock` |
| `core.user_auth.self_service.invalid_recovery_token` | `user.account.use_token` |
| `core.user_auth.self_service.password_reset.invalid_recovery_token` | `user.account.reset_password` |
| `core.user_auth.self_service.password_reset.invalid_security_answer` | `user.account.reset_password` |
| `core.user_auth.self_service.password_reset.invalid_sms_code` | `user.account.reset_password` |
| `core.user_auth.self_service.password_reset.invalid_user_state` | `user.account.reset_password` |
| `core.user_auth.self_service.password_reset.issued_recovery_token` | `user.account.reset_password` |
| `core.user_auth.self_service.password_reset.shared_email` | `user.account.reset_password` |
| `core.user_auth.self_service.password_reset.suspended_user` | `user.account.reset_password` |
| `core.user_auth.self_service.password_reset.unknown_user` | `user.account.reset_password` |
| `core.user_auth.self_service.password_reset` | `user.account.reset_password` |
| `core.user_auth.session_clear` | `user.session.clear` |
| `core.user_auth.session_created_using_api_token` | `user.session.start` |
| `core.user_auth.session_created_using_token` | `user.session.start` |
| `core.user_auth.session_expired` | `user.session.expire` |
| `core.user_auth.super_user_app_accessed` | `user.account.access_super_user_app` |
| `core.user_auth.user.account.unlock_failure` | `user.account.unlock_failure` |
| `core.user_group_member.user_add` | `group.user_membership.add` |
| `core.user_group_member.user_remove` | `group.user_membership.remove` |
| `cvd.appuser_profile_bootstrapped` | `directory.app_user_profile.bootstrap` |
| `cvd.appuser_profile_updated` | `directory.app_user_profile.update` |
| `cvd.group_rule_deactivated` | `group.user_membership.rule.deactivated` |
| `cvd.group_rule_invalidated` | `group.user_membership.rule.invalidate` |
| `cvd.group_rule_triggered` | `group.user_membership.rule.trigger` |
| `cvd.mappings_updated` | `directory.mapping.update` |
| `cvd.user_profile_bootstrapped` | `directory.user_profile.bootstrap` |
| `cvd.user_profile_updated` | `directory.user_profile.update` |
| `github.api.error.empty_oauth_token` | `application.integration.authentication_failure` |
| `github.api.error.not_a_member_of_the_org` | `application.integration.authentication_failure` |
| `github.api.error.not_admin_user` | `application.integration.authentication_failure` |
| `github.api.error.rate_limit.remaining` | `application.integration.rate_limit_exceeded` |
| `github.api.error.rate_limit.reset_date` | `application.integration.rate_limit_exceeded` |
| `github.api.error.user_not_found` | `application.provision.user.verify_exists` |
| `gooddata.api.error.incorrect.roles.count` | `application.provision.user.push` |
| `gooddata.api.error.project.access.forbidden` | `application.provision.user.import` |
| `gooddata.api.error.project.assignment.failed` | `application.provision.user.push` |
| `gooddata.api.error.project.not.found` | `application.provision.user.push` |
| `group.application_assignment.add` | `group.application_assignment.add` |
| `group.application_assignment.remove` | `group.application_assignment.remove` |
| `group.application_assignment.skip_assignment_reconcile` | `group.application_assignment.skip_assignment_reconcile` |
| `group.application_assignment.update` | `group.application_assignment.update` |
| `huddle.company_id.validation.failure` | `application.integration.general_failure` |
| `huddle.rate.limit.exceeded` | `application.integration.rate_limit_exceeded` |
| `invalidate_app_list.app.created` | `missing` |
| `invalidate_app_list.app.updated` | `missing` |
| `invalidate_app_list.app_details.updated` | `missing` |
| `invalidate_app_list.metadata.changed` | `missing` |
| `iwa.ad_agents_went_offline` | `system.iwa.go_offline` |
| `iwa.agentless.auth.failure` | `system.iwa_agentless.auth` |
| `iwa.agentless.auth.success` | `system.iwa_agentless.auth` |
| `iwa.agentless.update.failure` | `system.iwa_agentless.update` |
| `iwa.agentless.update.success` | `system.iwa_agentless.update` |
| `iwa.auth` | `user.authentication.auth_via_iwa` |
| `iwa.created_successfully` | `system.iwa.create` |
| `iwa.creating_failed` | `system.iwa.create` |
| `iwa.invalid_certificate` | `user.authentication.auth_via_iwa` |
| `iwa.invalid_token` | `user.authentication.auth_via_iwa` |
| `iwa.invalid_xml_signature` | `user.authentication.auth_via_iwa` |
| `iwa.no_agents_promoted_to_primary` | `system.iwa.promote_primary` |
| `iwa.no_certificate` | `user.authentication.auth_via_iwa` |
| `iwa.ping.error` | `missing` |
| `iwa.ping` | `missing` |
| `iwa.primary_not_found` | `system.iwa.use_default` |
| `iwa.promoted_to_primary` | `system.iwa.promote_primary` |
| `iwa.removed` | `system.iwa.remove` |
| `iwa.updated_successfully` | `system.iwa.update` |
| `iwa.updating_failed` | `system.iwa.update` |
| `iwa.went_online` | `system.iwa.go_online` |
| `mim.command.generic.acknowledged` | `mim.command.generic.acknowledged` |
| `mim.command.generic.cancelled` | `mim.command.generic.cancelled` |
| `mim.command.generic.delegated` | `mim.command.generic.delegated` |
| `mim.command.generic.error` | `mim.command.generic.error` |
| `mim.command.generic.new` | `mim.command.generic.new` |
| `mim.command.generic.notnow` | `mim.command.generic.notnow` |
| `mim.command.ios.acknowledged` | `mim.command.ios.acknowledged` |
| `mim.command.ios.cancelled` | `mim.command.ios.cancelled` |
| `mim.command.ios.error` | `mim.command.ios.error` |
| `mim.command.ios.formaterror` | `mim.command.ios.formaterror` |
| `mim.command.ios.new` | `mim.command.ios.new` |
| `mim.streamDevicesAppListCSVDownload` | `mim.streamDevicesAppListCSVDownload` |
| `mim.streamDevicesCSVDownload` | `mim.streamDevicesCSVDownload` |
| `moveit_dmz.error.too.long.username.or.email` | `application.provision.user.push_profile` |
| `network_zone.rule.disabled` | `network_zone.rule.disabled` |
| `omm.app.eas.cert_based.settings.changed` | `omm.app.eas.cert_based.settings.changed` |
| `omm.app.eas.disabled` | `omm.app.eas.disabled` |
| `omm.app.eas.settings.changed` | `omm.app.eas.settings.changed` |
| `omm.app.VPN.settings.changed` | `omm.app.VPN.settings.changed` |
| `omm.app.WIFI.settings.changed` | `omm.app.WIFI.settings.changed` |
| `omm.cma.created` | `omm.cma.created` |
| `omm.cma.deleted` | `omm.cma.deleted` |
| `omm.cma.updated` | `omm.cma.updated` |
| `omm.enrollment.changed` | `omm.enrollment.changed` |
| `org.not_configured_origin.redirection.usage` | `org.not_configured_origin.redirection.usage` |
| `platform.callback.create` | `callback.create` |
| `platform.callback.delete` | `callback.delete` |
| `platform.callback.execute` | `callback.execute` |
| `platform.callback.failed` | `callback.execute` |
| `platform.callback.modify` | `callback.modify` |
| `platform.field_mapping_rule.assign.change` | `missing` |
| `platform.field_mapping_rule.import.change` | `missing` |
| `platform.group_push.activate_mapping` | `application.provision.group_push.activate_mapping` |
| `platform.group_push.delete_appgroup` | `application.provision.group_push.delete_appgroup` |
| `platform.group_push.push_memberships` | `application.provision.group_push.push_memberships` |
| `platform.tokens.transform.process.failure` | `tokens.transform.process` |
| `platform.tokens.transform.response.failure` | `tokens.transform.response` |
| `platform.tokens.transform.response` | `tokens.transform.response` |
| `platform.union_gaa_failure_event` | `missing` |
| `plugin.downloaded` | `plugin.downloaded` |
| `plugin.script_status` | `plugin.script_status` |
| `policy.activated` | `policy.lifecycle.activate` |
| `policy.created` | `policy.lifecycle.create` |
| `policy.deactivated` | `policy.lifecycle.deactivate` |
| `policy.deleted` | `policy.lifecycle.delete` |
| `policy.execute.user.start` | `policy.execute.user.start` |
| `policy.overwritten` | `policy.lifecycle.overwrite` |
| `policy.rule.action.execute` | `policy.rule.action.execute` |
| `policy.rule.activated` | `policy.rule.activate` |
| `policy.rule.added` | `policy.rule.add` |
| `policy.rule.deactivated` | `policy.rule.deactivate` |
| `policy.rule.deleted` | `policy.rule.delete` |
| `policy.rule.invalidated` | `policy.rule.invalidate` |
| `policy.rule.updated` | `policy.rule.update` |
| `policy.scheduled.execute` | `policy.scheduled.execute` |
| `policy.updated` | `policy.lifecycle.update` |
| `roambi.api.error.auth.empty.account.response` | `application.integration.general_failure` |
| `roambi.api.error.auth.empty.code` | `application.integration.general_failure` |
| `roambi.api.error.auth.unexpected.response` | `application.integration.general_failure` |
| `roambi.api.error.deactivate_user.confirmation` | `application.provision.user.deactivate` |
| `roambi.api.error.reactivate_user.confirmation` | `application.provision.user.reactivate` |
| `security.device.add_request_blacklist_policy` | `security.device.add_request_blacklist_policy` |
| `security.device.remove_request_blacklist_policy` | `security.device.remove_request_blacklist_policy` |
| `security.device.temporarily_disable_blacklisting` | `security.device.temporarily_disable_blacklisting` |
| `security.session.detect_client_roaming` | `security.session.detect_client_roaming` |
| `security.zone.make_blacklist` | `security.zone.make_blacklist` |
| `security.zone.remove_blacklist` | `security.zone.remove_blacklist` |
| `SMS Usage Billing` | `SMS Usage Billing` |
| `verificationFailed` | `application.configuration.update` |
| `zone.activate` | `zone.activate` |
| `zone.create` | `zone.create` |
| `zone.deactivate` | `zone.deactivate` |
| `zone.delete` | `zone.delete` |
| `zone.make_blacklist` | `zone.make_blacklist` |
| `zone.remove_blacklist` | `zone.remove_blacklist` |
| `zone.update` | `zone.update` |

# Resources

This section contains a collection of useful resources that may help in making the switch from `/events` to `/logs`.

## [developer.okta.com](http://developer.okta.com)

The following are the formal developer documentation pages of each API:

- [Events API](https://developer.okta.com/docs/api/resources/events)
- [Logs API](https://developer.okta.com/docs/api/resources/system_log)

## [splunkbase.splunk.com](http://splunkbase.splunk.com)
The following is the add-on for Splunk that is capable of ingesting from Okta's System Log API into your Splunk instance for offline analysis:

- [Logs Splunk Add-On](https://splunkbase.splunk.com/app/3682/)

## [help.okta.com](http://help.okta.com)

The following topic provides a list of possible System Log error events that can occur related to provisioning integrations:

- [Provisioning Integration Error Events](https://help.okta.com/en/prod/Content/Topics/Reference/ref-apps-events.htm)

## [support.okta.com](http://support.okta.com)
The following are a collection of informational articles that dive into specifics of the System Log and its API:

- [About the System Log](https://support.okta.com/help/Documentation/Knowledge_Article/About-the-System-Log-651903282)
- [Exporting Okta Log Data](https://support.okta.com/help/Documentation/Knowledge_Article/Exporting-Okta-Log-Data)
- [Okta's Enhanced System Log Report - Part I](https://support.okta.com/help/blogdetail?id=a67F0000000L2aNIAS)
- [Okta's Enhanced System Log Report - Part II](https://support.okta.com/help/blogdetail?id=a67F0000000L2aSIAS)
- [Okta's Enhanced System Log Report - Part III (FAQ)](https://support.okta.com/help/blogdetail?id=a67F0000000L2aXIAS)
- [Using Session and Request ID Fields in the System Log](https://support.okta.com/help/Documentation/Knowledge_Article/65532538-Using-Session-and-Request-ID-Fields-in-the-System-Log)
- [Useful System Log Queries](https://support.okta.com/help/Documentation/Knowledge_Article/Useful-System-Log-Queries)

## [www.okta.com](http://www.okta.com)
The following covers what the System Log is and where to find it, how to translate logs to actual user activity, and how you can leverage the System Log during a security incident. It also reviews some of the actions you can take to respond to an incident identified within the System Log:

- [Okta Incident Response Guide](https://www.okta.com/incident-response-guide/)

## [github.com/OktaSecurityLabs](https://github.com/OktaSecurityLabs)
The following is a listing of Okta System Log event types of interest for security teams:

- [Security Events](https://github.com/OktaSecurityLabs/CheatSheets/blob/master/SecurityEvents.md)
