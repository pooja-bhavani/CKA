# Day 2: ☕ Services & Pod Communication Mastery

Welcome to Day 2 of the 10 Days of CKA Challenge 🚀 Today, you'll start by understanding Pods talk to each other within the cluster using a stable IP address.
Different Service types (ClusterIP, NodePort, LoadBalancer)

---

**Learning Goal**: Master Kubernetes Services and inter-pod communication

**Tasks**: 
Internal Communication (ClusterIP)
- Understand how Pods talk to each other within the cluster using a stable IP address.
- Learn about Service types (ClusterIP, NodePort, LoadBalancer)
- Task: Create a backend Deployment and expose it using a ClusterIP service.
- Test: Use kubectl exec from a different Pod to curl the backend service name. Verify that DNS (CoreDNS) is resolving the service name correctly.
- Write LinkedIn post comparing Kubernetes Services vs traditional load balancers
---
## 💬 Engagement Activity (Build Your Visibility)

- ✅ Post your Day 2 Learning on LinkedIn:
- Graphic/Screenshot: A simple diagram showing Pod A → Service → Pod B, or a screenshot of a successful curl to a service name.
- Key Insight: "What you learnt from day 2!"
- Hashtags: #10DaysCKAChallenge #KubernetesNetworking #DevOps #CKA2025


The more you engage, the more visibility you create for your profile 🌟
---
## 🧩 Finding it Difficult?

Don't worry — share your doubt as a post or reach out on:
- 💬 **[Discord Community](https://discord.gg/yMDNaYEP)**
- 💭 **[LinkedIn](https://www.linkedin.com/in/shubhamlondhe1996/)**
- 💬 **[Official Website](https://www.trainwithshubham.com/)** 

## 💡 Day 2 Pro-Tip
---
> Labels are the "Glue": If your service shows ENDPOINTS: <none>, your selectors don't match your Pod labels.

Happy Learning   
*TrainWithShubham*
