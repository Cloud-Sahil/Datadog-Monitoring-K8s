# Monitor NGINX Pod Logs on Datadog
This guide shows how to correctly install Datadog on Amazon EKS and collect NGINX pod logs, without common installation errors.

---

# Prerequisites
Make sure you have:

- EC2 
- Create EKS cluster
    - Add Eks Nodes 
- Aws configure
- Helm v3+
- A Datadog account
- Datadog API Key
- Datadog Application Key
- Datadog site:
   - datadoghq.com → https://app.datadoghq.com
   - datadoghq.eu → https://app.datadoghq.eu
   - us3.datadoghq.com → https://app.us3.datadoghq.com
  Important:
The Datadog UI URL (app.datadoghq.com) is NOT the value used in Helm.
Always use datadoghq.com, datadoghq.eu, or us3.datadoghq.com.
---

##  Installation Steps & Commands
###  Root 
~~~sh
sudo -i
~~~
###  Update 
~~~sh
apt update
~~~
