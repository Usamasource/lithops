Monitoring Performance with Lithops Profiler
============================================

Introduction to Lithops Profiler
---------------------------------

The **Lithops Profiler** is a tool designed to enhance workload optimization by providing real-time monitoring of performance metrics within the Lithops ecosystem. It captures a wide range of metrics, including CPU, memory, disk, and network activity, for both the main worker process and its child processes. This detailed insight into resource utilization in distributed computing environments enables significant optimizations and more efficient resource management.

Core Capabilities of the Lithops Profiler
-----------------------------------------

- **Comprehensive Monitoring**: Tracks real-time performance metrics of the main worker and its child processes, crucial for accurately analyzing distributed computing environments.

- **Detailed Analysis**: Provides insights into resource usage, covering CPU and memory activity, disk operations, and network behavior, invaluable for identifying bottlenecks and optimizing performance.

- **Integration with Prometheus Pushgateway**: Collected metrics are sent to the Prometheus Pushgateway, which pushes the data to a centralized Prometheus server, facilitating a graphical and detailed interpretation of performance data via Grafana.

- **Customization of Monitoring Frequency**: Users can adjust the data collection frequency of the profiler through the `lithops_config`, providing flexibility in performance analysis.

Enhancing Performance Analysis with Grafana
-------------------------------------------

Grafana, in synergy with Lithops Profiler, transforms the collected metrics into understandable visualizations. This ability to turn raw data into intuitive charts and dashboards is essential for identifying trends and anomalies, and for basing optimization decisions on concrete data.

For detailed instructions on configuring Grafana for use with Lithops, refer to the `Grafana Configuration Guide <https://lithops-cloud.github.io/docs/source/grafana_setup.html>`_.

Lithops Profiler Detailed Metrics Collection
--------------------------------------------

Upon task initiation, the Lithops Profiler activates to systematically collect a comprehensive set of performance metrics across several crucial areas:

**CPU Usage**

- **Percentage**: Provides immediate CPU consumption insights.

- **scputimes**: Breaks down CPU time into categories such as user and system, offering a nuanced understanding of CPU resource allocation.

**Memory**

- Focuses on the volume of memory read, identifying memory-intensive operations.

**Disk**

- Captures data volume read from the disk and analyzes read/write rates to evaluate storage interaction efficiency.

**Network**

- Assesses data transfer rates, crucial for tasks with significant network communication demands.

Each metric includes pertinent process identifiers—PID and parent PID—along with a unique collection ID, enabling targeted performance analysis. This meticulous approach facilitates immediate optimizations and informs longer-term strategic planning.

.. note::

   When reporting CPU utilization metrics for processes, values exceeding 100% can be observed. This is due to how ``psutil``, the underlying library used for system monitoring, calculates CPU usage for processes utilizing multiple CPU cores.

Why Values Can Exceed 100%
~~~~~~~~~~~~~~~~~~~~~~~~~~

- **Multi-Core Utilization**: ``psutil`` reports the total CPU utilization for a process across all cores. Therefore, a process running threads on multiple cores can have a cumulative CPU utilization that exceeds 100%. For example, a process fully utilizing two cores on a dual-core system could be reported as having 200% CPU utilization.

- **Consistency with UNIX Utilities**: This approach is consistent with how CPU usage is represented in UNIX-like systems, particularly in utilities like ``top``, which report CPU utilization per process without averaging it across the number of available CPU cores.

.. note::

   The decision by ``psutil`` to report CPU usage in this manner ensures that high CPU usage by processes is easily identifiable, irrespective of the total number of CPU cores. This might differ from the behavior seen in tools like Windows Task Manager, where CPU usage is averaged across all cores.

Potential Overheads with High Concurrency
-----------------------------------------

.. note::

   When executing hundreds of functions simultaneously with the profiler enabled, you may experience overheads in execution time due to scalability issues with Prometheus Pushgateway. The Pushgateway may struggle to handle the high volume of metrics being pushed in such scenarios. In future updates, we plan to implement open-source solutions like Prometheus Federation to address these scalability challenges and improve performance in high-concurrency scenarios.

Conclusion: Maximizing the Benefits of Lithops Profiler
-------------------------------------------------------

The Lithops Profiler, with its advanced monitoring and integration with visualization tools like Prometheus Pushgateway and Grafana, is an indispensable instrument for workload optimization in distributed computing. By providing a detailed understanding of resource utilization and offering graphical analysis through Grafana, it empowers users to delve into the performance of their systems and implement significant improvements.

To fully exploit the capabilities of the Lithops Profiler, it is recommended to:

- Regularly review the Grafana dashboards.

- Adjust the data collection frequency of the profiler to balance data granularity with system overhead.

- Use the collected metrics to identify and effectively resolve performance bottlenecks.

With these strategies and careful implementation, the Lithops Profiler becomes a powerful tool for optimizing applications and achieving optimal performance in distributed computing environments.

Lithops Configuration
=====================

Using the Profiler with Lithops
-------------------------------

To enable and fully utilize the **Lithops Profiler**, it is necessary to set the `log_level` to **DEBUG** in the Lithops configuration. This ensures that detailed profiling information is captured during execution.

This configuration change is essential for capturing detailed logs and performance metrics generated by the profiler. Once enabled, the profiler can monitor resource utilization efficiently and send detailed data to Prometheus for visualization in Grafana.

Profiler Timeout
----------------

The `profiler_timeout` setting determines the frequency (in seconds) at which the Lithops Profiler collects performance metrics. The default value is set to 10 seconds, offering a fine-grained view of the system's performance. Adjusting this value can help balance the detail of performance data collected with the overhead introduced by the monitoring process.

.. code-block:: yaml

    lithops:
        profiler_timeout: 5

Prometheus Pushgateway Configuration
------------------------------------

For more details on configuring Prometheus Pushgateway for use with Lithops, please refer to the dedicated section in the `metrics documentation <https://lithops-cloud.github.io/docs/source/metrics.html>`_.

Using Prometheus Pushgateway with Lithops Profiler
--------------------------------------------------

The Lithops Profiler can be configured to work with Prometheus Pushgateway in both local and remote setups. This flexibility allows for comprehensive monitoring across various environments, whether you're running Lithops on your local machine or utilizing remote backends like AWS Lambda, virtual machines (VMs), Kubernetes clusters, or other cloud services.

Configuring Prometheus Pushgateway
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To enable Prometheus Pushgateway to receive metrics from the Lithops Profiler, you can set it up on either a local or remote server. The key is to ensure that the Prometheus Pushgateway endpoint is accessible to all Lithops workers, especially when using remote backends.

1. **Install Prometheus Pushgateway**: Install Pushgateway using your package manager or Docker:

   .. code-block:: bash

       sudo apt-get update
       sudo apt-get install prometheus-pushgateway

   Or, for Docker:

   .. code-block:: bash

       docker run -d -p 9091:9091 prom/pushgateway

2. **Start Pushgateway**: Ensure the service is running:

   .. code-block:: bash

       sudo systemctl start prometheus-pushgateway

3. **Update Lithops Configuration**: In your `lithops_config`, specify the Pushgateway URL (whether local or remote):

   .. code-block:: yaml

       prometheus:
         pushgateway_url: 'http://<server-ip>:9091'

   Replace `<server-ip>` with the actual IP address or hostname of your server. If you are running Pushgateway locally, you can use `localhost`.

**Example**

If you are running Pushgateway locally or on a remote server with IP address `203.0.113.10`, your Lithops configuration would look like:

.. code-block:: yaml

    prometheus:
      pushgateway_url: 'http://203.0.113.10:9091'

Additional Considerations for Remote Backends
---------------------------------------------

When using remote backends, such as AWS Lambda, VMs, Kubernetes, or other cloud services, there are extra factors to consider:

- **Public Accessibility**: The Prometheus Pushgateway must be accessible over the internet if workers are running in environments outside your local network.

- **Network Configuration**: Adjust security groups to allow inbound traffic on the Pushgateway port (`9091`) from your workers.

By properly configuring the Prometheus Pushgateway endpoint and ensuring it's accessible, you can effectively collect and monitor performance metrics across your distributed computing environment.

Final Notes
-----------

By following these configurations and considerations, you can successfully set up Prometheus Pushgateway to monitor Lithops metrics in both local and remote scenarios, ensuring comprehensive visibility and facilitating performance optimization across distributed systems.
