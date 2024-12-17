# grpcos
gRPC OS - Dokumentacja

# gRPC OS - Dokumentacja

## Spis treści
1. [Wprowadzenie](#wprowadzenie)
2. [Instalacja](#instalacja)
3. [Konfiguracja](#konfiguracja)
4. [Shell](#shell)
5. [Zarządzanie procesami](#zarządzanie-procesami)
6. [Dodawanie nowych procesów](#dodawanie-nowych-procesów)
7. [API gRPC](#api-grpc)

## Wprowadzenie

gRPC OS to system operacyjny bazujący na protokole gRPC, oferujący:
- Zarządzanie procesami
- System plików oparty na obiektach
- Zarządzanie domenami
- System autoryzacji
- Interaktywny shell

## Instalacja


Uruchomienie systemu:

```bash
# Konfiguracja środowiska
cp .env.example .env
nano .env  # Dostosuj zmienne środowiskowe

# Uruchomienie kontenerów
docker-compose up -d
```

## Konfiguracja

Podstawowa konfiguracja w `.env`:

```bash
# Konfiguracja bazy danych
DB_HOST=localhost
DB_PORT=3306
DB_NAME=grpc_os
DB_USER=grpc_user
DB_PASSWORD=secure_password

# Konfiguracja systemu
GRPC_PORT=50051
JWT_SECRET=your_secret_key
DEBUG=false
```

2. Konfiguracja domen:
   
```bash
# /etc/grpc-os/domains.conf
domain example.com {
    process_id: "proc_123"
    port: 8080
}

domain api.example.com {
    process_id: "proc_456"
    port: 8081
}
```

## Shell

# Przykłady użycia gRPC OS Shell

## 1. Podstawowe operacje

### Logowanie i zarządzanie sesją

```bash
# Logowanie do systemu
grpc-os> login admin secure_password

# Sprawdzenie aktualnego użytkownika
grpc-os> whoami

# Wylogowanie
grpc-os> logout
```

### Zarządzanie procesami

```bash
# Lista procesów
grpc-os> ps

# Szczegóły procesu
grpc-os> ps -l proc_123

# Uruchomienie nowego procesu
grpc-os> start web-server "python app.py" PORT=8080 DEBUG=true

# Zatrzymanie procesu
grpc-os> kill proc_123

# Restart procesu
grpc-os> restart proc_123
```

### Operacje na obiektach

```shell
# Lista obiektów
grpc-os> ls /
grpc-os> ls /config

# Tworzenie obiektu
grpc-os> touch config.json '{"port": 8080}'

# Odczyt obiektu
grpc-os> cat obj_123

# Usunięcie obiektu
grpc-os> rm obj_123
```

### Zarządzanie domenami

```bash
# Lista domen
grpc-os> domains

# Przypisanie procesu do domeny
grpc-os> assign example.com proc_123 8080

# Status domeny
grpc-os> domain-status example.com

# Usunięcie przypisania
grpc-os> unassign example.com
```



## Zarządzanie procesami

1. Struktura procesu:
   
```python
class Process:
    id: str           # Unikalny identyfikator
    name: str         # Nazwa procesu
    command: str      # Polecenie do uruchomienia
    environment: Dict # Zmienne środowiskowe
    status: str       # Stan procesu
```

2. Operacje na procesach:
   
```python
# Tworzenie procesu
process = await client.create_process(
    name="web-server",
    command="python server.py",
    environment={"PORT": "8080"}
)

# Zatrzymanie procesu
await client.kill_process(process.id)

# Status procesu
status = await client.get_process_status(process.id)
```


## Dodawanie nowych procesów

1. Struktura katalogu procesu:
```
processes/
├── myapp/
│   ├── Dockerfile
│   ├── app.py
│   └── config.yml
└── process.yml
```

2. Konfiguracja procesu (process.yml):
   
```yaml
name: myapp
version: 1.0
command: python app.py
environment:
  PORT: 8080
  DEBUG: false
domains:
  - myapp.example.com
volumes:
  - data:/app/data
```

3. Rejestracja procesu:
   
```bash
grpc-os register-process ./processes/myapp
```

## API gRPC

1. Połączenie z systemem:
```python
import grpc
from grpc_os import client

# Utworzenie klienta
channel = grpc.insecure_channel('localhost:50051')
client = GrpcOSClient(channel)

# Autoryzacja
token = await client.login("username", "password")
metadata = [('authorization', f'Bearer {token}')]
```

2. Przykłady użycia API:
```python
# Zarządzanie procesami
process = await client.create_process(
    name="myapp",
    command="python app.py",
    environment={"PORT": "8080"}
)

# Zarządzanie obiektami
obj = await client.create_object(
    name="config.json",
    data=b'{"key": "value"}',
    metadata={"type": "config"}
)

# Zarządzanie domenami
await client.assign_process_to_domain(
    domain="myapp.example.com",
    process_id=process.id,
    port=8080
)
```

3. Obsługa błędów:
```python
try:
    response = await client.some_operation()
except grpc.RpcError as e:
    if e.code() == grpc.StatusCode.UNAUTHENTICATED:
        print("Unauthorized")
    elif e.code() == grpc.StatusCode.NOT_FOUND:
        print("Not found")
    else:
        print(f"Error: {e.details()}")
```


# Struktura projektu gRPC OS

```
grpc-os/
├── src/
│   ├── __init__.py
│   ├── server.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── filesystem.py
│   │   ├── process_manager.py
│   │   ├── memory_manager.py
│   │   ├── dns_manager.py
│   │   ├── nameserver_manager.py
│   │   ├── auth_manager.py
│   │   ├── backup_manager.py
│   │   ├── cert_manager.py
│   │   └── cache_manager.py
│   ├── shell/
│   │   ├── __init__.py
│   │   ├── grpc_os_shell.py
│   │   ├── completers.py
│   │   └── command_handlers.py
│   └── utils/
│       ├── __init__.py
│       ├── logging_config.py
│       ├── validators.py
│       ├── auth_middleware.py
│       └── metrics.py
├── protos/
│   ├── os_services.proto
│   ├── dns_services.proto
│   ├── auth_services.proto
│   └── backup_services.proto
├── client/
│   ├── __init__.py
│   ├── grpc_os_client.py
│   └── examples/
│       ├── __init__.py
│       ├── basic_operations.py
│       ├── advanced_usage.py
│       └── monitoring.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_filesystem.py
│   ├── test_process_manager.py
│   ├── test_memory_manager.py
│   ├── test_dns_manager.py
│   ├── test_auth_manager.py
│   ├── test_backup_manager.py
│   └── test_integration.py
├── docker/
│   ├── Dockerfile
│   ├── Dockerfile.mariadb
│   ├── Dockerfile.prometheus
│   └── prometheus/
│       └── prometheus.yml
├── deploy/
│   ├── kubernetes/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── ingress.yaml
│   └── ansible/
│       ├── playbook.yml
│       └── inventory.yml
├── docs/
│   ├── API.md
│   ├── SHELL.md
│   ├── INSTALLATION.md
│   ├── CONFIGURATION.md
│   ├── PROCESSES.md
│   ├── DOMAINS.md
│   ├── BACKUP.md
│   ├── SECURITY.md
│   └── CONTRIBUTING.md
├── scripts/
│   ├── install.sh
│   ├── generate_protos.sh
│   ├── run_tests.sh
│   ├── backup.sh
│   └── cert_renewal.sh
├── config/
│   ├── default/
│   │   ├── system.yml
│   │   ├── auth.yml
│   │   └── logging.yml
│   ├── templates/
│   │   ├── nginx.conf.template
│   │   └── mariadb.cnf.template
│   └── examples/
│       ├── process.yml.example
│       └── domain.yml.example
├── processes/
│   └── default/
│       ├── mariadb/
│       │   ├── init.sql
│       │   └── config.yml
│       ├── prometheus/
│       │   └── config.yml
│       └── nginx/
│           └── config.yml
├── data/
│   ├── backups/
│   ├── certs/
│   └── logs/
├── .env.example
├── .gitignore
├── docker-compose.yml
├── docker-compose.override.yml
├── docker-compose.prod.yml
├── Makefile
├── pyproject.toml
├── README.md
└── setup.py
```

## Opis głównych katalogów i plików:

1. `src/` - Kod źródłowy systemu
   - `services/` - Implementacje wszystkich usług systemu
   - `shell/` - Interaktywny shell i jego komponenty
   - `utils/` - Narzędzia pomocnicze

2. `protos/` - Definicje protokołów gRPC
   - Wszystkie pliki `.proto` definiujące interfejsy usług

3. `client/` - Implementacja klienta
   - `examples/` - Przykłady użycia API

4. `tests/` - Testy systemu
   - Testy jednostkowe i integracyjne dla każdej usługi

5. `docker/` - Konfiguracja konteneryzacji
   - Osobne Dockerfile dla różnych komponentów

6. `deploy/` - Konfiguracje wdrożeniowe
   - `kubernetes/` - Manifesty Kubernetes
   - `ansible/` - Playbooki Ansible

7. `docs/` - Dokumentacja
   - Szczegółowa dokumentacja każdego komponentu
   - Przewodniki instalacji i konfiguracji

8. `scripts/` - Skrypty pomocnicze
   - Instalacja, generowanie, backup, itp.

9. `config/` - Pliki konfiguracyjne
   - Konfiguracje domyślne i szablony

10. `processes/` - Definicje procesów
    - Konfiguracje domyślnych procesów systemowych

11. `data/` - Katalog na dane
    - Backupy, certyfikaty, logi

## Pliki konfiguracyjne:

1. `docker-compose.yml` - Podstawowa konfiguracja kontenerów
2. `docker-compose.override.yml` - Konfiguracja dla środowiska dev
3. `docker-compose.prod.yml` - Konfiguracja produkcyjna
4. `Makefile` - Komendy do zarządzania projektem
5. `pyproject.toml` - Konfiguracja projektu Python
6. `.env.example` - Przykładowe zmienne środowiskowe
