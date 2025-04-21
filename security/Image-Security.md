What secret type must we choose for docker registry?

k create secret

Create a secret with specified type.

A docker-registry type secret is for accessing a container registry.

A generic type secret indicate an Opaque secret type.

A tls type secret holds TLS certificate and its associated key.

Available Commands:
docker-registry Create a secret for use with a Docker registry
generic Create a secret from a local file, directory, or literal
value
tls

k create secret docker-registry

We have an application running on our cluster. Let us explore it first. What image is the application using?

k describe deploy web

We decided to use a modified version of the application from an internal private registry. Update the image of the deployment to use a new image from myprivateregistry.com:5000

The registry is located at myprivateregistry.com:5000. Don't worry about the credentials for now. We will configure them in the upcoming steps

k edit deploy web

spec:
containers: - image: myprivateregistry.com:5000/nginx:alpine
imagePullPolicy: IfNotPresent
name: nginx
resources: {}
terminationMessagePath: /dev/termination-log
terminationMessagePolicy: File
dnsPolicy: ClusterFirst

Are the new PODs created with the new images successfully running?

no

Create a secret object with the credentials required to access the registry.

Name: private-reg-cred
Username: dock_user
Password: dock_password
Server: myprivateregistry.com:5000
Email: dock_user@myprivateregistry.com

kubectl create secret docker-registry private-reg-cred \
 --docker-server=myprivateregistry.com:5000 \
 --docker-username=dock_user \
 --docker-password=dock_password \
 --docker-email=dock_user@myprivateregistry.com

Configure the deployment to use credentials from the new secret to pull images from the private registry

k edit deploy web

spec:
imagePullSecrets: - name: private-reg-cred
containers: - image: myprivateregistry.com:5000/nginx:alpine
imagePullPolicy: IfNotPresent
name: nginx
resources: {}
terminationMessagePath: /dev/termination-log
terminationMessagePolicy: File
