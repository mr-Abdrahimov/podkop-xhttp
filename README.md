
# 📖 Xray + VLESS+XHTTP+REALITY на OpenWrt 24.10.4 с Podkop

**Полная инструкция от установки до работы**  
*OpenWrt 24.10.4 + Podkop уже установлен*

## 📦 1. Установка пакетов

```bash
opkg update
opkg install xray-core nano
```

## ⚙️ 2. Конфигурация Xray
Открываем https://mr-abdrahimov.github.io/podkop-xhttp/vless-to-xray-generator.html и создаём конфиг для Xray
Далее в роутере редактируем файл и вставляем наш сгенерированный конфиг
```bash
nano /etc/xray/config.json
```
Вставляем конфиг который был сгененрирован

## ✅ 3. Проверка конфига

```bash
xray -test -config /etc/xray/config.json
```
```
Configuration OK.
```

## 🚀 4. Автозапуск

```bash
cat > /etc/init.d/xray << 'EOF'
#!/bin/sh /etc/rc.common

START=99
USE_PROCD=1
PROG=/usr/bin/xray

validate_config() {
    $PROG -test -config /etc/xray/config.json >/dev/null 2>&1
}

start_service() {
    validate_config || {
        echo "Xray: invalid config"
        return 1
    }
    procd_open_instance
    procd_set_param command $PROG -config /etc/xray/config.json
    procd_set_param respawn 60 5 5
    procd_set_param user root
    procd_set_param stdout 1
    procd_set_param stderr 1
    procd_close_instance
}
EOF

chmod +x /etc/init.d/xray
/etc/init.d/xray enable
/etc/init.d/xray start
```

## 🧪 5. Тест Xray

```bash
/etc/init.d/xray status
curl --socks5 127.0.0.1:10808 https://ifconfig.me
```
```
✅ running
✅ IP сервера (не роутера!)
```

## 🔗 6. Подключение к Podkop

### JSON (Outbound Configuration)
```json
{
  "type": "socks",
  "tag": "vless-xhttp",
  "server": "127.0.0.1",
  "server_port": 10808
}
```

## 🎯 7. Маршрутизация в Podkop

**Geosite / Geoblock / Custom Rules:**
```
Outbound: vless-xhttp
```

## 📊 8. Мониторинг

```bash
/etc/init.d/xray status          # Статус
logread | grep xray | tail -10   # Логи
logread -f | grep xray          # Живые логи
curl --socks5 127.0.0.1:10808 https://ifconfig.me  # Финальный тест
```

## ✅ Готово!

**Xray: `127.0.0.1:10808` → Podkop → Твоя маршрутизация**

| Метрика | Значение |
|---------|----------|
| **Время установки** | 5 минут |
| **Размер** | ~10 МБ |
| **Совместимость** | OpenWrt 24.10.4 + Podkop |
| **Протокол** | VLESS+XHTTP+REALITY |

**🎉 Туннель стабильно работает!**
