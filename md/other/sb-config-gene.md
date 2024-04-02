# [Spring Boot]配置文件生成

## 背景

&ensp;&ensp;使用浏览器，根据规则生成Spring Boot的application.properties文件。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Spring Boot Config Generator</title>
</head>
<body>
    <h1>Spring Boot Config Generator</h1>
    
    <form id="configForm">
        <label for="serverPort">Server Port:</label>
        <input type="text" id="serverPort" name="serverPort" placeholder="Enter server port"><br>

        <label for="dbUrl">Database URL:</label>
        <input type="text" id="dbUrl" name="dbUrl" placeholder="Enter database URL"><br>

        <label for="dbUsername">Database Username:</label>
        <input type="text" id="dbUsername" name="dbUsername" placeholder="Enter database username"><br>

        <label for="dbPassword">Database Password:</label>
        <input type="password" id="dbPassword" name="dbPassword" placeholder="Enter database password"><br>

        <button type="button" onclick="generateConfig()">Generate Config</button>
    </form>

    <script>
        function generateConfig() {
            var serverPort = document.getElementById('serverPort').value;
            var dbUrl = document.getElementById('dbUrl').value;
            var dbUsername = document.getElementById('dbUsername').value;
            var dbPassword = document.getElementById('dbPassword').value;

            var configContent = 
                'server.port=' + serverPort + '\n' +
                'spring.datasource.url=' + dbUrl + '\n' +
                'spring.datasource.username=' + dbUsername + '\n' +
                'spring.datasource.password=' + dbPassword;

            // Create a Blob containing the file data
            var blob = new Blob([configContent], { type: 'text/plain' });

            // Create a download link and trigger the download
            var link = document.createElement('a');
            link.href = window.URL.createObjectURL(blob);
            link.download = 'application.properties';
            link.click();
        }
    </script>
</body>
</html>
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
