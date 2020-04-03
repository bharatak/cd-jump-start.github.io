---
title: Configuration handling best practices
tags: [microservices, configuration]
last_updated: April 3, 2020
keywords: configuration, config, externalised config, central config
summary: "Configurations for services specificy initial settings and parameters for their runtime behaviour. These could be name-value pairs or more refined configuration settings with section and sub-section goruping. Config values could be specified in a file, passed via command line, read from environment variables,retrieved via a config service or other similar sources provided by the runtime framework or environment. They could be specified in a variety of formats including yaml, json, xml, and application property files."
sidebar: mydoc_sidebar
permalink: configuration.html
folder: mydoc

### Considerations
- Configuration values will be different in different environments
- Within one environment itself the values will need changes at runtime
- It should be possible to change the config values without a code update
- Service should assume exact location of config files; this should be supplied by the hosting runtime 
- It is better if the service is able to use the changed values without a restart

### Best Practices
- Read a config file from a path supplied at run time via command line or environment variables
- Read config values from environment variables
- Read config values from command line
- Read config values from a configuraton service
- Listen to config changes or provide an endpoint to be invoked when config changes


---
{% include links.html %}