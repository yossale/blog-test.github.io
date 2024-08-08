---
published: true

title: 'Monitoring CloudWatch statistics using Graphana, InfluxDB and Telegraf'
---

We've started checking out monitoring solutions for our AWS-based infrastructure, and we want it to be not-that-expensive, monitor infrastructure (cpu, I/O, network...) and Application statistics

We've looked into several options, and we're currently narrowing it down to Graphana-InfluxDB-Telegraph. 

The idea is as following: Use Telegraf to pull CloudWatch statistics from amazon, save them into the InfluxDB, and use Grafana to present these statistics and manage alerts and notifications. 

(Why not just use the Grafana CloudWatch plugin? Because it doesn't supoort notification, sadly )

## Set up the environment

To test everything, we've set up a docker env: 

#### Create a network
`docker network create monitoring`

#### The Grafana docker
`docker run -d -p 3000:3000 --name graphana --net=monitoring -v $PWD:/var/lib/grafana -e "GF_SECURITY_ADMIN_PASSWORD=secret" grafana/grafana`

#### The Influx docker
`docker run -p 8086:8086 -d --name influxdb --net=monitoring -v $PWD:/var/lib/influxdb influxdb`

**Important!** add `127.0.0.1	influxdb` to your hosts file (see Sanities for durther explanation)

#### The telegraph docker

First we need to generate a config file for our needs, so:
`docker run --rm telegraf --input-filter cloudwatch --output-filter influxdb config > telegraf.conf`

And then we need to fix the region, credentials, and so on (not a lot)
Then run the docker:

`docker run -d --name=telegraf --net=monitoring -v $PWD/telegraf-aws-influx.conf:/etc/telegraf/telegraf.conf:ro telegraf`

and 

`docker logs -f telegraf`

## Let's monitor!

So we have all the services up - graphana, influxDB and telegraf. By now, telegraf should be pulling data from aws cloudwatch, and storing them inside influxDB. So now we need to hook up grapfana into to that data stream! 

Create a new DataSource from your InfluxDb, with db = telegraf (you'll have to input it in the DataSource page) and call influx_cloudwatch

Create a new Dashboard with your influx_cloudwatch data source, and create a new Graph.

This graph is currently filled with Junk data, let's fill it up:
Click on the graph name, 


## Let's Alert!

In order to add alerts to our graphs, we first need to add Alert channels. 
In Grafana, go to Alerting, and add an alerting channel. 
The easiest one imho is the Telegram alert channel.

Just install telegram (on your local machine, and on your mobile phone), and then go to the [BotFather](https://telegram.me/botfather).
Create a bot according to the instructions, and after you create it, run `/token`. It will give you the Bot Api Token. 
The next thing you need is you chatId. To get that, just go to [get_id_bot](https://telegram.me/get_id_bot)
which will give you your Chat Id. 

That's all you need. Now you can go to one of the graphs, hit 'Alerts', and there 'Notifications'

## Sanities

If things don't seem to work:

First, log in to your Grafana docker and run 
`curl -G 'http://influxdb:8086/query?pretty=true' --data-urlencode "q=SHOW DATABASES"`
If the query passes, you can access influx db from the grafana docker
Now run the same thing from the telegraf docke.

Also, the reason you should add "influxdb" to your hosts file is this:
When we use all the dockers in the same network it means they can access each other seamlessly. However, when we open Grafana using our local browser, and try to add influxdb as a Data Source, it is all done on the client - which is the host(!) of the dockers. So, it's doesn't know what is "influxdb". That's why we add it to the hosts file. 







