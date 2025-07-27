# Day 7: Advanced Use Cases and Community Resources

## **Objective:** Explore more advanced Containerlab features and point to useful resources.

1.  **Containerlab Graph Visualization:**
Generate a visual representation of your topology.

```bash
sudo clab graph -t day6-bgp-fabric.yaml
```
<!-- ToDO : Add right decription here!-->

2.  **Monitoring and Telemetry Integration (Conceptual):**
Containerlab environments are perfect for testing monitoring solutions. You can deploy Prometheus, Grafana, and exporters as additional containers within your lab or on the host and configure your cEOS/SR Linux nodes to send telemetry data (e.g., via gNMI, NetFlow, or SNMP).

3.  **CI/CD Integration (Conceptual):**
Containerlab labs are highly reproducible, making them ideal for automated testing in CI/CD pipelines. You can define your network tests, spin up a lab with Containerlab, run the tests, and tear down the lab – all within your CI/CD workflow.

4.  **Explore More Lab Examples:**
The Containerlab documentation and GitHub repository are full of examples for various vendors and use cases:

* [Containerlab Lab Examples](https://www.google.com/search?q=https://containerlab.dev/lab-examples/)

5.  **Join the Community:**

* Containerlab GitHub: [https://github.com/srl-labs/containerlab](https://github.com/srl-labs/containerlab)
* Containerlab Discord channel (often linked from the main site)

