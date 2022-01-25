# note
都是一些Java学习类的笔记





VSCode配置Java的setting.json

```json
{

    "workbench.colorTheme": "Visual Studio Light",
    
    "git.autofetch": true,
    
    "java.semanticHighlighting.enabled": true,
    
    "files.exclude": {
    
    "**/.classpath": true,
    
    "**/.project": true,
    
    "**/.settings": true,
    
    "**/.factorypath": true
    
    },
    
    "editor.suggestSelection": "first",
    
    "vsintellicode.modify.editor.suggestSelection": "automaticallyOverrodeDefaultValue",
    
    "java.configuration.maven.userSettings": "D:\\java\\apache-maven-3.8.1\\conf\\settings.xml",
    
    "maven.executable.path": "D:\\java\\apache-maven-3.8.1\\bin\\mvn.cmd",
    
    "maven.terminal.useJavaHome": true,
    
    "maven.terminal.customEnv": [
    
    {
    
    "environmentVariable": "JAVA_HOME",
    
    "value": "D:\\java\\jdk\\jdk8"
    
    }
    
    ],
    
    "explorer.confirmDelete": false,
    
    "maven.executable.preferMavenWrapper": false,
    
    "python.languageServer": "Microsoft",
    
    "java.requirements.JDK11Warning": false,
    
    "java.maven.downloadSources": true,
    
    "java.maven.updateSnapshots": true,
    
    "maven.excludedFolders": [
    
    "**/.*",
    
    "**/node_modules",
    
    "**/target",
    
    "**/bin"
    
    ],
    
    }
```

