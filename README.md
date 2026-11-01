<h1>Jmix Grafana Dashboards</h1>
<h2>Standard Jmix Dashboards</h2>

This repository contains two dashboards for the Micrometer metrics that export for the Prometheus datasource.
This dashboards should be used for the Grafana monitoring system.

There are three dashboards:
<ul>
    <li>The first one is for the <b>Jmix 1.6.0</b> version and later.</li>
    <li>The second one is for the <b>Jmix 2.2.0</b> version and later.</li>
    <li>The third one, <b>Jmix 3 Monitoring</b>, is for the <b>Jmix 3.0</b> version and later.</li>
</ul>
<img src="images/jmix_dashboard.png" alt="Jmix Dashboard"/>

<h2>Jmix 3 Monitoring</h2>

The <code>Jmix 3 Monitoring.json</code> dashboard shows the following Jmix 3 metrics: view opening, data loaders, fragments, UI actions, <code>DataManager</code> loads and saves. It also shows active HTTP sessions, the database connection pool, HTTP requests and the JVM. Durations are shown as average and maximum values, so the dashboard does not require percentiles or histograms. The maximum is the longest single operation in the last few minutes (the Micrometer timer max); it shows rare slow operations that the average hides.

The dashboard can be filtered by application, instance, view and entity. It deliberately shows more panels than every application needs: remove the panels you do not use in the dashboard edit mode, or collapse a whole row.

The <b>Operations in progress</b> row is based on long task timers, which Micrometer creates for every observation. It shows how many operations have started and not yet finished, and how long the longest of them has been running. An operation that hangs appears there before it completes. If long task timers are disabled with <code>management.metrics.observations.ignored-meters = long-task-timer</code>, the row stays empty.

Jmix 3 records these metrics only if observations are enabled:
```
jmix.ui.ui-observation-enabled = true
jmix.core.data-observation-enabled = true
```
The dashboard uses the metric tags introduced in Jmix 3: <code>view_id</code>, <code>lifecycle_name</code>, <code>loader_id</code> and others. If the application sets <code>jmix.ui.legacy-monitoring-enabled = true</code>, view and data loader metrics are recorded with the Jmix 2 tags; use the Jmix 2.x dashboard for such applications.

Like the other dashboards, it reads metrics from the <code>/actuator/prometheus</code> endpoint (see the settings below) and separates applications by the <code>application</code> tag. When importing the dashboard, select the Prometheus data source.

<img src="images/jmix_3_monitoring.png" alt="Jmix 3 Monitoring dashboard"/>

<h2>Base Prometheus Micrometer Settings for Jmix Application</h2>

Add the following libraries:
```
implementation 'org.springframework.boot:spring-boot-starter-actuator'
implementation 'io.micrometer:micrometer-registry-prometheus'
```
Add the following properties to your application:
```
management.endpoints.web.exposure.include=prometheus,health,info,metrics
management.endpoint.health.show-details=always
```
Add the Spring Security configuration

```
@Configuration
public class ActuatorSecurityConfiguration {
    @Bean
    @Order(JmixSecurityFilterChainOrder.FLOWUI - 10)
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.securityMatcher("/actuator/**")
                .authorizeHttpRequests((authorize) -> authorize.requestMatchers("/actuator/**").permitAll());
        return http.build();
    }
}
```

<h2>Application Name and Other Common Tags</h2>
In order to see the application name on the dashboards (especially if you have more than one application), the following application property should be added.

```
management.metrics.tags.application = <YourApplicationName>
```
This tag will be added to all metrics exported with your application. You can also add your own common tags if necessary.

<img src="images/applications_list.png" alt="Applications list"/>

More details about the <b>Spring Boot common tags</b> you can find <a href="https://docs.spring.io/spring-boot/reference/actuator/metrics.html#actuator.metrics.customizing.common-tags">here</a>.

<h2>Percentiles and Histograms</h2>

In addition to the standard set of "Rate/Average Duration/Maximum Duration" metrics, you can export percentile metrics (such as "0.50" and "0.99") with the specific Spring Boot properties. For example, the following property will add percentile metrics for all Jmix metrics.

```
management.metrics.distribution.percentiles.jmix = 0.50,0.95,0.99,0.999
```
Note that you should create your own visualization in your Grafana to visualize such additional metrics.

More details about the <b>Spring Boot per-meter properties</b> you can find <a href="https://docs.spring.io/spring-boot/reference/actuator/metrics.html#actuator.metrics.customizing.per-meter-properties">here</a>. 




