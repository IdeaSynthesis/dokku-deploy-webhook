# dokku-deploy-webhook

The dokku-deploy-webhook plugin lets you invoke a remote URL immediately after every container deployment. The plugin performs a template replacement of the app name, internal IP address and internal port in the URL prior to the request. As an example, this can be used for notifying load balancers about new container instances.

## Installation

```sh
# dokku 0.5+
$ sudo dokku plugin:install https://github.com/IdeaSynthesis/dokku-deploy-webhook.git
```

### Upgrading from previous versions

```sh
# dokku 0.5+
$ sudo dokku plugin:update deploy-webhook
```

## Commands

```
$ dokku deploy-webhook:help
Usage: dokku deploy-webhook[:COMMAND]

Configure the deploy webhook behavior.
```

## Usage

Configure the deploy hook URL:

```
$ dokku config:set --no-restart myapp DOKKU_DEPLOY_WEBHOOK=http://hostname.tld/?IP=${IP_ADDRESS}&APP=${APP}&index=${index}&IMAGE=${IMAGE}&initial=true
-----> Setting config vars
       DOKKU_DEPLOY_WEBHOOK: http://hostname.tld/?IP=${IP_ADDRESS}&APP=${APP} 

## Configuration
`dokku-deploy-webhook` uses the [Dokku environment variable manager](http://dokku.viewdocs.io/dokku/configuration-management/) for all configuration. The important environment variables are:

Variable                   | Default     | Description
---------------------------|-------------|-------------------------------------------------------------------------
`DOKKU_DEPLOY_WEBHOOK`     | (none)      | **REQUIRED:** The URL to be called when a container deployment occurs.
`DOKKU_DEPLOY_WEBHOOK_METHOD`      | (none)      | **OPTIONAL:** The HTTP method to use: defaults to GET. 


## Design

`dokku-deploy-webhook` was built to allow a container deployment to cleanly notify a load balancer on a different machine about changes to the local container IPs caused by restarts.

For every container started with the ***web*** process type, the webhook will be invoked. Each invocation will be done with the following template parameters replaced in the URL:
- **${APP}**: the app the container is for.
- **${IP}**: the container's IP address that the load balancer can use to communicate with the container
- **${PORT}**: the listening port that the load balancer can use to communicate with the container
- **${IMAGE}**: the container image

All request URLs will include an **index** request parameter: this is the container index and can be used by the webhook receiver to differentiate between containers.
The very first request will include the **initial** request parameter, with the value set to ***true***: this can be used to clear the load balancers state.

## License

This plugin is released under the MIT license. See the file [LICENSE](LICENSE).
