# Certified Kubernetes Administrator (CKA) Study Notes

## 📚 Overview

This repository contains my personal study notes, exercises, and resources for the Certified Kubernetes Administrator (CKA) exam. The content is organized by exam domains to facilitate structured learning and easy reference.

## 🎯 Exam Information

- **Exam Version**: CKA (Current Version)
- **Kubernetes Version**: 1.28 (Check the [official documentation](https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad) for the version used in your exam)
- **Duration**: 2 hours
- **Passing Score**: 66%
- **Format**: Performance-based (hands-on tasks in a live Kubernetes environment)

## 📋 Curriculum Domains

The repository is organized according to the official CKA exam curriculum:

1. **[Core Concepts](./01-core-concepts/)** - 19%

   - Kubernetes API
   - Pods, Services, and Deployments
   - Namespaces and basic objects

2. **[Scheduling](./02-scheduling/)** - 12%

   - Labels, Selectors, and Annotations
   - Taints and Tolerations
   - Node Affinity

3. **[Networking](./03-networking/)** - 11%

   - Cluster Networking
   - Service Types
   - Network Policies

4. **[Storage](./04-storage/)** - 7%

   - Volumes
   - Persistent Volumes
   - Storage Classes

5. **[Security](./05-security/)** - 12%

   - Authentication
   - Authorization
   - Certificates

6. **[Cluster Maintenance](./06-cluster-maintenance/)** - 11%

   - Cluster Upgrades
   - Backup and Restore
   - Node Maintenance

7. **[Troubleshooting](./07-troubleshooting/)** - 30%
   - Application Failures
   - Control Plane Failures
   - Worker Node Failures

## 🛠️ Resources

### Official Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [CKA Exam Information](https://www.cncf.io/certification/cka/)
- [Candidate Handbook](https://docs.linuxfoundation.org/tc-docs/certification/lf-candidate-handbook)
- [Curriculum Overview](https://github.com/cncf/curriculum)

### Additional Learning Materials

- [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
- [Killer Shell CKA Practice Environment](https://killer.sh/cka)
- [Kubernetes in Action (Book)](https://www.manning.com/books/kubernetes-in-action)

## 🔑 Exam Tips

- Time management is crucial - flag challenging questions and return to them later
- Familiarize yourself with `kubectl` imperative commands
- Practice setting up contexts and switching between them
- Bookmark essential pages from Kubernetes documentation
- Use aliases and shell autocompletion to speed up command typing
- Check out the [exam-tips](./exam-tips/) directory for more specific advice

## 🏃‍♂️ Practice Labs

The [practice-labs](./practice-labs/) directory contains hands-on exercises that mirror the types of tasks you might encounter in the exam.

## 🤝 Contributions

Feel free to submit pull requests for corrections or additional content that might help others preparing for the CKA exam.

## 📝 License

This repository is shared under the [MIT License](LICENSE).

---

⭐ If you find these notes helpful, please consider giving this repository a star! ⭐

> **Note**: These notes are a personal study guide and not an official resource for the CKA exam. Always refer to the official documentation for the most accurate and up-to-date information.
