
# SRE Instrumentation Challenge - Candidate: Jesús A. Rodríguez

## Step 1: Implementation

* I'm using the python package [prometheus-flask-exporter](https://github.com/rycus86/prometheus_flask_exporter) 
  to expose the http metrics to prometheus because is easy and convenient.
* The docker image is midly optimized to work with python but it could be interesting to:
  * Add a WSGI server (e.g. gunicorn)
  * Add a user to run the app instead of using root
  * Add pip cache mounted from the host to avoid installing the same packages on every docker build
  * Use a package manager (eg. uv, poetry) to save the exact packages version in a lock file, among other benefits
* In the docker compose file it could be interesting to use a different user than root to run the containers,
  as it was mentioned in the previous point


## Step 2: Visualization

* Steps to reproduce the result of this part:
  * Start docker compose:
  ```shell
  $ docker compose start -d
  ```

  * Run the script to generate requests:
  ```shell
  $ scripts/generate_traffic.sh
  ```

  * Go to [Grafana](http://localhost:3000) > Dashboards > New dashboard

* The issue causing the 500 code errors was that the `delete_bucket` function ended up returning a 500 code instead
  of the correct one, 200. It should be fixed now.

