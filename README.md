# soar-web-vm

SOAR-ELK-ipset 프로젝트의 Web VM(센서/수집) 설정 저장소입니다.
담당 컴포넌트: Nginx(WAF) · Suricata · Filebeat · Logstash

## 네트워크 세그먼트

| 구간 | 대역 | 연결 |
|---|---|---|
| LAN0 | 192.168.10.0/24 | kali ↔ web |
| LAN1 | 192.168.20.0/24 | web ↔ was |
| LAN3 | 192.168.40.0/24 | 전 VM ↔ soc |

web VM NIC: LAN0(.10.10) / LAN1(.20.10) / LAN3(.40.10)

## 디렉토리 구조

```
web-vm-config/
├── docker-compose.yml
├── nginx/
│   └── sites-available/default
├── suricata/
│   ├── suricata.yaml
│   ├── classification.config
│   ├── reference.config
│   └── threshold.config
├── filebeat/
│   ├── filebeat.yml
│   └── modules.d/
└── logstash/
    ├── conf.d/pipeline.conf
    ├── logstash.yml
    └── pipelines.yml
```

## 설정 내용

- **nginx**: `owasp/modsecurity-crs:nginx` 이미지 기반, ModSecurity + OWASP CRS 룰 적용, 80번 포트로 서비스
- **suricata**: host에 직접 설치 (컨테이너화 안 함), eve.json으로 탐지 로그 출력
- **filebeat**: Suricata 로그(파일 기반) + Nginx/ModSecurity 로그(Docker autodiscover로 컨테이너 stdout 수집) 두 경로 모두 logstash로 전달
- **logstash**: filebeat로부터 로그 수신, 파이프라인 처리 후 Elasticsearch로 전달 (현재는 soc VM 미연결 상태)
- 전체 컨테이너는 `docker-compose.yml`로 정의, `docker compose up -d` 한 번에 기동
