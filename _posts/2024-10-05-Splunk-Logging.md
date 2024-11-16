---
layout: post
title: Debug in distributed system (Splunk)
date: 2024-05-21
description: "Splunk"
tag: Distributed system
---

## Splunk

Splunk

````
index=application_na sourcetype=fs_newarch_qa source=*gfs-feaid-services*

Response Time: index=application_na sourcetype=fs_newarch_prod source=*fs-utilities*  eventType=END OR eventType=ERROR | timechart avg(duration)

tps:
index=application_na sourcetype=fs_newarch_prod source=*fs-account-search*  eventType=START | eval tps=1 | timechart per_second(tps)

index=application_na sourcetype=fs_newarch_prod source=*fs-account-search* eventType=ERROR | stats count by error.message

sourcetype=fs_newarch_prod /fs/offers/v1/retrieveOffers level=ERROR source="*gfs-offers-services*"



Find what is the client from the time range:
sourcetype=fs_newarch_prod action=*fs.offers.retrieveOffers* source="*gfs-offers-services*" | stats count by common.clientName

Show what most of the error is from this client :
sourcetype=fs_newarch_prod action=*fs.offers.retrieveOffers* source="*gfs-offers-services*"  "common.clientName"="DGPrdODS-PSCU.fdclientcenter.com" eventType = ERROR | stats count by error.message

Show the avg time of that service from that client:
sourcetype=fs_newarch_prod action=*fs.offers.retrieveOffers* source="*gfs-offers-services*"  "common.clientName"="DGPrdODS-PSCU.fdclientcenter.com" eventType = ERROR OR eventType = END | timechart avg(duration)

leSplunk Queries

Version:
index=application_na sourcetype=fs_newarch_cat source=*gfs-offers-odm* caller=RestServletWriter "Writing application/json response" | rex field=_raw "^(?:[^,\n]*,){8}(?P<version>[^,]+)" | table host version | sort host | dedup host

index=application_na sourcetype=fs_newarch_prod service=fs.maintenance.customnewaccount eventType IN (START, ERROR, END, VALIDATION, AUDIT)

index=application_na sourcetype=fs_newarch_qa source=*fs-proxy* eventType=VALIDATION serviceUrl=*/fs/offersodm/v1*

index=application_na sourcetype=fs_newarch_prod source=*fs-account-search* host=L3BVAP1246 eventType=ERROR | stats count by error.message

index=application_na sourcetype=fs_newarch_prod source=*fs-account-search* host=L3BVAP1246 eventType=* | stats count by eventType

index=application_na sourcetype=fs_newarch_prod source=*fs-account-search* host=L3BVAP1246 (*error* OR *exception* ) NOT "RECORD NOT FOUND" NOT "Missing critical AUDIT" NOT "Overriding cid with the requests clientName/certName"

index=application_na sourcetype=fs_newarch_prod source=*fs-account-search* eventType IN (END, ERROR) common.service=fs.account.cardIssuanceTracking
| timechart p95(duration)

index=application_na sourcetype=fs_newarch_prod source=*fs-account-search* eventType IN (END, ERROR) common.service=fs.account.cardIssuanceTracking
| timechart p95(duration)

index=application_na sourcetype=fs_newarch_prod source=*fs-account-search* host=*l1pvap1155* eventType=ERROR | timechart span=1s count by error.message

index=application_na sourcetype=fs_newarch_prod source=*fs-account-search* host=*l1pvap1155*  ( eventType=ERROR OR eventType=END )  | timechart span=1s count

index=application_na sourcetype=fs_newarch_prod source=*fs-account-search* host=*l1pvap1155*  ( eventType=ERROR OR eventType=END )  | timechart avg(duration)

index=application_na sourcetype=fs_newarch_prod source=*fs-memos* host=*L3PVAP1397*  | timechart span=1s count by eventType

index=application_na sourcetype=fs_newarch_cat action=fs.archive.asynccreate | timechart count span=1m

Audit delete
index=application_na sourcetype=fs_newarch_qa source=gfs-audit-delete-batch-1 ("caller=DB2Base" OR "caller=ApiServiceAuditDao" OR "caller=AuditDeleteBatchScheduler" OR service="AuditDeleteBatchScheduler")

Alert:
index=application_na sourcetype=fs_newarch_qa source=gfs-audit-delete-batch-1 service="AuditDeleteBatchScheduler" eventType IN (END, ERROR) | table eventType, detail, duration, eventTime

index=datapower_na "SalesAndProductManagement"

"/SalesAndProductManagement/Originations/"

index=datapower_na "UP MEMBER COUNT in PRIMARY LBG FS_NSA_Originations_LBG IS 0. IT IS LESS THAN MINIMUM UP COUNT - 1. BACKEND CALLS WILL BE ROUTED TO FAILOVER LBG - FS_NSA_Originations_Failover_LBG"

index=datapower_na host=X*CPIR* "*:500 Internal*"  client_ip_addr=*   | eval error_500=1 | timechart  span=1m per_second(error_500) AS 500_responses

index=datapower_na host=X*CPIR* "*:500 Internal*"  client_ip_addr=*   | eval error_500=1 | timechart  span=1m per_second(error_500) AS 500_responses

tps vs response
index=datapower_na host_env=prod host=*Issuing-T1R* latency | dedup gtid | eval tps=1 | timechart span=1s per_second(tps) avg(resp_xmit)

Trafffic
index=application_na sourcetype=fs_newarch_prod source=*fs-account-maint* host=l3pvap1192 eventType=* | timechart span=1s count

Error check
index=application_na sourcetype=fs_newarch_prod source=*fs-account-maint* (*error* OR *exception* ) NOT "RECORD NOT FOUND" NOT "Missing critical AUDIT" NOT "Overriding cid with the requests clientName/certName" NOT "INVALID-ACCT-EXTERNAL-STATUS" host IN (l3pvap1192,l3pvap1193,l3pvap1705)

index=application_na sourcetype=fs_newarch_prod source=*fs-account-maint* (*error* OR *exception* ) NOT "RECORD NOT FOUND" NOT "Missing critical AUDIT" NOT "Overriding cid with the requests clientName/certName" NOT "INVALID-ACCT-EXTERNAL-STATUS" host IN (l3pvap1192,l3pvap1193,l3pvap1705) eventType=* | stats count by error.message

Response time
index=application_na sourcetype=fs_newarch_prod source=*fs-utilities*  eventType=END OR eventType=ERROR | timechart avg(duration)

TPS
index=application_na sourcetype=fs_newarch_prod source=*fs-account-search* host IN (r3pvap1377, l3pvap1194, l3pvap1195, l3pvap1176, l3pvap1177) eventType=START | eval tps=1 | timechart per_second(tps)

Cluster traffic (OM vs CH)
index=application_na sourcetype=fs_newarch_prod source=*fs-account-main* | eval cluster=case(like(host, "l1%"), "Omaha", like(host, "l3%"), "Chandler") | timechart usenull=f count by cluster

index=application_na sourcetype=*prod* source=*gfs-offers-services* eventType=E*
| spath | eval cluster = case( like(host, "%prod-azc%" ), "central", like(host, "%prod-aze%") , "east") | timechart  span=1m count by cluster

Response
index=datapower_na host_env="prod" */fs/offers/v1/retrieveOffers* resp_xmit>3000

index=datapower_na host_env="prod" */fs/* resp_xmit>7000 | stats count by req_uri

Good Reference
index=application_na sourcetype=fs_newarch_prod source=*fs-utilities* eventType=* | eval env=case(like(host, "l%"), "linux", like(host, "fs-utilities%"), "openshift") | timechart count by env

index=application_na sourcetype=fs_newarch_prod source=*fs-utilities* eventType=ERROR | eval env=case(like(host, "l%"), "linux", like(host, "fs-utilities-prod%"), "openshift") | timechart count by env

index=application_na sourcetype=fs_newarch_prod source=*fs-utilities* eventType=ERROR "error.message"="RECORD NOT FOUND" | eval env=case(like(host, "l%"), "linux", like(host, "fs-utilities-prod%"), "openshift") | timechart span=1m count by env

ERROR group by env
index=application_na sourcetype=fs_newarch_prod source=*fs-utilities* eventType=ERROR | eval env=case(like(host, "l%"), "linux", like(host, "fs-utilities%"), "openshift") | stats count by env, error.message

ODS
index=gfsauth_na ```Always used for ODS``` AND
sourcetype=gfsauth_prod ```Production=gfsauth_prod``` ```CAT=gfsauth_cat``` AND
eventType=ODS ```Always used for ODS``` AND
common.cycle=N ```Needs to exist in the sourcetype selection``` AND
transaction.terminalId="MQ8C"

index=gfsauth_na ```Always used for ODS``` AND
sourcetype=gfsauth_prod ```Production=gfsauth_prod``` ```CAT=gfsauth_cat``` AND
eventType=ODS ```Always used for ODS``` AND
common.cycle=N ```Needs to exist in the sourcetype selection``` AND
transaction.terminalId="MQ8C" AND
transaction.tranDesc=*PCF*

index=gfsauth_na ```Always used for ODS``` AND
sourcetype=gfsauth_prod ```Production=gfsauth_prod``` ```CAT=gfsauth_cat``` AND
eventType=ODS ```Always used for ODS``` AND
common.cycle=N ```Needs to exist in the sourcetype selection``` AND
transaction.terminalId="MQ8C" AND
transaction.tranDesc=*PCF* AND
error.message=*"RECORD NOT FOUND"* | stats count by error.cause

index=gfsauth_na ```Always used for ODS``` AND
sourcetype=gfsauth_prod ```Production=gfsauth_prod``` ```CAT=gfsauth_cat``` AND
eventType=ODS ```Always used for ODS``` AND
common.cycle=* ```Needs to exist in the sourcetype selection``` AND
transaction.terminalId="MQ8C" AND
transaction.tranDesc=*PCF* AND
error.message=*"RECORD NOT FOUND"* | stats count by error.cause, common.cycle
````

index=application_na sourcetype=fs_newarch_cat source=_fs-internal-services_ host=_l1cvap1047_ eventType=ERROR | stats count by error.message

index=application_na sourcetype=fs_newarch_cat source=_fs-internal-services_ host=_l1cvap1047_ eventType=ERROR | stats count by eventType
