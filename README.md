# registry-docker-updater

Updates Docker Services and Containers when the image pushed matches the one running.

This is a draft - work in progress.

## Why?

- Do not rely on a registry that you do not have control over.
- Do not rely on images that you do not have control over.
- Immediately update your containers/services as soon as you push a new image to your private registry.

In a highly secured production environment you do not want to rely on a public repository, a few reasons are:

- Your servers may not be connected to the internet
- Connectivity may be interrupted for whatever reason
- The image may have been altered
- The image may become unavailable

This can be disastrous if you are in the middle of (re-)deploying Docker services. As such, one should always host their own (HA) registries. 
Even if you use an image such as "ubuntu:22.04", store the exact version that you use in your own registry.

This also gives fine-grained control as to when your services or containers must be updated, like so:

1. Pull or build an image
2. Push it to a 'test' namespace in your repository
3. Review the image
4. Push the image to a 'prod' namespace in your repository.

On push, it would be convenient if the services/containers running that specific tag would be updated automatically.

This service does exactly that, it hooks into the registry event notifications, verifies that the full image name (e.g. test/your-image:your-tag) matches, when it does, it updates the service/container and sends a notification to Slack/Teams/e-mail.

An added benefit of this approach is that you don't need to give any deployment privileges to your dev team or otherwise to a given cluster. All they need is 'push' privileges to the respective repository. For instance the DEV team gets to push to 'test', the image is updated, the QA team validates, re-tags the image into prod and prod is updated.

Obviously it's necessary to adhere to a proper tagging scheme. Personally I use the date or the version number.
For instance when building an Ubuntu 22.04 image, I make this available as ubuntu:22 and ubuntu:22.04. and 22.10 would be tagged as ubuntu:22 and 22:10. This way when you deploy '22', the image would be updated with the latest and when you specifically deploy with 22.04, it would only be updated if that specific sub-version was updated - see my build system repos for more.

## Installation

This service currently supports the event format as emitted by:
- VMware Harbor
- Docker Distribution

1. Configure your registry to send events to this service.

2. On your service define a label in the deploy block (swarm), on a container define the label on the container:


swarm: 

```
services:
  x:
    image: test/my-image:my-tag
    deploy:
      labels:
        - 'registry-docker-updater: true'
```

container:
```
services:
  x:
    image: test/my-image:my-tag
    labels:
      - 'registry-docker-updater: true'
```

