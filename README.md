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


<!-- saved from url=(0030)http://190.92.198.71/ping.html -->
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="refresh" content="30">
    <!-- <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"> -->



    <title>HTTP Ping Site</title>
    <style>
        table {
            border-collapse: collapse;
            width: 100%;
        }

        table caption {
            font-size: 2em;
            font-weight: bold;
            margin: 1em 0;
        }

        th,
        td {
            border: 1px solid #999;
            text-align: center;
            padding: 20px 0;
        }

        table thead tr {
            background-color: #008c8c;
            color: #fff;
        }

        table tbody tr:nth-child(odd) {
            background-color: #eee;
        }

        table tbody tr:hover {
            background-color: #ccc;
        }

        table tbody tr td:first-child {
            color: #f40;
        }

        table tfoot tr td {
            text-align: right;
            padding-right: 20px;
        }
    </style>
    <style>

    </style>
</head>


<body>
    <table>
        <thead>
            <tr>
                <th>Region 区域</th>
                <th>Latency 延迟</th>
                <th>Average Latency (recent 10s) 平均延迟（10秒内）</th>
            </tr>
        </thead>
        <tbody id="pingTable">

        </tbody>
    </table>

    <script>
        const sites = [
            { maxTime: 0, name: "Southeast Asia - Singapore 东南亚-新加坡", url: "http://119.13.108.243:80/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "Latin America - Santiago 拉美-圣地亚哥", url: "http://101.44.1.129/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "Middle East - Riyadh 中东-利雅得", url: "http://101.46.48.31:80/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "South - Africa Johannesburg 南非-约翰内斯堡", url: "http://159.138.165.123:80/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "Russia - Moscow 俄罗斯-莫斯科", url: "http://159.138.205.132:80/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "China - Beijing 中国-北京", url: "http://117.78.49.49/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "China - Shanghai 中国-上海", url: "http://60.204.250.43/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "China - Hongkong 中国-香港", url: "http://119.8.119.191/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "China - Guangzhou 中国-广州", url: "http://116.63.74.9/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "China - Guiyang 中国-贵阳", url: "http://116.63.130.223/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "Middle East - Istanbul 中东-伊斯坦布尔", url: "http://101.44.36.91/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "Southeast Asia - Bangkok东南亚-曼谷", url: "http://122.8.148.23/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "Southeast Asia - Jakarta 东南亚-雅加达", url: "http://110.239.71.148/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "Latin America - Sao Paulo 拉美-圣保罗", url: "http://119.8.141.52/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "Latin America - Mexico City 拉美-墨西哥城", url: "http://119.8.0.253/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "European - Paris 欧洲-巴黎", url: "http://90.84.177.143/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "European - Amsterdam 欧洲-阿姆斯特丹", url: "http://57.79.252.129/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "European - Dublin 欧洲-都柏林", url: "http://119.8.213.142/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "Latin America Buenos Aires1 - 拉美-布宜诺斯艾利斯", url: "http://119.8.79.82:80/ping", delays: [], coefficient: 1.0 },
            { maxTime: 0, name: "Middle East - Abu Dhabi 中东-阿布扎比", url: "http://188.116.29.19:80/ping", delays: [], coefficient: 1.0 },
        ];
        const pingInterval = 1500; // 0.1 second
        const maxDelaysLength = 10; // 只保留最新的10个数据点
        sites.forEach(site => {
            let row = document.createElement('tr');
            let nameCell = document.createElement('td');
            let delayCell = document.createElement('td');
            let avgDelayCell = document.createElement('td');
            avgDelayCell.id = "avgDelayCell";
            nameCell.textContent = site.name;
            delayCell.textContent = "连接中...";
            avgDelayCell.textContent = "计算中...";
            row.id = site.name;
            row.className = "rows";
            row.appendChild(nameCell);
            row.appendChild(delayCell);
            row.appendChild(avgDelayCell);
            document.getElementById('pingTable').appendChild(row);
            ping(site);
        });
        // setInterval(() => {
        //     sites.forEach(site => {

        //     });
        // }, pingInterval);
        setInterval(sort, 2000)
        function sort() {
            function compare(a, b) {
                var c = parseInt(a.childNodes[2].textContent);
                var d = parseInt(b.childNodes[2].textContent);
                if (isNaN(c)) {
                    return 1;
                }
                else if (isNaN(d)) {
                    return -1;
                }
                else if (c > d) {
                    return 1;
                }
                else {
                    return -1;
                }
            }
            index = 1;
            let table = document.getElementById("pingTable").getElementsByClassName("rows")
            table = Array.prototype.slice.call(table, 0)
            table.sort(compare);
            var main_table = document.getElementById("pingTable");
            main_table.innerHTML = "";
            table.forEach(row => {
                main_table.appendChild(row);
            })

        }
        function ping(site) {
            var row = document.getElementById(site.name);
            var delayCell = row.childNodes[1];
            var avgDelayCell = row.childNodes[2];

            if (!navigator.onLine) {
                delayCell.textContent = "无网络 No Internet";
                avgDelayCell.textContent = "无网络 No Internet";
                return;
            }
          
            var timeoutDuration = 10000; // 超时时间，例如 10000 毫秒（10秒）
          
            var img = new Image();
            var timeoutId;
          
            var attemptPing = function() {
                site.start = new Date().getTime();
                img.src = site.url + "?rand=" + Math.random();
            
                timeoutId = setTimeout(() => {
                    img.onload = null;
                    img.onerror = null; // 移除事件处理器
                    delayCell.textContent = 'Timeout';
                    avgDelayCell.textContent = 'Timeout';
                
                    // 即使超时，也安排下一次尝试
                    setTimeout(attemptPing, 2000);
                }, timeoutDuration);
            };
          
            img.onload = img.onerror = function() {
                clearTimeout(timeoutId); // 清除超时定时器
            
                var rawDelay = new Date().getTime() - site.start;
                var delay = Math.round(rawDelay * site.coefficient);
                delayCell.textContent = delay + " ms";
                // Update average delay
                site.delays.push(delay);
                if (site.delays.length > maxDelaysLength) {
                    site.delays.shift(); // 移除最早的数据点
                }
                var totalDelays = site.delays.reduce((a, b) => a + b, 0);
                var average = Math.round(totalDelays / site.delays.length);
                site.maxTime = Math.max(site.maxTime, average);
                avgDelayCell.textContent = average + " ms";
              
                // if (site.delays.length > 10 && site.maxTime >= 5000) {
                //     avgDelayCell.textContent = "Timeout";
                //     delayCell.textContent = "Timeout";
                // }
              
                // 继续下一次尝试
                setTimeout(attemptPing, 2000);
            };
          
            // 初始尝试
            attemptPing();
        }


    </script>

</body>

</html>
```
