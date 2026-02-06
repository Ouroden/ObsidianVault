apache beam hsbc
maly case study

cloud run

DevOps interview questions only hands-on engineers can answer  
  
No definitions. No buzzwords. Only scars.  
1. A deployment succeeded, but users get 502 errors.  
What do you check first, and in what order?  
2. Kubernetes pods are in Running state, but traffic is not reaching the app.  
List the exact components you validate.  
3. CPU is low, memory is stable, but latency is high.  
What metrics do you look at next and why?  
4. A service works in staging but fails in production.  
How do you narrow the difference without guessing?  
5. Your Terraform plan shows unexpected resource replacement.  
How do you stop it without breaking the pipeline?  
6. HPA is not scaling even though traffic increased.  
Which three things do you verify before touching code?  
7. Logs look fine, metrics look fine, but customers report errors.  
What signal do you trust next?  
8. A rollout caused partial outage.  
How do you decide between rollback, roll-forward, or hotfix?  
9. A node suddenly goes NotReady in Kubernetes.  
What are the first commands you run and why?  
10. Cost spiked 40% overnight.  
How do you identify the exact root cause, not guesses?



11. How does DNS resolution work inside a pod?
→ And what do you check when a service isn’t reachable by name?

12. Walk me through what the controller manager does during a Deployment.
→ No rollout status. Reconciliation logic.

3. What happens if a node with local storage gets autoscaled down?
→ Be careful. This one causes data loss in prod more often than you’d think.

4. Post-deploy, latency spikes for 30% of users. No errors. No logs. What now?
→ Your answer reveals if you know how to triage chaos.

5. How do you enforce runtime security in Kubernetes?
→ PSP? AppArmor? OPA? Most people just hope for the best.

6. HPA vs VPA vs Karpenter; when would you NOT use each?
→ Bonus: How would you simulate HPA behavior in staging?

7. Tell me about the last outage you debugged in Kubernetes.
→ No postmortem? You weren’t there.