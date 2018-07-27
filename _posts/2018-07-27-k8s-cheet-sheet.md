# Kubernetes 명령어 모음


1. POD 정보 보기
```
  kubectl get pods -n [namespace명] --selector='[selector명]'
```
```
ex) kubectl get pods -n joyoungc-dev --selector='app=joyoungc-spring'
```

2. 실시간 로그 보기 (tail)
```
  kubectl logs -n [namespace명] $(kubectl get pods -n [namespace명] --selector='[selector명]' | grep -v 'NAME' | awk '{print $1}') --tail=100 -f
```
```
ex) kubectl logs -n joyoungc-dev $(kubectl get pods -n joyoungc-dev --selector='app=joyoungc-spring' | grep -v 'NAME' | awk '{print $1}') --tail=100 -f
```

3. GREP 하기
```
  kubectl logs -n [namespace명] --selector='[selector명]' | grep '[찾고자 하는 문자열]'
``` 
```
ex) kubectl logs -n joyoungc-stg --selector='app=joyoungc-spring' --since=1h | grep 'EXCEPTION'
※ --since=2h : 2시간 전 데이터에 한해 검색 (5s, 2m, or 3h. Defaults to all logs.)
```
