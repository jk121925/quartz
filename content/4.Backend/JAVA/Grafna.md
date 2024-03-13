```
sudo apt-get install -y adduser libfontconfig1
# grafana 다운로드 
wget https://dl.grafana.com/oss/release/grafana_7.5.2_amd64.deb
# grafana 설치 
sudo dpkg -i grafana_7.5.2_amd64.deb
# 실행
sudo service grafana-server start
# 실행 상태 확인 
sudo service grafana-server status
```
   