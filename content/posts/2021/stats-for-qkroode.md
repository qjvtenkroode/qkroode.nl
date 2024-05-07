---
title: "Stats for Qkroode"
date: 2021-06-03T10:50:04+02:00
categories: ["homelab"]
tags: ["minio", "nomad", "cribl", "s3"]
draft: false
---

## Why?
I'm always fascinated with data and now that this website is getting some more purpose for me I've decided to make use of the access and error logs to at least get a sense of the usages.

## No cookies
The easiest route would be to just use something like Google Analytics and install a tracking cookie, but this implies that you have to get consent for tracking activities and thus a cookie banner. You might have noticed that there is no cookie banner on this website since I don't deploy any cookies and despise the cookie banners because most of the time they don't really give you a choice. So no easy route, which is the way I like it. This might be a bit harder, more work and more complex but it also gives greater flexibility for me to play around with the generated data. I'll explain why in a bit.

## Deployment
My homelab setup uses a HashiCorp stack: [Nomad](https://www.nomadproject.io) as a workload manager, [Consul](https://www.consul.io) for service discovery and [Vault](https://www.vaultproject.io) for secret management. Nomad also runs [Traefik](https://traefik.io) as a proxy / service mesh. One of the services that is actually exposed to the outside is this website. It runs as a container and uses Splunk's Docker logging driver to send data to an Observability pipeline, in this case [Cribl](https://cribl.io). The Observability pipeline slices and dices data to make sure it is routed to the right places and only parts of the data that I'm interested in are stored. This setup is for a whole subject on it's own and will be part of a another story. So first things first, storing the raw data somewhere and making sure that it is available in some long term storage. Enter [Minio](https://min.io).

## Minio for storing long term data
Cribl has an default output to AWS S3. But this is of no use to me as I want to keep almost everything locally (on-prem so to speak). Minio to the rescue, Minio is an S3 compatible Object storage solution that is able to run on-premise.

Why use something to store long term data? Well if I wanted to play around with some new data platform or try my hand at some Machine Learning or just a new database it is alway handy if you have a sample dataset. I like to have something that is relatable to me and with which I can build comparative use-cases. Cribl allows me to just replay all or some of the data that I have stored in S3 compatible object storage and send it to a different destination.

### Nomad job
The full nomad job can be found in my [GitHub Homelab-iac repository](https://github.com/qjvtenkroode/homelab-iac/blob/main/nomad/minio.nomad)

Some things that are important to watch out for; keep in mind how you run persistent data loads. In my case a docker volume is mounted from the host machine to the docker container and the Nomad job is only allowed to run on a node that has this location available. This is done by referencing the volume that is need:

```
volume "minio" {
  type = "host"
  read_only = false
  source = "minio"
}
```

For this to work, the volume has to be defined in Nomad's client config.

```
client {
  host_volume "minio" {
    path = "/mnt/minio"
    read_only = false
  }
}
```

Now that Minio is up and running it is time to setup Cribl to store all data that goes through my website's data pipeline in a Minio bucket. This is done by creating a new destination:

![Cribl to MinIO destination](/img/posts/2021/minio-001.png)

and adding this destination to the routes:

![Add destination to Cribl route](/img/posts/2021/minio-002.png)

Also note that the Minio bucket has to exist before this will work. 

Some other observations:
- Data is buffered by Cribl and stored in a bucket every so often. 
- Buffering is done with a temporary file on the Cribl host machine.
- The time between buffering and storing is configurable and best changed to an appropriate duration depending on the thru-put and volume of your pipeline. 

![Cribl to MinIO buffering](/img/posts/2021/minio-003.png)

## Timeseries
For now all this data will be left to its devices and maybe replayed into a Influxdb database to play around with.  The best part is that I do not have to decide just yet and even when I've decided and want to try something new it is easy to replay all data and send it off to another destination. But that is something for another time. 

