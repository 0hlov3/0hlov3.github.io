+++
title = "Hardening Kubernetes Nodes on Ubuntu"
subtitle = "A CIS Benchmark Approach"
date = 2025-03-09T16:00:00
created_at = "06-03-2025 09:14 PM"
updated_at = "09-03-2025 09:34 PM"
image = "gabriel-heinzer-4Mw7nkQDByk-unsplash.jpg"
tags = ["Kubernetes Security","CIS Benchmark","Ubuntu Hardening","CIS-CAT","Node Security","Kubernetes"]
+++
{{< figure src="gabriel-heinzer-4Mw7nkQDByk-unsplash.jpg" caption="Photo by {{< newtablink \"https://unsplash.com/@6heinz3r?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash\" >}}Luke Chesser{{< /newtablink >}} on {{< newtablink \"https://unsplash.com/photos/text-4Mw7nkQDByk?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash\" >}}Unsplash{{< /newtablink >}}" alt="Speedcurve Performance Analytics - Grafana Dashboard" >}}

{{< page-toc >}}

Hardening a Kubernetes cluster begins at the node level. This guide explains how to secure Ubuntu-based Kubernetes nodes by applying the CIS (Center for Internet Security) Benchmark, which provides a detailed set of best practices for reducing vulnerabilities and strengthening your security posture.

The CIS Benchmark outlines comprehensive security standards for various systems, including operating systems, applications, and network devices. Its recommendations help organizations adhere to industry best practices, thereby minimizing risks and improving overall system security.

For Ubuntu 22.04, the CIS Benchmark offers tailored configuration guidelines to ensure that systems are set up securely from the ground up. By implementing these controls, administrators can significantly reduce the likelihood of security breaches and better meet regulatory and compliance requirements.

CIS Benchmarks typically feature multiple configuration profiles:

- **Level 1:** Offers basic security measures that can be applied with minimal impact on system performance and functionality.

- **Level 2:** Introduces more robust settings intended for environments where a higher security standard is necessary, though these measures might affect system performance if not carefully managed.

In Kubernetes, each node running workloads represents a potential entry point for attackers. Even though Kubernetes provides powerful container orchestration capabilities, its security depends on securing both the control plane and the individual nodes. A compromised node can jeopardize the entire cluster, making it critical to follow these hardening practices.

## What is the Center for Internet Security

The Center for Internet Security (CIS) is a community-driven nonprofit dedicated to making the connected world a safer place. CIS is renowned for developing the CIS Controls and CIS Benchmarks, globally recognized as best practices for securing IT systems and data. By bringing together a global network of IT professionals, CIS continuously evolves these standards to protect against emerging cyber threats. Additionally, CIS provides secure cloud computing solutions through its CIS Hardened Images and supports government cybersecurity initiatives via the Multi-State Information Sharing and Analysis Center (MS-ISAC).

## What are CIS Benchmarks?

CIS Benchmarks are a set of best practice guidelines designed to help organizations secure their IT systems and data. Developed through a collaborative process involving cybersecurity experts, industry stakeholders, and IT professionals from around the globe, these benchmarks provide prescriptive guidance on how to configure systems securely. Here are some key points about CIS Benchmarks:

- **Collaborative Development:**  
    The benchmarks are the result of input from a diverse community of cybersecurity experts and industry stakeholders. This collaborative approach ensures that the guidelines address real-world challenges and incorporate practical, tested security measures.

- **Regular Updates:**  
    As cyber threats continue to evolve, the CIS Benchmarks are periodically updated to reflect the latest security challenges and technological advancements. This ongoing revision process helps ensure that the benchmarks remain relevant and effective against emerging risks.

- **Wide Applicability:**  
    CIS Benchmarks cover a broad range of systems, including operating systems, applications, and network devices. This comprehensive coverage makes them a trusted resource for organizations seeking to establish a strong security posture across various environments.

- **Prescriptive Guidance:**  
    Rather than offering generic advice, the CIS Benchmarks provide specific configuration recommendations and controls. This prescriptive nature helps organizations implement clear, actionable steps to secure their infrastructure.

By following the CIS Benchmarks, organizations can adopt a proactive approach to cybersecurity, minimizing vulnerabilities, ensuring compliance with regulatory standards, and maintaining a resilient security posture in the face of ever-changing threats.

### Deeper Dive into Configuration Profiles

CIS Benchmarks provide two primary configuration profiles, **Level 1** and **Level 2**, each tailored to different operational needs and risk environments. Here's a closer look at what these profiles entail:

#### Level 1: The Baseline

- **Purpose:**  
    Level 1 settings are designed as a security baseline. They enforce essential security measures while minimizing impact on system performance and functionality.

- **Typical Configurations:**

    - **User and Access Controls:** Enforcing strong password policies and ensuring basic role-based access control (RBAC).

    - **Service Management:** Disabling or removing unnecessary services to reduce the attack surface.

    - **Network Security:** Configuring basic firewall rules to limit inbound and outbound traffic.

    - **System Hardening:** Setting secure file permissions, applying essential kernel parameter tweaks via sysctl, and configuring logging and auditing.

    - **Example Setting:** Disabling SSH root login by setting `PermitRootLogin no` in the SSH configuration file.

- **When to Use:**  
    Level 1 is ideal for most production environments where a balance between security and usability is required. It provides a solid foundation for security without significantly disrupting operational workflows.

#### Level 2: Enhanced Security

- **Purpose:**  
    Level 2 builds on the Level 1 baseline with additional controls aimed at environments that demand a higher security posture. These settings are often more restrictive and may have a noticeable impact on system functionality.

- **Typical Configurations:**

    - **Advanced Access Controls:** Implementing stricter authentication methods and more rigorous account lockout policies.

    - **Service and Application Hardening:** Enforcing advanced security configurations on critical services, such as deeper configuration of daemons and additional restrictions on network-facing applications.

    - **System and Kernel Tweaks:** Applying more extensive kernel hardening, such as stricter memory protections and additional sysctl parameters.

    - **Enhanced Logging and Monitoring:** Increasing the verbosity and retention of audit logs to ensure comprehensive tracking of system activities.

    - **Example Setting:** In addition to disabling SSH root login, Level 2 might enforce multifactor authentication for SSH access and further restrict which users can perform administrative tasks.

- **When to Use:**  
    Level 2 is suited for environments where security is paramount, such as in government, finance or any setting dealing with highly sensitive data. The stricter settings provide an extra layer of defense but require careful planning to mitigate potential impacts on system performance and user experience.

#### Choosing Between Profiles

The choice between Level 1 and Level 2 depends on your organization’s risk tolerance and the operational requirements of your environment. While Level 1 is often sufficient for many production systems, organizations with high security demands or compliance requirements may opt for Level 2 to ensure a more robust security posture.

By understanding the distinctions between these profiles, administrators can tailor their security configurations to best match the needs of their specific operational environment while still aligning with industry best practices.

## Why hardening Kubernetes Nodes is Critical

Securing individual Kubernetes nodes is crucial because a single compromised node can expose the entire cluster to attack. Hardening these nodes means systematically reducing vulnerabilities and applying layers of security measures to limit the damage in case of a breach. Here’s why this is so important:

- **Defense in Depth:** By implementing multiple layers of security—such as firewalls, intrusion detection systems, secure container runtimes (with proper seccomp, AppArmor, or SELinux profiles), and strict network segmentation—you create barriers that an attacker must overcome. This layered approach makes it far more challenging for attackers to move laterally across the network once they breach one layer.

- **Regulatory Compliance:** Many industries require adherence to specific security standards and regulations (like PCI DSS, HIPAA, or GDPR). Hardened nodes, configured according to best practices, help organizations meet these requirements, ensuring that sensitive data is protected and that the infrastructure remains audit-ready.

- **Operational Resilience:** A hardened system is inherently more robust. Technical measures such as strict role-based access controls (RBAC), secure boot configurations, and regular patch management reduce the risk of exploitation. Even if an attacker gains access, these controls limit the scope of the breach, protecting critical services and data from cascading failures.

In summary, hardening Kubernetes nodes not only minimizes the immediate risk of an attack but also strengthens the overall security posture of the entire infrastructure, ensuring continuity and reliability in the face of evolving cyber threats.

## Overview of the CIS Benchmark for Ubuntu

The CIS Benchmark for Ubuntu provides a detailed roadmap for securing systems by applying best practices across several technical areas:

- **User and Access Controls:**  
    The benchmark advises enforcing strong password policies, implementing secure PAM (Pluggable Authentication Modules) configurations, and applying role-based access control (RBAC) to restrict system privileges. It also recommends deploying auditing tools to monitor user activities and log any access anomalies for forensic analysis.

- **Service Configuration:**  
    Reducing the attack surface is key. The guidelines suggest disabling unnecessary services and tightly configuring essential daemons. This includes ensuring that running services operate with the least privilege necessary and are isolated using tools such as systemd service sandboxing.

- **System Settings:**  
    Hardening the system involves adjusting kernel parameters via sysctl, setting strict file system permissions, and configuring comprehensive logging through systems like syslog or journald. Additionally, tools like auditd should be employed to continuously monitor system changes and security events.

- **Network Configuration:**  
    A secure network setup is critical. The benchmark recommends configuring firewalls (using tools like ufw or iptables) to restrict inbound and outbound traffic, applying rate limiting, and ensuring that only essential ports and protocols are open.

By aligning Ubuntu nodes with these recommendations, administrators establish a strong security baseline that not only mitigates vulnerabilities but also supports compliance with industry standards, thereby reinforcing the overall security of Kubernetes environments.

## Introducing the CIS-CAT Tool: Lite vs. Pro

The CIS Configuration Assessment Tool (CIS-CAT) offers a streamlined method to automatically evaluate your system's security configuration against the CIS Benchmarks. It detects deviations from best practices and provides detailed reports with actionable recommendations to remediate potential vulnerabilities.

### What is CIS-CAT?

CIS-CAT is a powerful utility designed to compare your Ubuntu nodes' configurations against the established CIS Benchmarks. It scrutinizes settings across multiple domains—such as user privileges, network configurations, and system services—to identify areas of non-compliance. By generating a comprehensive report, CIS-CAT not only highlights misconfigurations but also outlines specific remediation steps to enhance your overall security posture.

**CIS-CAT Lite vs. CIS-CAT Pro**

CIS-CAT is available in two variants, each suited to different operational scales and requirements:

- **CIS-CAT Lite:**

    - **Cost-Effective and Accessible:** Tailored for smaller environments or organizations beginning their security assessments, this version covers core security checks without incurring significant costs.

    - **Self-Service Reporting:** Users can generate straightforward compliance reports that pinpoint critical issues, ideal for initial assessments or less complex infrastructures.

    - **Limited Automation and Customization:** While it effectively executes the essential checks defined by the CIS Benchmarks, it offers less flexibility for integrating into larger, automated security workflows or customizing the assessments to specific needs.

- **CIS-CAT Pro:**

    - **Enhanced Enterprise Features:** Designed for larger or more dynamic environments, this version offers deep configuration analysis, integrating with other security tools and platforms via APIs.

    - **Continuous Compliance Monitoring:** Beyond one-off scans, Pro supports ongoing monitoring of system configurations, issuing real-time alerts when deviations occur.

    - **Customizable Profiles and Automation:** Organizations can tailor the assessment profiles to meet specific regulatory or internal security requirements. It also enables automated remediation workflows, reducing manual intervention and ensuring consistent enforcement of security policies.

Leveraging the right version of CIS-CAT helps maintain a secure baseline for your Kubernetes nodes. For smaller deployments or initial hardening efforts, CIS-CAT Lite provides essential insights. In contrast, enterprises with complex or continuously evolving infrastructures will benefit from the advanced features of CIS-CAT Pro, ensuring that security remains robust as changes occur.

## Installing the CIS-CAT Tool on Ubuntu

This section explains how to install CIS-CAT Lite on your Ubuntu system—a free option ideal for initial hardening efforts.

**Obtaining CIS-CAT Lite**

For this guide, we'll use CIS-CAT Lite due to its cost-effectiveness compared to the Pro version. To get started, visit {{< newtablink \"https://learn.cisecurity.org/cis-cat-lite\" >}}CIS-CAT Lite{{< /newtablink >}}, fill out the registration form, and you’ll receive a download link via email.

**Prerequisites**

Before installing CIS-CAT, ensure your Ubuntu system meets the necessary requirements. The most critical prerequisite is a Java Runtime Environment (JRE), as CIS-CAT is a Java-based tool.

**Step 1: Install Java**

If Java isn’t already installed, you can install OpenJDK 11 by running the following commands in your terminal:

```shell
sudo apt-get update
sudo apt-get install openjdk-11-jre -y
```

- **sudo apt-get update**: Updates your package list, ensuring you have the latest information on available packages.

- **sudo apt-get install openjdk-11-jre -y**: Installs the OpenJDK 11 JRE without prompting for confirmation.

After installation, verify that Java is correctly installed by running:

```shell
java -version
```

This command should display the installed Java version, confirming that your system is ready to run CIS-CAT Lite.

Once Java is installed and verified, you can proceed with the rest of the CIS-CAT Lite setup as provided in the tool’s documentation.

### Step 2: Download the CIS-CAT Tool

Begin by downloading the latest version of CIS-CAT—whether you're opting for Lite or Pro—directly from the official CIS website. For example, if you're interested in the free CIS-CAT Lite version, you can execute a command like the following directly on your server:

```shell
wget https://workbench.cisecurity.org/api/vendor/v1/cis-cat/lite/latest
```

**Note:** The provided URL is for illustration purposes only. Always verify the current download link on the official CIS website to ensure you have the most up-to-date version.

If you receive an email with the download link and a ZIP file, you can transfer the file to your server using a tool like `rsync`. For instance:



```shell
rsync -av "CIS-CAT Lite Assessor v4.48.0.zip" ubuntu@yourserver:/home/ubuntu
```

In this command:

- **`-a`** enables archive mode, preserving the file's properties.

- **`-v`** provides verbose output to help you track the file transfer.

By downloading and transferring the file correctly, you'll be set to proceed with the installation and configuration of CIS-CAT on your Ubuntu system.

### Step 3: Extract the CIS-CAT Package

After downloading the CIS-CAT package, you'll need to extract its contents. First, ensure the `unzip` utility is installed on your Ubuntu system:

```shell
sudo apt-get install unzip -y
```

Next, use the unzip command to extract the downloaded ZIP file:

```shell
unzip CIS-CAT\ Lite\ Assessor\ v4.48.0.zip
```

This command will create a directory containing all the necessary files for CIS-CAT Lite. Once the extraction is complete, you can verify the contents by listing the directory:

```shell
ls -l
```

This step prepares your environment for the subsequent configuration and use of the CIS-CAT tool.

### Step 4: Execute the CIS-CAT Tool

After unzipping the package, change to the extracted directory (typically named "Assessor") and run the tool with root privileges. For example, if you're using CIS-CAT Lite, you can initiate a benchmark scan with the following commands:

```shell
cd Assessor
sudo ./Assessor-CLI.sh -i -rd ./ -nts -rp index
```

Here's what each option does:

- **\-i:** Runs the tool in interactive mode.

- **\-rd ./:** Specifies the directory (in this case, the current directory) where the benchmark data and configuration files are located.

- **\-nts:** Disables the timestamp in the output report name, keeping the file name consistent.

- **\-rp index:** Determines the report profile to use, which directs how the report is generated.

Additionally, you can customize the command with extra parameters:

- **\-b benchmarks/Ubuntu20.04.cfg:** This option specifies the configuration file tailored for Ubuntu 20.04. Make sure to adjust this file path to match your system’s Ubuntu version.

- **\-o report.html:** Defines the output file name for the compliance report, allowing you to easily reference or share the generated report.

Running this command will assess your system against the selected CIS Benchmark, producing a detailed report that highlights both compliant settings and areas needing remediation.

### Step 5: Retrieve the Compliance Report

After CIS-CAT generates the compliance report, you’ll need to transfer it from the Kubernetes node to your local system for review—especially since a web server isn’t running on the node. Use the `rsync` command to securely copy the report. For instance:

```shell
rsync -av ubuntu@yourserver:/home/ubuntu/Assessor/index.html ./
```

In this command:

- **ubuntu@yourserver:** Specifies the login credentials and address of your server.

- **/home/ubuntu/Assessor/index.html:** Is the path to the generated report on the server.

- **./:** Indicates that the report should be copied to your current local directory.

This method ensures you have the report readily available for offline analysis.

**Step 6: Review the Compliance Report**

Once downloaded, open the report using your preferred web browser. The HTML report (commonly named `report.html` or `index.html`) provides a detailed overview of your system’s compliance status with the CIS Benchmark. It will highlight:

- **Compliant Settings:** Areas where your Ubuntu node meets the recommended security guidelines.

- **Non-Compliant Areas:** Specific settings or configurations that deviate from best practices, along with suggested remediation steps.

Examine these details to identify any necessary changes to enhance your system’s security. You can open the report by double-clicking the file in your file explorer or by using a command like:

```shell
xdg-open report.html
```

This final review will help you understand the security posture of your Ubuntu node and guide further improvements as needed.

### Step 7: Apply Remediation Changes

The CIS-CAT report highlights any failed tests along with clear instructions for remediation. For instance, if the SSH configuration does not meet the CIS Benchmark, you might see a recommendation similar to the following:

> ### 4\.2.7 Ensure SSH root login is disabled
> 
> Description:
> 
> The PermitRootLogin parameter specifies if the root user can log in using SSH. The default is prohibit-password .
> 
> Disallowing root logins over SSH requires system admins to authenticate using their own individual account, then escalating to root . This limits opportunity for non-repudiation and provides a clear audit trail in the event of a security incident.
> 
> Edit the /etc/ssh/sshd\_config file to set the parameter above any Include entries as follows:
> 
> `PermitRootLogin no`
> 
> **Note:** First occurrence of a option takes precedence, Match set statements withstanding. If Include locations are enabled, used, and order of precedence is understood in your environment, the entry may be created in a file in Include location.
> 
> 
> 
> Show Assessment Evidence  
>   
> 
> 
> Show Rule Result XML
> 
> References:
> 
> - URL: SSHD\_CONFIG(5)
> 
> 
> 
> CIS Controls V7.0:
> 
> - **Control 4: Controlled Use of Administrative Privileges: **\-- More
> 
> - \>
> 
> 
> 
> CIS Critical Security Controls V8.0:
> 
> - **Control 5: Account Management: **\-- More
> 
> - \>

## **Conclusion**

Securing your Kubernetes cluster starts at the node level, and applying robust hardening techniques is essential to mitigate potential vulnerabilities. In this article, we explored how to use the CIS Benchmark for Ubuntu to establish a secure baseline, emphasizing best practices in user access, service configuration, system settings, and network management. By leveraging tools like CIS-CAT Lite, administrators can efficiently assess their security posture, obtain detailed compliance reports, and implement targeted remediations, such as disabling SSH root logins, to reinforce system integrity.

Following these step-by-step instructions not only helps to minimize the attack surface of your nodes but also ensures that your infrastructure aligns with industry-recognized standards. Whether you are managing a small deployment or a large-scale enterprise environment, adopting these practices is a critical step towards enhancing resilience, ensuring compliance, and maintaining continuous operational security in your Kubernetes ecosystem.

## Sources & Further Reading

- {{< newtablink \"https://www.cisecurity.org/benchmark/ubuntu_linux\" >}}https://www.cisecurity.org/benchmark/ubuntu_linux{{< /newtablink >}}
- {{< newtablink \"https://www.cisecurity.org/cybersecurity-tools/cis-cat-pro/cis-benchmarks-supported-by-cis-cat-pro\" >}}https://www.cisecurity.org/cybersecurity-tools/cis-cat-pro/cis-benchmarks-supported-by-cis-cat-pro{{< /newtablink >}}
- {{< newtablink \"https://learn.cisecurity.org/cis-cat-lite\" >}}https://learn.cisecurity.org/cis-cat-lite{{< /newtablink >}}
- {{< newtablink \"https://medium.com/@aika.nazhimidinova/cis-benchmark-of-ubuntu-22-04-openscap-security-guide-707f206e73c8\" >}}https://medium.com/@aika.nazhimidinova/cis-benchmark-of-ubuntu-22-04-openscap-security-guide-707f206e73c8{{< /newtablink >}}
- {{< newtablink \"https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/03-Lab-Run-CIS-Benchmark-Assessment-tool-on-Ubuntu.md\" >}}https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/03-Lab-Run-CIS-Benchmark-Assessment-tool-on-Ubuntu.md{{< /newtablink >}}

## Don’t Trust Me - Seriously

The author takes no responsibility for any mishaps, broken servers, or existential crises caused by following this information.

If you spot a mistake, have a better way of doing things, or just want to chat about tech, feel free to reach out.

Also, this isn’t an ad - unless my enthusiasm and advocacy for cool stuff count as advertising.

