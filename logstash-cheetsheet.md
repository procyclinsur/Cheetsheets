## Logstash Cheetsheet

#### logstash log dir

the following logdirs are recognized:

| Deployment Type | Output Location |
|-|-|
| docker | stdout |
| apt/dnf | /etc/logstash |
| tar.gz | $LS_HOME/logs |

#### config files

| File Name | Description |
|-|-|
| `logstash.yml` | contains overall system settings including locations for the default pipeline file. |
| `pipeline.yml` | contains general configurations for pipelines and multipipeline deployments. |
