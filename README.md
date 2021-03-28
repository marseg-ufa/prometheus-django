#### Monitored WSGI HTTP server with multiple Django workers, behind nginx reverse proxy.
-----------------------------------------------------------------------------------------

The purpose of the presented approach is to have metrics from all services
within an instance (be it a VM, k8s pod, etx.) served from a single endpoint,
with the instance's internal services being blackboxed from the actual metric
aggregation service's perspective.

In this example, all entities - nginx, Django workers, gunicorn, and prometheus
itself - have their metrics collected.

Django exposes metrics through [django-prometheus][dp] app. There is a port
range that workers can use to expose their metrics.

[statsd_exporter][se] is used as an intermediary between gunicorn and Prometheus.
It collects statsd metrics from gunicorn and exposes them in
Prometheus-consumable format.

Nginx metrics are exposed through [nginx-prometheus-exporter][ne].

Django, gunicorn and nginx expose their metric endpoints inside a Docker network
called 'internal'.

A Prometheus instance collects metrics from these services. It's API endpoint
is accessible from outside 'internal' network.

prometheus_external serves as an example of how metric collection can be
achieved using the federation feature.

It is not recommended to scrape all metrics with federation. In production
environment there should be some aggregation rules applied.

##### Prerequisites
docker and docker-compose

##### Running
Inside project directorym run `docker-compose up`

##### Usage
The application endpoint is available at `localhost:80`
The prometheus endpoint is available at `localhost:9090`
The external prometheus endpoint is available at `localhost:9091`



[dp]: https://github.com/korfuri/django-prometheus 
[se]: https://github.com/prometheus/statsd_exporter
[ne]: https://github.com/nginxinc/nginx-prometheus-exporter
