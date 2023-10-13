# http-ping-website
A website that can use http request to simulate ping command to get the latency.
# 时延网站报告
## 背景介绍
为满足客户可能在华为没有服务器节点的地区部署本地机器并连接到华为云的需求，需要获取从客户的本地机器上到各个节点的时延。与常规的拨测网站有所区别，该网站运行在客户本地机器上而不是获取各个节点之间拨测的时延，因此不受限于华为云拨测节点的部署地区。但因为不是由服务器进行时延测试，所以采用http ping的方式来模拟ping以获取时延。该方法获取的时延比用真正使用ICMP协议的ping稍大。
## 具体部署
### 时延网站机器部署
需要将以下html脚本部署到ECS上以供客户访问。部署时开放22端口（SSH）和20-21端口（FTP），部署后关闭。始终开放80、443、8080端口。
```shell
#安装httpd
sudo yum install httpd -y
#启动httpd
sudo systemctl start httpd
#设置开机启动httpd
sudo systemctl enable httpd
```
在本地机器上用scp上传html脚本（在最后的附录中有响应代码）到对应目录下（也可以用FTP等）：
```shell
#进入文件所在文件夹
cd xxx/xxxx/
#上传文件到服务器
scp ping.html username@remote_server:/var/www/html/
```
在浏览器中打开以验证：server_ip/ping.html

### 目的地机器部署
在对应地区购买ECS,并部署响应文件。
```shell
#安装httpd
sudo yum install httpd -y
#启动httpd
sudo systemctl start httpd
#设置开机启动httpd
sudo systemctl enable httpd
#创建响应文件
cd /var/www/html
vim ping
```
按下i进入编辑模式，输入“ok”，按下esc键退出编辑模式，输入:wq保存并退出。
在脚本中的sites列表中填入对应的目的地机器的信息即可。

#### 附：时延网站html脚本
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>HTTP Ping Site</title>
    <style>
        table {
            border-collapse: collapse;
            width: 100%;
        }

        table, th, td {
            border: 1px solid black;
        }

        th, td {
            padding: 8px 12px;
        }
    </style>
</head>
<body>
    <table>
        <thead>
            <tr>
                <th>Name</th>
                <th>Delay (ms)</th>
                <th>Average Delay (ms) (recent 10s)</th>
            </tr>
        </thead>
        <tbody id="pingTable">
            <!-- Dynamic rows will be added here -->
        </tbody>
    </table>

    <script>
        const sites = [
            {name: "Singapore", url: "http://119.13.108.243:80/ping", delays: [], coefficient: 1.0},
            {name: "Latin America-Santiago", url: "http://159.138.116.254:80/ping", delays: [], coefficient: 1.0},
            {name: "Middle East-Riyadh", url: "http://101.46.48.31:80/ping", delays: [], coefficient: 1.0},
            {name: "Afica-Johannesburg", url: "http://159.138.165.123:80/ping", delays: [], coefficient: 1.0},
            {name: "Russia", url: "http://159.138.205.132:80/ping", delays: [], coefficient: 1.0},

        ];

        const pingInterval = 1000; // 1 second
        const maxDelaysLength = 10; // 只保留最新的10个数据点

        sites.forEach(site => {
            let row = document.createElement('tr');
            let nameCell = document.createElement('td');
            let delayCell = document.createElement('td');
            let avgDelayCell = document.createElement('td');

            nameCell.textContent = site.name;
            delayCell.textContent = "Checking...";
            avgDelayCell.textContent = "Calculating...";

            row.appendChild(nameCell);
            row.appendChild(delayCell);
            row.appendChild(avgDelayCell);
            document.getElementById('pingTable').appendChild(row);

            setInterval(() => {
                ping(site, delayCell, avgDelayCell);
            }, pingInterval);
        });

        function ping(site, delayCell, avgDelayCell) {
            if (!navigator.onLine) {
                delayCell.textContent = "No Internet";
                avgDelayCell.textContent = "No Internet";
                return;
                }
            const img = new Image();
            const startTime = new Date().getTime();

            img.onload = img.onerror = function() {
                const rawDelay = new Date().getTime() - startTime;
                const delay = Math.round(rawDelay * site.coefficient);
                delayCell.textContent = delay + " ms";
                
                // Update average delay
                site.delays.push(delay);
                if (site.delays.length > maxDelaysLength) {
                    site.delays.shift(); // 移除最早的数据点
                }
                
                const totalDelays = site.delays.reduce((a, b) => a + b, 0);
                const average = Math.round(totalDelays / site.delays.length);
                avgDelayCell.textContent = average + " ms";
            }
            // var d = site.url;
            // d += site.url.fixed ? "" : "?" + Date.now().toString(36) + "=" + Math.random().toString(36).substr(2),
            // img.src = d;
            img.src = site.url + "?rand=" + Math.random();
        }
    </script>
</body>
</html>
```
