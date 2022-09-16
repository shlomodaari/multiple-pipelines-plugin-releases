RunMultiplePipelines plugin adds a new custom stage to Spinnaker to trigger multiple child pipelines based on a yaml config.

### Version Compatibility
Minimum tested version is 2.26.x

| Plugin  | Spinnaker Platform |
|:------------- | :--------- |
| 1.0.2 <= |  2.27.x |
| 1.0.3 <= |  2.27.x |
| 1.0.4 <= |  2.27.x |

NOTE: The plugin is not actively tested in all compatible versions with all variants, but is expected to work in the above.

Check [release notes](https://github.com/armory/multiple-pipelines-plugin-releases#release-notes) for more info

# Installation & Configuration
Note: alternative you can consume the plugin using this example patch as well. [example](https://github.com/armory/multiple-pipelines-plugin-releases/blob/main/plugins/custom/patch-plugin-multiple-pipelines-2.yml)
1. Identify the released version of the RunMultiplePipelines plugin you wish to install. Official releases are found [here](https://github.com/armory/multiple-pipelines-plugin-releases).
2. In your Spinnaker configuration, add the following repository & plugin configuration. [example](https://github.com/armory/multiple-pipelines-plugin-releases/blob/main/plugins/custom/patch-plugin-multiple-pipelines.yml)
```yaml
spinnakerConfig:
  # spec.spinnakerConfig.profiles - This section contains the YAML of each service's profile
  profiles:
    gate:
      spinnaker:
        extensibility:
          deck-proxy: # you need this for plugins with a Deck component
            enabled: true
            plugins:
              Armory.RunMultiplePipelines:
                enabled: true
                version: 1.0.4
          repositories:
            runMultiplePipelinesRepo:
              url: https://raw.githubusercontent.com/armory/multiple-pipelines-plugin-releases/main/plugins.json
    orca:
      spinnaker:
        extensibility:
          plugins:
            Armory.RunMultiplePipelines:
            enabled: true
            version: 1.0.4
            extensions:
              armory.runMultiplePipelinesStage:
                enabled: true
          repositories:
            runMultiplePipelinesRepo:
            url: https://raw.githubusercontent.com/armory/multiple-pipelines-plugin-releases/main/plugins.json
```
3. On startup, the plugin will be downloaded and installed to the appropriate Spinnaker services.

- if you don't specify a version it will take the newest.

- after redeploying spinnaker this will load and start the plugin during app startup for gate and orca.

The custom stage consumes a yaml under yaml-config with the next format:
```yaml
bundle_web: 
  app_name: 
    arguments: 
      - object
      - object
      - object
      - object
    child_pipelines: string
    depends_on: 
      - string
      - string
  rollback_onfalure: boolean
```
Example:
```yaml
bundle_web:
  appName2:
    arguments:
      app: appName2
      deploymentFrezeOverride: true
      skipCanary: true
      tag: "1.1.0"
      targetEnv: targetEnv
    child_pipeline: childPipeline
  appName1:
    arguments:
      app: appName1
      deploymentFrezeOverride: true
      skipCanary: true
      tag: "1.1.0"
      targetEnv: targetEnv
    child_pipeline: childPipeline
```

## yaml config specifics
- The bundle name needs to be bundle_web
- All apps need an argument app with the same name example:
```
  appName1:
    arguments:
      app: appName1
  ```
- The child_pipeline name needs to exist on the same spinnaker application
- The Deploy (manifest) stage you wish to rollback in your child_pipeline has to have the prefix "Deploy"
- The plugin will look a created artifact name that includes the app argument
- For rollback_onfailure or manual rollbacks to work you need to create a pipeline with the name rollbackOnFailure in the same application

### release notes
- Version 1.0.2 only supports concurrent executions using Redis
- Version 1.0.3 supports concurrent executions using SQL database
- Version 1.0.4:
    - Breaking change don't use depends_on property 
    - The plugin used to work in one task and had a loop to control the order of executions but this had a bug where the "main" task on previous versions restarted itself after 10 min
    - Now the plugin stage uses three tasks - the last task monitor child executions
