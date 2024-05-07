---
title: "Nomad - Using Vault secrets in docker logging driver"
date: 2021-01-23T22:12:02+01:00
categories: ["homelab"]
tags: ["infrastructure-as-code", "nomad", "vault", "secrets", "cribl", "splunk", "how-to"]
draft: false
---

## Scheduler
[Nomad](https://www.nomadproject.io) is an awesome scheduler for not only docker containers but also single run applications or scripts. The only downside to having containers running, being destroyed and having a new fresh one spin up whenever I want or whenever it fails a health check is that collecting logging can be a hassle.

## Logging
Luckily Docker has a logging driver that sends data towards my favorite data platform, [Splunk](https://www.splunk.com). The logging driver uses HTTP Event Collector (HEC) to push data towards a HEC endpoint. Since the driver uses a HEC endpoint I can send this data towards a data pipeline first that accepts HEC connections. [Cribl](https://cribl.io) is my go-to data pipeline, accepts HEC connections and is able to send data towards Splunk after passing it through a pipeline!

Setting the log driver is easy in Nomad, just add a logging stanza in `job > group > task > config`:
```
logging {
	type = "splunk"
	config {
		splunk-token = "<a-HEC-token>"
		splunk-url = "http://cribl.service.consul:2400"
		splunk-format = "json"
	}
}
```
The downside to this is a potential token exposure when committing the Nomad job as code (yes, it [happened to me](https://github.com/qjvtenkroode/homelab-iac/commit/af85ba40fe5026a5918f5c57b748319fc8d691c3#diff-8d2ed2ed1a347671a22ea67e668a4181402877a8d392452d3e3fa01addb8e499R24).... luckily my data platform and data pipeline are not exposed to the outside). We can do better.

## Secrets management - Vault
All of HashiCorp’s products provide easy and deep integrations, Nomad and [Vault](https://www.vaultproject.io) do as well. Nomad gives an example where it uses the `template` stanza in combination with a `vault` stanza to set a policy to use with this job, get some secrets from Vault and write them to a file or as an environment variable. At first I thought to use the same strategy but somewhere around setting the `splunk-token` and getting this from Vault. Sadly this fails horribly. After having my first Gitter experience in the Nomad repository and being set in the right direction by @angrycub and this [Nomad job example](https://github.com/angrycub/nomad_example_jobs/blob/master/docker/auth_from_template/auth.nomad), I got this to work. Because Vault secrets are injected into the environment they can be interpolated by Nomad in a job file.

```
config {
	logging {
	type = "splunk"
		config {
			splunk-token = "${TOKEN}"
			splunk-url = "http://cribl.service.consul:2400"
			splunk-format = "json"
		}
	}
}

template {
	change_mode = "restart"
	destination = "local/values.env"
	env = true

	data = <<EOF
{{ with secret "secret/cribl/docker_hec" }}
TOKEN = "{{ .Data.token }}"{{ end }}
EOF
}

vault {
	policies = ["homelab"]
}
```

This time Nomad inserts the token that is pulled from Vault into the config and data starts flowing. The moment you hit this webpage, a log line is send through HEC towards Cribl and then Splunk.

I’ll be off, time to setup logging for all my Nomad jobs 🥳.
