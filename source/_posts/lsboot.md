---
title: logstash 启动脚本
date: 2020-12-09 14:42:09
tags: Logstash
categories:
---

- logstash 通用启动脚本

```bash
#!/bin/bash
# FileName: lsboot
# Description: 启动logstash实例
# Dispatcher:
# Description: Usage: lsboot xxx.conf
# CreateDate: 2020-12-06

if [ $# -lt 1 ];then
	echo -e "  \e[033m Usage: $0 [FileName: e.g.: xxx.conf OR path/xxx.conf] \e[0m"
	exit 1
fi

#config file OR path
conFile=$1
conFilePath=$(cd `dirname $conFile`;pwd)
appName=$(basename ${conFile} | sed s/.conf.*//)
currPath=$(cd `dirname $0`;pwd)

LS_HOME=${LS_HOME:-$currPath}
cd $LS_HOME

check_pid(){
  pids=`ps -ef | grep -v grep | grep -w "\-f ${conFile}" | awk '{print $2}'`
  #echo "PID : ${pids}"
}

[[ $2 == "test" ]] && appName=test && rm -rf data/test && rm -rf logs/test

#./stop.sh

#sleep 1

check_pid
cmd="bin/logstash -f ${conFile} -l logs/${appName} --path.data data/${appName} -n ${appName}"

if [ $pids ];then
  echo " APP $appName was running on :"
  echo -e "    \e[032m 【 `pwdx $pids` 】\e[0m"
  echo " If restart, kill $pids first."
  exit 0
else
  cd $LS_HOME
  echo "$cmd"
  if [ ${appName} == "test" ];then
    $cmd
  else
    eval $cmd &
    sleep 1
    check_pid
    [[ -n "${pids}" ]] && echo -e "\e[032m success \e[0m: $appName boot on: $pids" || echo -e "  \e[31m [ $appName : $conFile ] Boot Failed. \e[0m "
  fi
fi

#sleep 1

#ps -ef | grep -v grep | grep -w "${conFile}" | awk '{print $2"\t"$NF}' 

exit 0

```



- logstash将时间转换成unix时间戳

```ruby
# https://zerlong.com/886.html
ruby { code => 'event.set("unix_ts",(event.get("@timestamp").to_f.round(3)*1000).to_i)' }
```

