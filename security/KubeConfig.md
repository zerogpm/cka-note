Where is the default kubeconfig file located in the current environment?

Find the current home directory by looking at the HOME environment variable.

ls -la .kube/config

output:
-rw------- 1 root root 5652 Apr 4 16:44 .kube/config

How many clusters are defined in the default kubeconfig file?

k config view

How many Users are defined in the default kubeconfig file?

output:

users:

- name: kubernetes-admin
  user:
  client-certificate-data: DATA+OMITTED
  client-key-data: DATA+OMITTED

just 1

How many contexts are defined in the default kubeconfig file?

output:

contexts:

- context:
  cluster: kubernetes
  user: kubernetes-admin
  name: kubernetes-admin@kubernetes
  current-context: kubernetes-admin@kubernetes

just 1

What is the user configured in the current context?

user: kubernetes-admin

What is the name of the cluster configured in the default kubeconfig file?

cluster: kubernetes

A new kubeconfig file named my-kube-config is created. It is placed in the /root directory. How many clusters are defined in that kubeconfig file?

```yaml
apiVersion: v1
kind: Config

clusters:
  - name: production
    cluster:
      certificate-authority: /etc/kubernetes/pki/ca.crt
      server: https://controlplane:6443

  - name: development
    cluster:
      certificate-authority: /etc/kubernetes/pki/ca.crt
      server: https://controlplane:6443

  - name: kubernetes-on-aws
    cluster:
      certificate-authority: /etc/kubernetes/pki/ca.crt
      server: https://controlplane:6443

  - name: test-cluster-1
    cluster:
      certificate-authority: /etc/kubernetes/pki/ca.crt
      server: https://controlplane:6443

contexts:
  - name: test-user@development
    context:
      cluster: development
      user: test-user

  - name: aws-user@kubernetes-on-aws
    context:
      cluster: kubernetes-on-aws
      user: aws-user

  - name: test-user@production
    context:
      cluster: production
      user: test-user

  - name: research
    context:
      cluster: test-cluster-1
      user: dev-user

users:
  - name: test-user
    user:
      client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
      client-key: /etc/kubernetes/pki/users/test-user/test-user.key
  - name: dev-user
    user:
      client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
      client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
  - name: aws-user
    user:
      client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
      client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key

current-context: test-user@development
preferences: {}
```

4 clusters

How many contexts are configured in the my-kube-config file?

4 context

What user is configured in the research context?

dev-user

What is the name of the client-certificate file configured for the aws-user?

aws-user.crt

What is the current context set to in the my-kube-config file?

current-context: test-user@development

I would like to use the dev-user to access test-cluster-1. Set the current context to the right one so I can do that.

Once the right context is identified, use the kubectl config use-context command.

k config use-context research --kubeconfig /root/my-kube-config

We don't want to specify the kubeconfig file option on each kubectl command.

Set the my-kube-config file as the default kubeconfig file and make it persistent across all sessions without overwriting the existing ~/.kube/config. Ensure any configuration changes persist across reboots and new shell sessions.

source ~/.bashrc

Open your shell configuration file:
vi ~/.bashrc

Add the following line to export the variable:

export KUBECONFIG=/root/my-kube-config

Apply the Changes:

Reload the shell configuration to apply the changes in the current session:

source ~/.bashrc

14 / 14

With the current-context set to research, we are trying to access the cluster. However something seems to be wrong. Identify and fix the issue.

Try running the kubectl get pods command and look for the error. All users certificates are stored at /etc/kubernetes/pki/users.

```yaml
- name: aws-user
  user:
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
- name: test-user
  user:
    client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key
```

when you at /etc/kubernetes/pki/users/dev-user/

output:

ls /etc/kubernetes/pki/users/dev-user/
dev-user.crt dev-user.csr dev-user.key

you will the name is mispell it. it should

client-certificate: /etc/kubernetes/pki/users/dev-user/dev-user.crt
