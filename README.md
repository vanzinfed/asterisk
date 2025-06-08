#!/bin/bash
set -e

echo "Обновляем систему и устанавливаем Asterisk..."
sudo apt update
sudo apt install -y asterisk

echo "Проверяем версию Asterisk:"
asterisk -V

echo "Останавливаем Asterisk перед настройкой..."
sudo systemctl stop asterisk || true

echo "Настраиваем /etc/asterisk/pjsip.conf..."
sudo tee /etc/asterisk/pjsip.conf > /dev/null <<'EOF'
[global]
type=global
user_agent=Asterisk PBX 22

[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0

[6001]
type=endpoint
context=internal
disallow=all
allow=ulaw
auth=6001
aors=6001

[6001]
type=auth
auth_type=userpass
password=6001
username=6001

[6001]
type=aor
max_contacts=1

[6002]
type=endpoint
context=internal
disallow=all
allow=ulaw
auth=6002
aors=6002

[6002]
type=auth
auth_type=userpass
password=6002
username=6002

[6002]
type=aor
max_contacts=1
EOF

echo "Настраиваем /etc/asterisk/extensions.conf..."
sudo tee /etc/asterisk/extensions.conf > /dev/null <<'EOF'
[internal]
exten => _600X,1,Dial(PJSIP/${EXTEN},20)
EOF

echo "Запускаем Asterisk..."
sudo systemctl start asterisk

echo "Перезагружаем конфигурацию Asterisk..."
sudo asterisk -rx "pjsip reload"
sudo asterisk -rx "dialplan reload"

echo "Показываем состояние SIP-эндоинтов:"
sudo asterisk -rx "pjsip show endpoints"

echo "Развёртывание завершено! Для управления Asterisk запускайте:"
echo "sudo asterisk -rvvv"
