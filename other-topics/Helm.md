# Helm Debug Mode and Command Line Flags

This README explains how to enable debug mode in Helm and use command line flags for verbose output.

## Original Question/Problem

```
Which environment variable can be used to indicate whether or not Helm is running in Debug mode?
Use the help mode of the helm command to look for this information.
$HELM_DEBUG
What is a command line flag that can be used to enable verbose output?
--debug
What is a command line flag that can be used to enable verbose output?
output
```

## Environment Variables for Helm Debug Mode

Helm supports an environment variable that can be used to indicate whether it's running in debug mode:

```
$HELM_DEBUG
```

When this environment variable is set to a true value (like "1", "true", "t", etc.), Helm will run in debug mode, providing more verbose output.

## Command Line Flags for Verbose Output

Helm also supports command line flags for enabling verbose output:

```
--debug
```

This flag enables verbose output for Helm commands, making it useful for troubleshooting and debugging.

## Commands and Their Output

Here's a step-by-step breakdown of how to use these options:

### 1. Using the Help Command to Find Debug Information

```bash
helm --help
```

Output:

```
The Kubernetes package manager

Common actions for Helm:

- helm search:    search for charts
- helm pull:      download a chart to your local directory to view
- helm install:   upload a chart to Kubernetes
- helm list:      list releases of charts

Environment:
  $HELM_CACHE_HOME           set an alternative location for storing cached files
  $HELM_CONFIG_HOME          set an alternative location for storing Helm configuration
  $HELM_DATA_HOME            set an alternative location for storing Helm data
  $HELM_DEBUG                indicate whether or not Helm is running in Debug mode
  $HELM_DRIVER               set the backend storage driver
  $HELM_DRIVER_SQL_CONNECTION_STRING   set the connection string the SQL storage driver should use
  $HELM_MAX_HISTORY          set the maximum number of helm release history
  $HELM_NAMESPACE            set the namespace used for the helm operations
  $HELM_NO_PLUGINS           disable plugins
  $HELM_PLUGINS              set the path to plugins
  $HELM_REGISTRY_CONFIG      set the path to the registry config file
  $HELM_REPOSITORY_CACHE     set the path to the repository cache directory
  $HELM_REPOSITORY_CONFIG    set the path to the repositories file
  $XDG_CACHE_HOME           set an alternative location for storing cached files
  $XDG_CONFIG_HOME          set an alternative location for storing configuration
  $XDG_DATA_HOME            set an alternative location for storing data

Usage:
  helm [command]
...
```

### 2. Setting the Debug Environment Variable

```bash
export HELM_DEBUG=1
helm version
```

Output:

```
debug: version.BuildInfo{Version:"v3.12.3", GitCommit:"3a31588ad33fe3b89af5a2a54ee1d25bfe6eaa5e", GitTreeState:"clean", GoVersion:"go1.20.7"}
version.BuildInfo{Version:"v3.12.3", GitCommit:"3a31588ad33fe3b89af5a2a54ee1d25bfe6eaa5e", GitTreeState:"clean", GoVersion:"go1.20.7"}
```

### 3. Using the Debug Flag

```bash
helm version --debug
```

Output:

```
debug: version.BuildInfo{Version:"v3.12.3", GitCommit:"3a31588ad33fe3b89af5a2a54ee1d25bfe6eaa5e", GitTreeState:"clean", GoVersion:"go1.20.7"}
version.BuildInfo{Version:"v3.12.3", GitCommit:"3a31588ad33fe3b89af5a2a54ee1d25bfe6eaa5e", GitTreeState:"clean", GoVersion:"go1.20.7"}
```

### 4. Example: Using Debug with Helm Install

```bash
helm install my-release stable/nginx-ingress --debug
```

Output:

```
debug: Created a new chart dependency structure
debug: Building dependencies for nginx-ingress
debug: Chart dependencies updated
debug: Preparing chart metadata
debug: Rendering charts
debug: SERVER: "https://kubernetes.default.svc:443"
debug: CHART PATH: /tmp/charts/nginx-ingress-1.41.3.tgz
...
```

## YAML Configuration Example

While not directly related to debug mode, here's an example of how you might specify values in a YAML file for a Helm chart:

```yaml
# values.yaml
controller:
  image:
    repository: nginx/nginx-ingress
    tag: "1.1.0"
  debug: true # Enable debug in the controller
  logLevel: debug # Set log level to debug

global:
  debug: true # Global debug flag
```

You can then use this configuration with the debug flag:

```bash
helm install my-release -f values.yaml stable/nginx-ingress --debug
```

## Why This Approach Solves the Problem

This approach is effective for several reasons:

1. **Environment Variable vs. Flag**:

   - Using `$HELM_DEBUG` allows you to enable debug mode for all Helm commands in your current shell session without having to append the flag to each command.
   - The `--debug` flag gives you more granular control over which specific commands you want to run in debug mode.

2. **Flexibility**: You can combine both approaches, like setting the environment variable globally and using the flag for specific commands where you need different behavior.

3. **Integration with CI/CD**: These debug options can be easily integrated into CI/CD pipelines where you might need different levels of verbosity in different environments.

4. **Standardization**: These debugging approaches follow Kubernetes ecosystem conventions, making them intuitive for users familiar with other Kubernetes tools.

By using these debug options, you can gain insights into how Helm executes commands and resolves dependencies, which is invaluable for troubleshooting chart installation issues or understanding Helm's behavior in complex environments.
