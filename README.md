# Google Gemini REST Collector IO
----
## About this Pack

* This Pack is built as a complete SOURCE + DESTINATION solution (identified by the IO suffix). Data collection and delivery happen entirely within the Pack's context, eliminating the need to connect it to globally defined Sources and Destinations. 

* This Pack does ***NOT*** connect directly to Gemini model APIs. Instead, it uses the Cloud Logging API (logging.googleapis.com/v2/entries:list) to retrieve Gemini audit events based on the protoPayload.serviceName field.

* ***Note:*** This Pack does not collect events from Google Gemini for Workspace. For that, you will require the [Cribl Google Workspace Pack](https://packs.cribl.io/packs/cribl-google-workspace-rest-io) and enable the `Gemini AI usage in Workspace apps` input. 

* It queries `logging.googleapis.com/v2/entries:list` and filters on `protoPayload.serviceName` to capture Gemini activity. 

### Supported sources (by serviceName filter)

* [Gemini for Google Cloud](https://docs.cloud.google.com/gemini/docs/audit-logging) (AI Companion) — primary source today. Covers Gemini usage across Cloud Console/Workspace routed through AI Companion.
	* Filter: `protoPayload.serviceName="cloudaicompanion.googleapis.com"`
* [Gemini Cloud Assist](https://docs.cloud.google.com/gemini/docs/cloud-assist/audit-logging) (future-proofed) — included for forward compatibility. Most Cloud Assist interactions currently appear under AI Companion above; this selector is included in anticipation of Google shifting more events here.
  	* Filter: `protoPayload.serviceName="geminicloudassist.googleapis.com"`
* [Gemini Code Assist](https://developers.google.com/gemini-code-assist/docs/configure-logging) — Developer/IDE activity. Code completion/explain/refactor in Cloud Workstations / Cloud Shell Editor
    * Filter: `protoPayload.serviceName="geminicodeassist.googleapis.com"`
* [Vertex AI Gemini API](https://cloud.google.com/vertex-ai/docs/generative-ai/models/gemini) — Programmatic Gemini API usage under Vertex AI.      
	* Filter: `protoPayload.serviceName="gemini.googleapis.com"`
* [AI Studio / Generative Language API](https://ai.google.dev) — Developer API key usage (non-Vertex Gemini API).
    * Filter: `protoPayload.serviceName="generativelanguage.googleapis.com"`

## Deployment

* This Pack is configured by default to use the Worker Group's *Default Destination*.
* To use the *Default Destination*: No changes are required. The Pack will route the data to the destination currently set as the Default on the Worker Group.
* To use a different Destination: You must update the Pack's routes to specify your desired Destination.
* For immediate functionality without requiring Pack route filter expression modifications, every bundled Source within this Pack adds a hidden field: `__packsource`. This field allows for seamless routing based on the Pack source.

### Configure the Collectors

* The Collector is currently configured at the project level, but you can also expand to an organization-level log sink by adding a variable called *ORG_ID* in the Pack → Knowledge → Variables page and setting the value to your Organization ID. 

* When ORG_ID is defined, the Collector automatically overrides the PROJECT_ID scope and collects audit logs from all projects within the organization.

* ***Tip***: Organization-level collection requires that your service account has Logging Viewer access at the organization level. If the service account only has project-level permissions, it will not return any data when using ORG_ID.

* After installing the Pack, you must perform the following:

### Configure Authentication

* Create or select a Google Cloud project to use for Gemini log collection.
* From within that project:
	1.	Go to IAM & Admin → Service Accounts.
	2.	Create a service account with the role Logging Viewer.
	3.	Generate a JSON key and save it securely — you’ll provide this key file to the Cribl Collector configuration.
	4.	The collector authenticates using this service account to call the Cloud Logging API (logging.googleapis.com/v2/entries:list).
	5.	Required OAuth scope: ***https://www.googleapis.com/auth/cloud-platform***

### Update and Enable the Collectors
* Update the **Service account credentials** with the JSON key obtained from the above steps.
* Update the **Impersonated account's email address** with the email address used to create the Service account. Note: This is NOT the email of the Service account. It needs to have Owner permissions.

### Configure your Destination/Update Pack Routes
To ensure proper data routing, you must make a choice: retain the current setting to use the Default Destination defined by your Worker Group, or define a new Destination directly inside this pack and adjust the pack's route accordingly.

### Commit and Deploy
Once everything is configured, perform a Commit & Deploy to enable data collection.

## Pack Configurable Items 
The following are the in-Pack configurable items - review/update them as needed. 

### Lookups
The `google_gemini_event_severity.csv` lookup is used to drop selected events based on severity. Update the `event_action` to `drop` for selected severities you do not require.  
* DEFAULT	- No assigned severity level.
* DEBUG	    - Debug or trace information.
* INFO	    - Routine information, such as ongoing status or performance.
* NOTICE	- Normal but significant events, such as start up, shut down, or a configuration change.
* WARNING	- Events that might cause problems.
* ERROR	    - Events that are likely to cause problems.
* CRITICAL	- Critical events that cause more severe problems or outages.
* ALERT	    - A person must take action immediately.
* EMERGENCY	- One or more systems are unusable.

### Variables

The Pack has the following variables:
* `google_gemini_splunk_default_index`: Default index for the Splunk output - defaults to `gemini`.
* `google_gemini_splunk_default_sourcetype`: Default sourcetype for the Splunk output – defaults to `google:gcp:gemini`
* `PROJECT_ID`: Set this to your Project ID.
* `LOOKBACK_DAYS`: This is how far back, in days, the Collector will look for data when first initialized.  – defaults to `7 days`

## Upgrades

Upgrading certain Cribl Packs using the same Pack ID can have unintended consequences. See [Upgrading an Existing Pack](https://docs.cribl.io/stream/packs#upgrading) for details.

## Release Notes

### Version 1.0.0
Initial release

## Contributing to the Pack

To contribute to the Pack, please connect with us on [Cribl Community Slack](https://cribl-community.slack.com/). You can suggest new features or offer to collaborate.

## License
This Pack uses the following license: [Apache 2.0](https://github.com/criblio/appscope/blob/master/LICENSE).