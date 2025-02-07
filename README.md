# ITBench for Site Reliability Engineering (SRE) and Financial Operations (FinOps)

**[Paper](#paper) | [Incident Scenarios](./docs/incident_scenarios.md) | [Tools](./docs/tools.md) | [Maintainers](#maintainers)**

## Walkthrough
### Cluster Setup

#### Local Cluster

For instruction on how to create a kind cluster, please see the instructions [here](./local_cluster/README.md).

#### Remote Cluster

For instruction on how to create an cloud provider based Kubernetes cluster, please see the instructions [here](./remote_cluster/README.md). 

Currently, only AWS is supported.

### Running the Incident Scenarios

Now that our cluster is up and running, let's proceed with the deployment of the observability tools and application stack, injecting the fault, and monitoring of alerts in the Grafana dashboard.

1. Create the `all.yaml` file from the template and update the `kubeconfig` field with the path to the configuration of the Kubernetes cluster. While the file creation needs only to be done once, the `kubeconfig` field must be updated if the file path changes or the cluster you intend to leverage changes.

```bash
cp group_vars/all.yaml.example group_vars/all.yaml
```

2. Deploy the observability tools. 

```bash
make deploy_observability_stack
```
The observability tools deployment includes Prometheus, Grafana, Loki, Elasticsearch, Jaeger, OpenSearch and K8s-events-exporter. For additional details on the observability tools deployed please head [here](./docs/tools.md).

3. Deploy one of the sample applications. In this case we are deploying OpenTelemetery's Astronomy Shop Demo.

```bash
make deploy_astronomy_shop
```
Currently IT-Bench supports two sample applications--OpenTelemetery's Astronomy Shop Demo and Deathstartbench's Hotel Reservation. For additional details on the sample applications please head [here](./docs/sample_applications.md).

4. Once all pods are running, inject the fault for an incident.

```bash
INCIDENT_NUMBER=1 make inject_incident_fault
```
Currently the incident scenarios open-sourced are incidents 1, 3, 23, 26, 27, and 102. One can leverage any one of these incidents at this point in time in their own environemnts. Additional details on the incident scenarios themselves and the fault mechanisms can be found [here].

5. After fault injection, to view alerts in the grafana dashboard, use Port Forward to access the Grafana service.

```bash
kubectl port-forward svc/ingress-nginx-controller -n ingress-nginx 8080:80
```

6. To view Grafana dashboard in your web browser, use the following URL: 

```bash
http://localhost:8080/prometheus/alerting/list
```

7. In the right panel, under the `Grafana` section, click on the `AstronomyNotifications` folder to view the alerts on the dashboard. Four alerts are defined:
- To track `error` across the different services
- To track `latency` across the different services
- To track status of deployments across the different namespaces
- To track Kafka connection status across the Kafka-related components
An Alert's default `State` is `Normal`. After few minutes, the fault `State` changes to `Firing`, indicating fault manifestation. The alert definitions for Grafana located [here](roles/observability_tools/tasks/alert_rules) and has been curated using this [guide](https://grafana.com/docs/grafana/latest/alerting/alerting-rules/create-grafana-managed-rule/). 

8. (Optional) You only need to do this if you plan to leverage our [SRE-Agent](https://github.com/IBM/itbench-sre-agent). Port forward the topology mapper service by running. 
```bash
kubectl -n kube-system port-forward svc/topology-monitor 8081:8080
```

9. (Optional) You only need to do this if you plan to leverage our [SRE-Agent](https://github.com/IBM/itbench-sre-agent). Leverage the values below for the `.env.tmpl`
```
GRAFANA_URL=http://localhost:8080/prometheus
TOPOLOGY_URL=http://localhost:8081
```

10. To remove the injected fault, run the following `make` command:

```bash
INCIDENT_NUMBER=1 make remove_incident_fault
```
After executing the command, the alert's `State` should change back to `Normal` from `Firing`, indicating that the fault has been removed.

11. Once done you can undeploy the observability, followed by the application stack by running:
```bash
make undeploy_astronomy_shop
make undeploy_observability_stack
```

_Note_: For a full list of `make` commands, run the following command:

```bash
make help
```
## Contributing

To contribute an incident to the testbed, follow these steps:

1. Create an incident directory.

```bash
mkdir roles/incident_<number>
mkdir roles/incident_<number>/tasks
touch roles/incident_<number>/tasks/main.yaml
mkdir roles/incident_<number>/vars
touch roles/incident_<number>/vars/main.yaml
```

2. Add the following variables to the `vars/main.yaml` file of the new incident. Modify the values of each field as needed. The current available domains are `sre` and `finops`. The current available sample applications are `otel_astronomy_shop` and `dsb_hotel_reservation`.

```yaml
scenario_domain: sre
sample_application: otel_astronomy_shop
```

3. Add any more variables needed for the execution of the tasks.

4. Add the tasks to the `tasks/main.yaml` file. All tasks should have the tag `incident_<number>`, where `<number>` is the number of the new incident. Tasks needed for the fault inject should only run when the variable `is_fault_injection` is true. Tasks needed for fault removal should only run when the variable `is_fault_removal` is true. See the existing tasks for details.

5. Add documentation. See the existing tasks for details. Upon completion, run the following command:

```bash
make documentation
```

## Maintainers
- [@mudverma](https://github.com/mudverma)
- [@divyapathak24](https://github.com/divyapathak24)
- [@fali007](https://github.com/fali007)
- [@tingdai](https://github.com/tingdai)
- [@Red-GV](https://github.com/Red-GV)
- [@bekiroguzhan](https://github.com/bekiroguzhan)
