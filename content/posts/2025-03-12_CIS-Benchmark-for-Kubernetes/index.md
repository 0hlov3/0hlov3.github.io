+++
title = "CIS Benchmark for Kubernetes"
subtitle = "A kube-bench Approach"
date = 2025-03-12T16:00:00
created_at = "12-03-2025 09:14 PM"
updated_at = "12-03-2025 09:34 PM"
image = "ian-taylor-jOqJbvo1P9g-unsplash.jpg"
tags = ["Kubernetes Security","CIS Benchmark","Container Security","kube-bench","Security Auditing","Kubernetes"]
+++
{{< figure src="ian-taylor-jOqJbvo1P9g-unsplash.jpg" caption="Photo by {{< newtablink \"https://unsplash.com/@carrier_lost?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash\" >}}Ian Taylor{{< /newtablink >}} on {{< newtablink \"https://unsplash.com/photos/blue-and-red-cargo-ship-on-sea-during-daytime-jOqJbvo1P9g?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash\" >}}Unsplash{{< /newtablink >}}" alt="A large cargo ship carrying stacks of multicolored shipping containers travels across a body of water under a cloudy sky." >}}

{{< page-toc >}}

In today’s rapidly evolving IT landscape, ensuring the security and compliance of container orchestration platforms like Kubernetes is more critical than ever. The {{< newtablink \"https://github.com/aquasecurity/kube-bench\" >}}kube-bench{{< /newtablink >}} tool, developed by {{< newtablink \"https://www.aquasec.com/\" >}}Aqua Security{{< /newtablink >}}, plays a pivotal role in this domain. It is an open-source utility designed to assess Kubernetes deployments against the {{< newtablink \"https://www.cisecurity.org/benchmark/kubernetes\" >}}CIS (Center for Internet Security) Benchmark for Kubernetes{{< /newtablink >}}. This benchmark outlines a comprehensive set of security best practices that help organizations safeguard their containerized environments from misconfigurations and vulnerabilities.

By automating the process of security checks, kube-bench empowers Administrators and security professionals to identify potential weaknesses and remediate issues before they can be exploited. It not only streamlines the audit process but also serves as a continuous monitoring solution to ensure that Kubernetes clusters remain compliant with industry-standard security guidelines. Whether you are managing a single cluster or a fleet of them, integrating kube-bench into your security workflow is a proactive step towards achieving a robust and secure infrastructure.

This blog article will explore the inner workings of kube-bench, explain its relevance in today’s security landscape, and provide insights into how it can be effectively used to maintain compliance with the CIS Benchmark for Kubernetes.

## Benefits of Following the CIS Benchmark

Adhering to the CIS Benchmark is not just a regulatory exercise, it’s a strategic decision to enhance your organization’s security posture. By following the CIS Benchmark, you ensure that your Kubernetes environment is configured to mitigate common security risks and vulnerabilities. This rigorous approach not only protects your infrastructure from potential threats but also builds trust with stakeholders by demonstrating a steadfast commitment to security best practices.

## Common Security Pitfalls in Kubernetes

Kubernetes, while powerful, can be misconfigured in several ways. From improper role-based access control (RBAC) settings to exposed API endpoints, there are various vulnerabilities that can be exploited if not addressed. Understanding these pitfalls can help you appreciate the importance of regular security assessments and guide you in taking preemptive measures to secure your clusters.

## How kube-bench Works Under the Hood

kube-bench automates the process of evaluating your Kubernetes setup against the established security guidelines provided by the CIS Benchmark. It systematically runs a series of tests designed to probe various aspects of your cluster's configuration—from node security settings and API server configurations to control plane parameters and more. Each test scrutinizes a specific area of your deployment, checking for potential misconfigurations or deviations from recommended best practices.

After completing its checks, kube-bench compiles a detailed report that outlines the current security posture of your environment. This report not only highlights areas that meet the standards but also clearly identifies configurations that require attention or improvement. Such transparency is invaluable; it provides both new users and seasoned professionals with actionable insights, enabling them to fine-tune their deployments and address vulnerabilities before they can be exploited.

By offering an automated, consistent, and comprehensive audit of your Kubernetes configuration, kube-bench serves as a critical tool in the ongoing effort to secure containerized environments.

## Running kube-bench

There are several ways to deploy kube-bench, each catering to different operational requirements and environments. Here’s a quick overview of the options:

- **As a Docker Container:**  
  Run kube-bench as a standalone Docker container. This method is ideal for quick audits or testing since you can simply pull the container image and execute the security checks without any additional configuration.

- **As a Pod in a Kubernetes Cluster:**  
  Deploying kube-bench directly as a pod within your Kubernetes cluster allows for continuous and integrated security checks. This method is useful if you want kube-bench to run alongside your other workloads, leveraging Kubernetes-native features for scheduling and resource management.

- **Install from Binary:**  
  For environments where containerization is not preferred, you can download the pre-built binary and run kube-bench directly on your host machine. This approach is beneficial for scenarios that require a minimal footprint or when operating on bare metal servers.

- **Compile from Source:**  
  If you need to customize the tool or contribute to its development, you can clone the repository and compile kube-bench from source. This gives you full control over the build process and the ability to modify the tool to suit your specific needs.

### Deploying kube-bench as a Kubernetes Job

In this article, we will deploy kube-bench as a Kubernetes job. Running it as a job provides a clean, repeatable, and isolated way to execute security audits across your cluster. Using a Kubernetes job ensures that the audit runs to completion and produces detailed output that can be used for further analysis and reporting.

We will use the example {{< newtablink \"https://github.com/aquasecurity/kube-bench/blob/main/job.yaml\" >}}Kubernetes Job manifest{{< /newtablink >}} for kube-bench. You should review the versions and edit the configuration to meet your needs. Here, we'll use the default configuration:

```shell
# Run kube-bench on the control plane nodes.
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/refs/heads/main/job-master.yaml
# Run kube-bench on the nodes.
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/refs/heads/main/job-node.yaml
# Run kube-bench using the default job manifest.
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/refs/heads/main/job.yaml
```

By deploying kube-bench as a Kubernetes job, you can schedule regular audits, integrate the results into your CI/CD pipeline, or trigger the job manually whenever you need to assess your cluster’s security posture. This method leverages Kubernetes’ robust job management capabilities, ensuring that each audit is executed in a controlled and predictable manner.

After the job starts, verify its status:

```shell
kubectl get pods
```

You should see an output similar to:

```shell
NAME               READY   STATUS      RESTARTS   AGE
kube-bench-h57nl   0/1     Completed   0          6s
```

### Review the output

To view the audit results, fetch the logs:

```shell
kubectl logs kube-bench-h57nl
```

Below is a shortened version of the log output highlighting key details:

```shell
[INFO] 4 Worker Node Security Configuration
[INFO] 4.1 Worker Node Configuration Files
[FAIL] 4.1.1 Ensure that the kubelet service file permissions are set to 644 or more restrictive
[FAIL] 4.1.2 Ensure that the kubelet service file ownership is set to root:root

== Remediations node ==
4.1.1 Run the below command (based on the file location on your system) on the each worker node.
For example,
chmod 644 /var/vcap/jobs/kubelet/monit

4.1.2 Run the below command (based on the file location on your system) on the each worker node.
For example,
chown root:root /var/vcap/jobs/kubelet/monit
Exception
File is group owned by vcap

== Summary node ==
0 checks PASS
13 checks FAIL
10 checks WARN
0 checks INFO

[INFO] 5 Kubernetes Policies
[INFO] 5.1 RBAC and Service Accounts
[WARN] 5.1.1 Ensure that the cluster-admin role is only used where required
[WARN] 5.1.2 Minimize access to secrets

== Remediations policies ==
5.1.1 Identify all clusterrolebindings to the cluster-admin role. Check if they are used and
if they need this role or if they could use a role with fewer privileges.
Where possible, first bind users to a lower privileged role and then remove the
clusterrolebinding to the cluster-admin role :
kubectl delete clusterrolebinding [name]
Exception
This is site-specific setting.

5.1.2 Where possible, remove get, list and watch access to secret objects in the cluster.
Exception
This is site-specific setting.

== Summary policies ==
0 checks PASS
0 checks FAIL
24 checks WARN
0 checks INFO

== Summary total ==
0 checks PASS
13 checks FAIL
34 checks WARN
0 checks INFO
```

As you can see, the descriptions for each section are clear, providing guidance on how to fix the failing and warning checks. Now it's up to you to address these issues and enhance the security posture of your Kubernetes environment.

## Conclusion

In summary, kube-bench offers a robust and automated way to assess your Kubernetes clusters against the CIS Benchmark. By integrating kube-bench into your security workflow—whether through a Docker container, a pod, or a Kubernetes job—you gain valuable insights into configuration gaps and vulnerabilities that could compromise your environment.

Regular audits using kube-bench not only help you identify and remediate misconfigurations but also build a strong foundation for continuous security improvements. As Kubernetes environments grow more complex, tools like kube-bench become indispensable for ensuring compliance and maintaining a secure infrastructure.

Take the time to review the audit outputs, address any issues highlighted in the reports, and adapt your configurations accordingly. With proactive monitoring and continuous improvement, you can significantly enhance your Kubernetes security posture and build trust with your stakeholders.

## Sources & Further Reading

- **kube-bench Repository:**  
  Explore the official {{< newtablink \"https://github.com/aquasecurity/kube-bench\" >}}kube-bench GitHub repository{{< /newtablink >}} for source code, documentation, and updates.
  
- **Aqua Security:**  
  Learn more about the company behind kube-bench by visiting {{< newtablink \"https://www.aquasec.com/\" >}}Aqua Security{{< /newtablink >}}.
  
- **CIS Benchmark for Kubernetes:**  
  Read the detailed security guidelines and best practices on the {{< newtablink \"https://www.cisecurity.org/benchmark/kubernetes\" >}}CIS website{{< /newtablink >}}.
  
- **Kubernetes Documentation:**  
  For comprehensive guidance on Kubernetes configuration and security.  
  {{< newtablink \"https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/\" >}}Securing a Cluster{{< /newtablink >}}  
  {{< newtablink \"https://kubernetes.io/docs/concepts/security/security-checklist/\" >}}Security Checklist{{< /newtablink >}}

- **OWASP Kubernetes Top Ten:**  
  When adopting Kubernetes, we introduce new risks to our applications and infrastructure. The OWASP Kubernetes Top 10 is aimed at helping security practitioners, system administrators, and software developers prioritize risks around the Kubernetes ecosystem. The Top Ten is a prioritized list of these risks. In the future we hope for this to be backed by data collected from organizations varying in maturity and complexity.  
  {{< newtablink \"https://owasp.org/www-project-kubernetes-top-ten/\" >}}About the Kubernetes Top 10{{< /newtablink >}}

## Don’t Trust Me - Seriously

The author takes no responsibility for any mishaps, broken servers, or existential crises caused by following this information.

If you spot a mistake, have a better way of doing things, or just want to chat about tech, feel free to reach out.

Also, this isn’t an ad - unless my enthusiasm and advocacy for cool stuff count as advertising.