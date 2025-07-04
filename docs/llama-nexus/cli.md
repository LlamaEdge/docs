---
sidebar_position: 6
---

# CLI Options

After installing the Llama-Nexus software, you can use the llama-nexus CLI to configure the setting.

The following are the CLI options.

```
Options:
      --config <CONFIG>
          Path to the config file [default: config.toml]
      --check-health
          Enable health check for downstream servers
      --check-health-interval <CHECK_HEALTH_INTERVAL>
          Health check interval for downstream servers in seconds [default: 60]
      --web-ui <WEB_UI>
          Root path for the Web UI files [default: chatbot-ui]
      --log-destination <LOG_DESTINATION>
          Log destination: "stdout", "file", or "both" [default: stdout]
      --log-file <LOG_FILE>
          Log file path (required when log_destination is "file" or "both")
  -h, --help
          Print help
  -V, --version
          Print version
```