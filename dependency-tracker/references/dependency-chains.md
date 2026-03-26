## Common Dependency Chains

### Module rename/repurpose
```
config_file -> main app imports -> admin/management scripts
             -> scheduled task scripts -> monitoring/health checks
             -> documentation -> memory/knowledge base
```

### ID/routing change
```
config_file -> routing logic -> admin tools -> monitoring checks
```

### New env var
```
.env -> shell profile (source) -> systemd EnvironmentFile -> config files (if referenced)
     -> application code (os.getenv) -> CI/CD config
```

### File rename/move
```
old_path -> all imports -> startup scripts -> crontab -> systemd ExecStart
         -> documentation -> architecture docs -> memory files
```

### Cron job change
```
crontab entry -> monitoring schedule checks -> admin display
              -> documentation -> project docs
```
