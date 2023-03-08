# Week 2 — Distributed Tracing

Observability Tools - is a tool designed to monitor systems and applications via monitors, logs. Unlike individual monitoring tools, observability tools enable a business to have constant insight and receive continuous feedback from their systems. This approach to monitoring and logging provides actionable information to businesses faster than tools focusing on only monitoring or logging can. Observability platforms help businesses understand system behavior and predict outages or problems before they occur. Businesses use observability tools to act proactively and prevent system problems.
OpenTelemetry is a vendor neutral open source standard.

## **Required Homework**

### **Distributed Tracing with Honeycomb**
**Instrument our backend flask application to use Open Telemetry (OTEL) with Honeycomb.io as the provider**
    ![Honeycomb Trace](/journal/images/Week2-Backend_Trace_Honeycomb.png)
**Run queries to explore traces within Honeycomb.io**
    ![Honeycomb Query](/journal/images/Week2-Query_Honeycomb.png)

### **Distrbuted Tracing with AWS Xray**

**Instrument AWS X-Ray into backend flask application**

**Configure and provision X-Ray daemon within docker-compose and send data back to X-Ray API**

**Observe X-Ray traces within the AWS Console**
    ![XRay Trace](/journal/images/Week2-XRay%20Trace.png)

### **Logging with Rollbar**

**Integrate Rollbar for Error Logging**
**Trigger an error an observe an error with Rollbar**
1. Create account at [rollbar.com](https://rollbar.com)
2. Skip Add Apps. Add SDK - Flask
3. Add to `requirements.txt`
    ```
    blinker
    rollbar
    ```
4. Install deps
    ```sh
    pip install -r requirements.txt
    ```
5. Set our access token. You can obtain access token from the flask sample code in rollbar
    ```sh
    export ROLLBAR_ACCESS_TOKEN=""
    gp env ROLLBAR_ACCESS_TOKEN=""
    ```
6. Add in `app.py`

    ```py
    import rollbar
    import rollbar.contrib.flask
    from flask import got_request_exception
    ```

    ```py
    rollbar_access_token = os.getenv('ROLLBAR_ACCESS_TOKEN')
    @app.before_first_request
    def init_rollbar():
        """init rollbar module"""
        rollbar.init(
            # access token
            rollbar_access_token,
            # environment name
            'production',
            # server root directory, makes tracebacks prettier
            root=os.path.dirname(os.path.realpath(__file__)),
            # flask already sets up logging
            allow_logging_basic_config=False)

        # send exceptions from `app` to rollbar, using flask's signal system.
        got_request_exception.connect(rollbar.contrib.flask.report_exception, app)
    ```
7. Add to backend-flask for `docker-compose.yml`
    ```yml
    ROLLBAR_ACCESS_TOKEN: "${ROLLBAR_ACCESS_TOKEN}"
    ```
    
8. On compose up,you can hit the endpoint `/rollbar/test` or cause an error, and all activity will be logged into rollbar
    ![Rollbar](/journal/images/Week%202-Rollbar.png)


### **Logging with AWS CloudWatch**
**Install WatchTower and write a custom logger to send application log data to CloudWatch Log group**
1. add to the requirements.txt
    ```sh
    watchtower
    ```
2. 
    ```sh
    cd backend-flask
    pip install -r requirements.txt
    ```
3. In `app.py`
    ```py
    import watchtower
    import logging
    from time import strftime
    ```
4.  In `app.py` add following
    ```py
    # Configuring Logger to Use CloudWatch
    LOGGER = logging.getLogger(__name__)
    LOGGER.setLevel(logging.DEBUG)
    console_handler = logging.StreamHandler()
    cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
    LOGGER.addHandler(console_handler)
    LOGGER.addHandler(cw_handler)
    LOGGER.info("some message")

    ...
    ...

    @app.after_request
    def after_request(response):
        timestamp = strftime('[%Y-%b-%d %H:%M]')
        LOGGER.error('%s %s %s %s %s %s', timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
        return response

    ..
    ..
    ..

    @app.route("/api/activities/home", methods=['GET'])
    def data_home():
    data = HomeActivities.run(logger=LOGGER)
    return data, 200
    ```
5.  Log in home_activities
    ```py
    class HomeActivities:
        def run(logger):
            logger.info("HomeActivities")
            with tracer.start_as_current_span("home-activities-mock-data"):

    ```
6. Set env vars in `docker-compose.yml`
    ```py
      AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
    ```
7. Logs will appear in AWS Cloudwatch under Log Groups `cruddur`
    ![Cloudwatch Logs](/journal/images/Week2-CloudWatchLogs.png)


## Homework Challenges
**Instrument Honeycomb for the frontend-application to observe network latency between frontend and backend[HARD]**

**Add custom instrumentation to Honeycomb to add more attributes eg. UserId, Add a custom span**

**Run custom queries in Honeycomb and save them later eg. Latency by UserID, Recent Traces**
    ![Honeycomb Query](/journal/images/Week2-SaveQuery_Honeycomb.io.png)

**Setup segments and subsegments in XRay**
  
So I tried to figure this out on my own. https://docs.aws.amazon.com/xray-sdk-for-python/latest/reference/basic.html was a useful resource. In referring that I realised that there was no need to declare segments. we only need to create segments if the framework does not support them. The data is already sent to XRay as "segments". Ideally the next layer of granularity that we needed is "sub-segments". The simplest way to add subsegments with annotations and meta data is as below

```sh
subsegment = xray_recorder.begin_subsegment('user_activities_start')
#Add annotation
subsegment.put_annotation('id',12345)
#add meta data
dict={
    "now":now.isoformat(),
    "results-size":len(model['data'])
}
subsegment.put_metadata('results',dict,'dataset')
xray_recorder.end_subsegment()
```

Additionally we can use the decorator to identify our endpoint. xray_recorder generates a subsegment for the decorated function

```sh
@app.route("/api/activities/@<string:handle>", methods=['GET'])
@xray_recorder.capture('activities_user')
def data_handle(handle):
model = UserActivities.run(handle)
if model['errors'] is not None:
    return model['errors'], 422
else:
    return model['data'], 200

```

By adding the above 2 things to my codebase I was able to get subsegment details in my XRay Trace as shown

![XRay Subsegment Setup](/journal/images/Week2-XRaySubSegment.png)


    
