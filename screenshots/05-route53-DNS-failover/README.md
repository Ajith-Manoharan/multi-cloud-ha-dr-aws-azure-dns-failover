# Failover Simulation

This section demonstrates real-time failover testing.

---

## Hosted Zone Created

![Hosted Zone](./01-hosted-zone-created.png)

---

## DNS Test – Primary Response

![DNS Primary](./02-dns-test-primary-response.png)

---

## nslookup Validation

![nslookup](./03-dns-test-response-nslookup.png)

---

## Deregister AWS Target

![Deregister AWS](./04-deregister-aws-target.png)

---

## Route 53 Health Detected Unhealthy

![Route53 Unhealthy](./05-route53-health-detected-unhealthy.png)

---

## Target Draining

![Target Draining](./06-target-draining.png)

---

## AWS Site Down (503 Error)

![AWS Down](./07-aws-site-down.png)

---

## DNS Switched to Azure

![DNS Switched](./08-dns-switched-to-azure.png)

---

## Azure DR Page Loaded

![Azure DR](./09-browser-loaded-azure-dr.png)

---

### Outcome

Automatic DNS-based failover successfully redirected traffic from AWS (Primary) to Azure (Secondary DR) without manual intervention.

