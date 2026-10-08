# Volume Reduction

## The problem

Firewall traffic logs are among the highest-volume sources in most environments, and most records are routine. In this lab's mock environment, about 92 percent of PAN-OS TRAFFIC records are permitted sessions that ended normally (`tcp-fin`, `aged-out`). The remaining 8 percent, sessions with `deny`, `drop`, or `reset-both`, are the ones needed for investigations and must arrive complete.

Parsing has also added redundancy: each record now carries both the original CSV string in `body.message` and the same values as named fields.

This section addresses both, in order:

1. **Remove the raw message.** Drop `body.message` now that its contents are parsed.
2. **Sample routine traffic.** Forward all denied, dropped, and reset sessions in full, and sample permitted sessions at a ratio you choose.

Reducing volume in the pipeline lowers network, storage, and query costs at every destination, especially ingest-priced SIEM platforms, where firewall logs are often a major licensing cost. Because only routine traffic is sampled, security analysis is unaffected.

## Volume reduction

### Volume reduction by removing the raw message

##### Add another processor to your Syslog source

- Click on `Edit processors` close to the `syslog` source (it should read as 1 to indicate it already has one processor)
- Click on ` Add processor`
- Search and click **Delete Fields**. Telemetry type is **LOGS**.
- Enter a Short description `Delete body.message`
- Add these two conditions (this is same as what we defined before)

| Match | Field | Operator | String |
|---|---|---|---|
| Body | `appname` | Equals | `PAN-OS` |
| Body | `message` | Contains | `,TRAFFIC,end,` |

!!! danger "The second condition is CONTAINS and not EQUAL"
    Make sure you select CONTAINS for the condition that matches the text `,TRAFFIC,end,`

Screenshot of the conditions:

 ![PAN-OS CSV Condition](img/pipeline-field-extraction/parse-csv-condition.png)   

- Click inside `Body Fields` and select `message` [This means if the above conditions are met, the `body` field `message` will be dropped]

Screenshot showing that the `message` field will be deleted:

![Delete Fields](img/volume-reduction/delete-fields.png)

## Sampling ALLOW logs

- Add another processor to the existing 2 processors
- Search for **Sample Logs** and select it
- Give a short description like **sample**
- Click on **Add Condition**
- Select ``Body`` for Match
- Click inside Field and select `pan["action"]`
- Condition to `Equals` (default)
- Click inside String and select `allow`
- Select Drop Ratio to `0.9`

![Delete Fields](img/volume-reduction/sample-pan-os-allow-logs-0.9.png)

| Drop Ratio | How much logs are sent out|
|---|---|
| `0.9` | 10 percent |
| `0.8` | 20 percent |
| `0.5` | 50 percent |
 

Validate within Bindplane before saving to view the volume reduction

![Validate sampling ](img/volume-reduction/sampling-before-after.png)
