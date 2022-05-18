# 介绍
- 公网nat服务器用于frp穿透串流moonlight, server端口不能用8个固定串流端口或一台server串流多个win时的解决方案
- 只需要任意1个公网tcp端口, 基于frp: server和client端各开6和9个内部端口即可, 除了client内部8串流端口固定, 其他端口随意
  - server
      - frps 监听: xx, 4001-4005 (win.frpc), 4000 (client.frpc)
      - frpc 访问: 4000
          - 转发: 8串流 → 4001-4005
  - win
      - frpc 访问: xx
          - 转发: 4001-4005 → 8串流
  - client
      - frps 监听: 1000, 8串流 (server.frpc)
      - frpc 访问: xx
          - 转发: 4000 → 4000
  - 启动顺序: server.frps, client.frps → win.frpc, client.frpc → server.frpc
  - 8个串流端口
      - TCP 47984, 47989, 48010
      - UDP 47998, 47999, 48000, 48002, 48010
  - 最终client上填写的串流ip: 127.0.0.1
- 文件基于 frp v0.42.0: https://github.com/fatedier/frp

# 首先: 进入配置文件修改为自己的ip和端口
1. 修改为自己的服务器公网tcp端口: config/server/frps.ini
2. 修改为自己的服务器ip+公网tcp端口: config/client/frpc.ini
3. 修改为自己的服务器ip+公网tcp端口: config/streaming/frpc.ini

# 然后: 进入当前目录执行步骤
0. 如果需要赋予权限: chmod 777 -R .
1. server: ./soft/linux_64/frps -c ./config/server/frps.ini
2. client: 3种机器
   - win: .\soft\win_64\frps -c .\config\client\frps.ini
   - mac-silicon: ./soft/mac_apple/frps -c ./config/client/frps.ini
   - mac-intel: ./soft/mac_intel/frps -c ./config/client/frps.ini
3. client: 3种机器
   - win: .\soft\win_64\frpc -c .\config\client\frpc.ini
   - mac-silicon: ./soft/mac_apple/frpc -c ./config/client/frpc.ini
   - mac-intel: ./soft/mac_intel/frpc -c ./config/client/frpc.ini
4. streaming(win): .\soft\win_64\frpc -c .\config\streaming\frpc.ini
5. server: ./soft/linux_64/frpc -c ./config/server/frpc.ini

# 安装 iperf3
- win: https://files.budman.pw
- mac:
  - /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  - brew install iperf3
- centos: yum install iperf3
- ubuntu: apt install iperf3

# 关闭串流端口进行 iperf3 测试
- streaming(win): .\iperf3 -s -p 48010
- client:
  - udp上传: iperf3 -c 127.0.0.1 -p 48010 -u -l 1k
  - udp下载: iperf3 -c 127.0.0.1 -p 48010 -u -l 1k -R
  - tcp上传: iperf3 -c 127.0.0.1 -p 48010
  - tcp下载: iperf3 -c 127.0.0.1 -p 48010 -R
