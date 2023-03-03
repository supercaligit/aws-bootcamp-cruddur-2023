# Week 2 — Distributed Tracing

Observability Tools - is a tool designed to monitor systems and applications via monitors, logs. Unlike individual monitoring tools, observability tools enable a business to have constant insight and receive continuous feedback from their systems. This approach to monitoring and logging provides actionable information to businesses faster than tools focusing on only monitoring or logging can. Observability platforms help businesses understand system behavior and predict outages or problems before they occur. Businesses use observability tools to act proactively and prevent system problems.

## Homework Tasks
1. Instrument our backend flask application to use Open Telemetry (OTEL) with Honeycomb.io as the provider
2. Run queries to explore traces within Honeycomb.io
3. Instrument AWS X-Ray into backend flask application
4. Configure and provision X-Ray daemon within docker-compose and send data back to X-Ray API
Observe X-Ray traces within the AWS Console
Integrate Rollbar for Error Logging
Trigger an error an observe an error with Rollbar
Install WatchTower and write a custom logger to send application log data to CloudWatch Log group





## Homework Challenges
Instrument Honeycomb for the frontend-application to observe network latency between frontend and backend[HARD]
Add custom instrumentation to Honeycomb to add more attributes eg. UserId, Add a custom span
Run custom queries in Honeycomb and save them later eg. Latency by UserID, Recent Traces

