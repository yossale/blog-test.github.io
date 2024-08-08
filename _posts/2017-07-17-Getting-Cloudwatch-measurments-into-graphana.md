---
published: false
title: 'Adding monitoring to Graphana from InfluxDB'
---

Log into the InfluxDB console, and run
`SHOW DATABASES` 

This shows you all the current databases. We're interested in the telegraf one, so run:
`use telegraf`

Now, let's see what measurments we have. But first of all, what are "measurments" in influxDB? Well, measurments [describes the data stored in the associated fields](https://docs.influxdata.com/influxdb/v1.2/concepts/glossary/#measurement)



# TODO
1. Add more agents to the telegraf, so we'll bring AWS/ElasticBeanstalk, AWS/EC2, AWS/RDS
2. 




