# Cribl Pack for Palo Alto XSIAM
----

Palo Alto Networks' Cortex XSIAM (Extended Security Intelligence and Automation Management) is an AI-driven Security Operations Center (SOC) platform designed to unify and streamline threat detection, investigation, and response across an organization's digital environment. It integrates extended detection and response (XDR), security information and event management (SIEM), and security orchestration, automation, and response (SOAR) capabilities, leveraging advanced analytics and automation to accelerate mean-time-to-respond (MTTR) while reducing manual workload for security teams.

Cribl Steam is often used to modernize customer data architectures by centralizing the aggregation of third-party data sources and simplifying data onboarding into Cortex XSIAM through a single management console. If you're planning a migration from a legacy SIEM to Cortex XSIAM, Cribl Stream is the industry standard for quickly routing your security-relevant data to both platforms. This reduces migration costs, minimizes downtime, and ensures that historical data is preserved and accessible, supporting compliance and analysis needs during the transition.

## About this pack
This pack contains sample data and pipelines that detail the requirements for sending data into Palo Alto Cortex XSIAM.

## Before You Begin
The integration between Cribl and Cortex XSIAM requires some configuration on both platforms. Before you begin, ensure that you have Admin access to both platforms. You will create an API token in the Cortex XSIAM platform and input both that token and an API endpoint in the new XSIAM destination in Cribl Stream. This allows security-relevant events from multiple third-party data sources to be sent from Cribl Stream into a single Cortex XSIAM API endpoint. Stream pipelines must be used to add data source-specific context (fields) to each event to ensure proper parsing and analysis by the Cortex XSIAM platform.

The required source-specific content is maintained by Palo Alto and documented [here](https://docs-cortex.paloaltonetworks.com/r/Cortex-XSIAM/Cortex-XSIAM-Documentation/Data-source-UUIDs).

[This github repo](https://github.com/demisto/content/tree/master/Packs/FortiGate) provides a listing of Palo Alto Content Packs which are helpful in understanding the Palo Alto required ingest format for most data sources.  For example, if you look at the pack for Fortigate, you will see that the required ingest format is CEF.

## Deployment
Log in to your Cortex SIAM Console, select “Settings” from the left menu, then select “Data Sources” to begin adding the third-party integration for Cribl. Click “Add Data Source” from the top left of your console.

Search for “Cribl” or click the “Analytics and SIEM” to select the new “Cribl” integration. After adding the Cribl Integration to your Cortex XSIAM instance, you will be provided with an API URL and an authentication token, which you will use to configure the XSIAM destination in Cribl Stream. This is the only time you will be presented with the token, and you can only add a single instance of the Cribl integration.

Make sure you have saved your endpoint URL and token from above and have them available for reference. The token is presented to the creator of the XSIAM Cribl integration once, upon creating the Cribl integration.

You should now see a new destination in your Cribl Stream Data > Destinations collection titled “XSIAM”. Configuring this new destination should be as easy as clicking on the XSIAM tile, then clicking the “Add Destination” button, and populating the fields.

## Using this Pack
You will need to create a Cribl Stream Pipeline (reference the examples in this pack) for each of your data sources to add several fields that direct each event to the appropriate parsing functionality within XSIAM. Several pipelines and data source samples are included in the Palo Alto XSIAM pack, available in the Cribl Pack Dispensary, for your reference and to help you get started building upon them.

It's important to note that since this pack involves the routing of multiple sources into a single destination, you need to pay attention to your route filters.  This pack highlights a couple different approaches for filtering based on either contents of the event itself or reference to the source __inputID.  The route filtering is very dependent on your environment and the many data sources you pass through them.  Be careful of filter collisions that send events to the wrong pipeline as you begin and continue to add more sources that traverse your route filtering.

This pack contains three pipelines which highlight a couple of important scenarios to understand as you begin your data ingestion with XSIAM.

It is very important that you do not modify events or drop events being sent to Palo Alto XSIAM, as that may affect parsing, detections, and analytics related to streaming, behavioral, or baselining analytics.
A combination of the below fields are to be added to every event in a pipeline that sends data to the XSIAM destination (note the leading double-underscore):
* __sourceIdentifier is required for ALL events
* __vendor and __product this pair of fields are required for SOME events

[This PAN documentation](https://docs-cortex.paloaltonetworks.com/r/Cortex-XSIAM/Cortex-XSIAM-Documentation/Ingest-data-from-Cribl), which provides mappings for each data source to the __sourceIdentifier, __vendor, and __product values, is **required** to send data to XSIAM. Palo Alto will update it over time. Several examples of these field mappings that have been validated with the excellent PAN Engineering and Product teams are included in this Palo Alto XSIAM pack.

For common data sources related to well-known vendor/product combinations, you are required to provide only the __sourceIdentifier.

For syslog, CEF, LEEF, or custom data sources, you will need to send a non-unique __sourceIdentifier (af01292940d7426594d3d3e55ae17ee0) with __vendor and __product fields populated within a pipeline. Depending on the specific data source, you can either statically assign all the values or need to extract them for the __vendor and __product pair from each event when present.  This pack includes a CEF pipeline where the __vendor/__product pair are regex extracted with samples for CheckPoint VPN-1 & Firewall-1 and Fortinet Fortigate.

The most common example of why you might need to dynamically assign or extract the values for __vendor and __product would be when you are receiving multiple data sources via a single syslog source. Cribl provides you with the control, choice, and flexibility to use a combination of filtering and routing to maintain multiple syslog pipelines (per data source) or conditions with a single syslog pipeline to assign values. Within this pack, I’m sticking with my personal preference for using separate pipelines for each data source, rather than assigning multiple __vendor and __product pairs in a single pipeline.

## Validating in XSIAM Console
Log in to your XSIAM console and select “Investigation” and “Query Builder” from the Cortex XSIAM menu on the left, then click the “XQL Builder” tile to validate received events.  Build a query similar to the below.  Dataset names generally follow the following format:  vendor_product_format and are created dynamically.

Query:  dataset = okta_sso_raw

Prepend the above query with 'datamodel' to validate field mnormalization into the appropriate datamodel for XSIAM visibility.

## Release Notes

### Version 2.0.1 - 2026-02-18
Added support for Microsoft Defender data.  The pipeline needs to extract individual records into multiple events and then applies the Palo provided UUID for ingest into XSIAM.

### Version 2.0.0 - 2026-02-13
Added route/pipeline placeholders for all of the currenntly documented data sources that XSIAM supports with unique UUIDs.

### Version 1.1.0 - 2025-11-15
Create a generic pipeline for regex extraction of __vendor/__product for all CEF formatted events.  Added additional data source examples.

### Version 1.0.0 - 2025-05-7
Initial Release timed with the new Cribl Stream XSIAM destination.

## Contributing to the Pack
Please DM the author of this pack on our [Community Slack channel](https://cribl-community.slack.com).

## Contact
The author of this pack is Jim Apger <japger@cribl.io>.

## License
Pack uses the following license: [Apache 2.0](https://github.com/criblio/appscope/blob/master/LICENSE).


